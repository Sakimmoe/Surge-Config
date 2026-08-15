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
8. 一键体检（服务 / BBR / 网络优化 / Swap / DNS / UFW / 时区）
9. 重新应用网络优化 / 调整 Swap
10. 卸载
0. 退出

## 升级 Snell 版本

脚本已移除在线更新功能。官方发布新版本后，完整升级教程（改脚本版本号与 vendor 备用源 → 推送仓库 → 服务器重装）见 [UPDATE.md](UPDATE.md)。

## 日志与清理

- Snell 日志写入 systemd journal，服务以 `--loglevel warning` 运行，日常日志量很小
- 每周日 07:07 自动清理：apt 残留依赖（`--purge`）、journal 保留 3 天 / 30M（含 Snell 日志）、Snell 更新残留、/tmp 中 7 天以上的文件
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

## 与旧版差异

- 官方 v6 仅提供 amd64 / i386 / aarch64，不再有 armv7l
- `listen` 支持多地址同时监听，例如 `0.0.0.0:26216,[::]:26216`，不再依赖 IPv6 套接字兼容 IPv4
- 新增 `mode`：`default`（混淆 + AES）、`unshaped`（仅 AES，约快 10%）、`unsafe-raw`（明文，仅限内网）
- 新增 `dns-ip-preference` 与 `dns` 配置，脚本按监听模式自动生成：
  - 仅 IPv4：`dns-ip-preference = ipv4-only`
  - 仅 IPv6：`dns-ip-preference = ipv6-only`
  - 双栈：`dns-ip-preference = prefer-ipv4`（IPv4 出站优先）
- v6 移除 QUIC Proxy Mode，防火墙只需放行 TCP
- PSK 由协议内派生为部署级流量特征，不同 PSK 的服务器流量特征不同；PSK 长度 12-255
- systemd 以 nobody 运行，配置权限收紧为 640，节点信息 600
- 安装依赖只保留 Snell 实际用到的：curl、unzip、UFW、iproute2、cron、ca-certificates（不再装 wget / tar / firewalld）
- 官方下载源 `dl.nssurge.com` 只有 IPv4，纯 IPv6 服务器会自动改用仓库内 `vendor/` 的官方二进制备用源

## 注意事项

- Snell v6 仍为 RC 测试版，官方可能在正式版前做不兼容协议调整，服务端与客户端都应保持最新测试版
- 会启用 UFW 并重置现有防火墙规则
- 会禁用 systemd-resolved 并覆盖 DNS 为 1.1.1.1 / 8.8.8.8
- 内存 ≥ 1G 时会关闭全部 Swap 并安装开机禁用服务
- 卸载不会还原以上系统改动
- 仅适合“服务器只跑代理”的场景

## 官方安装包 SHA256（vendor/ 备用源）

```text
8a9c4463ca87cfa5eaa37c6af0d37ab93ea275aa12391985bb2a375ca3abd7f2  snell-server-v6.0.0rc2-linux-amd64.zip
67060ef79ac4ef0bb64c520302396620b3a06f8f6d5ceb450be283e4fa749335  snell-server-v6.0.0rc2-linux-i386.zip
a0b2915cbc77dc3baf8fa069e741c20808d8a10c3a8a93e709a0a580645c3bd7  snell-server-v6.0.0rc2-linux-aarch64.zip
```
