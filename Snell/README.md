## 快速开始

```bash
bash <(curl -sL https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Snell/snell)
```

运行过上面这行一次后，服务器上就会留下 `s` 快捷命令，以后直接输入 `s` 就能打开面板；不需要先完成搭建，即使只运行过一次并退出也可以用。

## 功能

1. 安装 / 重装 Snell（snell-server v6.0.0rc2 官方二进制）
2. 查看节点配置
3. 更改端口
4. 更改密码
5. 切换监听模式（IPv4 / 双栈 / IPv6）与协议模式（default / unshaped / unsafe-raw）
6. 重启服务
7. 查看运行状态
8. 一键体检（服务 / BBR / 网络优化 / Swap / DNS / UFW / 时区 / 开机自愈服务）
9. 重新应用系统设置（时区 / 内核优化 / DNS / Swap / 定时清理）
10. 卸载
0. 退出

## 重启后依然生效（重要）

脚本对系统做的改动都会持久化，并且有开机自愈服务兜底：

| 项目 | 落地位置 | 开机兜底 |
| --- | --- | --- |
| BBR / fq / TCP Fast Open | `/etc/sysctl.d/99-snell-network.conf` + `/etc/modules-load.d/snell-bbr.conf` | `snell-net-tune.service` |
| 静态 DNS（1.1.1.1 / 8.8.8.8） | `/etc/resolv.conf`（并 mask `systemd-resolved`） | `snell-dns.service` |
| 时区 Asia/Shanghai + NTP | `/etc/localtime` + `systemd-timesyncd` | `snell-net-tune.service` |
| Swap 关闭 / 512M swapfile | `/etc/fstab` + `disable-swap.service` | `disable-swap.service` |
| Snell 服务自启 | `snell.service` | systemd |

**为什么必须写 `/etc/sysctl.d/` 而不是 `/etc/sysctl.conf`：**
Debian 13（trixie）起，`systemd` 删除了兼容软链 `/etc/sysctl.d/99-sysctl.conf`，`procps` 也不再提供 `/etc/sysctl.conf`，`systemd-sysctl.service` 只读 `/usr/lib/sysctl.d/`、`/etc/sysctl.d/`、`/run/sysctl.d/`。写 `/etc/sysctl.conf` 会出现“装完当时有效、一重启 BBR 就没了”。本脚本已改为写入 `/etc/sysctl.d/99-snell-network.conf`，并在首次运行时把旧版写进 `/etc/sysctl.conf` 的三行迁移走（备份在 `/etc/snell/sysctl.conf.bak`）。

装完或重启后，用菜单 8「一键体检」能看到 BBR / DNS / 时区的实际值与自愈服务状态；如果是从旧版覆盖安装，重新运行一次脚本（菜单 9 也可以）即可完成迁移。

## 升级 Snell 版本

脚本已移除在线更新功能。官方发布新版本后，完整升级教程（改脚本版本号与 vendor 备用源 → 推送仓库 → 服务器重装）见 [UPDATE.md](UPDATE.md)。

想直接让 AI 帮忙升级，把 [AI_UPDATE.md](AI_UPDATE.md) 的内容整段发给 AI 即可。

## 日志与清理

- Snell 日志写入 systemd journal，服务以 `--loglevel warning` 运行，日常日志量很小
- 每周日 07:07 自动清理：apt 残留依赖（`--purge`）与索引缓存（pkgcache.bin）、journal 保留 3 天 / 30M（含 Snell 日志）、Snell 更新残留、/tmp 中 7 天以上的文件
- 旧版本装的服务在重新运行本脚本时会自动补上日志级别限制，不需要重装

## 客户端配置（Surge）

默认模式：

```ini
Snell_26216 = snell, 服务器IP, 26216, psk=密码, version=6, reuse=true, ecn=true, tfo=true
```

非 default 模式（例如 unshaped）：

```ini
Snell_26216 = snell, 服务器IP, 26216, psk=密码, version=6, mode=unshaped, reuse=true, ecn=true, tfo=true
```

服务端与客户端的 mode 必须一致。Snell v6 目前是测试版，客户端需要支持 v6 的最新 Surge 测试版。

安装完成会同时输出 IPv4 / IPv6 两条节点行（服务器有哪种就输出哪种），节点名固定为 `IPv4` / `IPv6`。`reuse` / `ecn` / `tfo` 都是可选优化参数，删掉只保留 `version=6` 也能连通。

注意：Snell 的 IPv6 节点行地址**不要加方括号**，例如 `IPv6 = snell, 2a0e:97c0:3f4:1::d8e, 26216, ...`，加括号会连不通。

## 与旧版差异

- 官方 v6 仅提供 amd64 / i386 / aarch64，不再有 armv7l
- `listen` 支持多地址同时监听，例如 `0.0.0.0:26216,[::]:26216`，不再依赖 IPv6 套接字兼容 IPv4
- 新增 `mode`：`default`（混淆 + AES）、`unshaped`（仅 AES，约快 10%）、`unsafe-raw`（明文，仅限内网）
- 新增 `dns-ip-preference` 与 `dns` 配置，按服务器真实网络自动生成：
  - 仅 IPv4 服务器：`ipv4-only` + IPv4 DNS，入站 / 出站都是 IPv4
  - 双栈服务器：`ipv4-only` + IPv4 DNS，入站可收 IPv4 / IPv6，出站只用 IPv4（彻底关闭 IPv6 出站）
  - 纯 IPv6 服务器：`ipv6-only` + IPv6 DNS，入站 / 出站只能用 IPv6
- 双栈或仅 IPv4 服务器手动选“仅 IPv6 入站”时：监听只开 IPv6，出站仍是 IPv4
- v6 移除 QUIC Proxy Mode，防火墙只需放行 TCP
- PSK 由协议内派生为部署级流量特征，不同 PSK 的服务器流量特征不同；PSK 长度 12-255
- systemd 以 nobody 运行，配置权限收紧为 640，节点信息 600
- 安装依赖只保留 Snell 实际用到的：curl、unzip、UFW、iproute2、cron、ca-certificates（不再装 wget / tar / firewalld）
- 网络优化只保留最简三项：BBR、fq、TCP Fast Open（`tcp_fastopen = 3`），写入 `/etc/sysctl.d/99-snell-network.conf`，其余参数保持系统默认（不再覆盖 `/etc/sysctl.conf`）
- 官方下载源 `dl.nssurge.com` 只有 IPv4，纯 IPv6 服务器会自动改用仓库内 `vendor/` 的官方二进制备用源

## 注意事项

- Snell v6 仍为 RC 测试版，官方可能在正式版前做不兼容协议调整，服务端与客户端都应保持最新测试版
- 会启用 UFW 并重置现有防火墙规则
- 会禁用 systemd-resolved 并覆盖 DNS 为 1.1.1.1 / 8.8.8.8（由 `snell-dns.service` 保持）
- 内存 ≥ 1G 时会关闭全部 Swap 并安装开机禁用服务
- 会安装 `snell-net-tune.service` / `snell-dns.service` 两个开机自愈服务
- 卸载默认保留以上系统改动（卸载时会打印手动还原命令）
- 仅适合“服务器只跑代理”的场景

## 官方安装包 SHA256（vendor/ 备用源）

```text
8a9c4463ca87cfa5eaa37c6af0d37ab93ea275aa12391985bb2a375ca3abd7f2  snell-server-v6.0.0rc2-linux-amd64.zip
67060ef79ac4ef0bb64c520302396620b3a06f8f6d5ceb450be283e4fa749335  snell-server-v6.0.0rc2-linux-i386.zip
a0b2915cbc77dc3baf8fa069e741c20808d8a10c3a8a93e709a0a580645c3bd7  snell-server-v6.0.0rc2-linux-aarch64.zip
```
