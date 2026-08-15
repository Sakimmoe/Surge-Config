# Snell 脚本审查与变更记录

## v3.0.4（IPv4 / IPv6 节点行都输出）

- 安装完成与菜单 3 同时输出 IPv4 和 IPv6 两条 Surge 节点行（服务器有哪种就输出哪种），不再只给一条
- 说明：`reuse` / `ecn` / `tfo` 都是可选优化参数，删掉只留 `version=6` 也能连通，不影响协议工作

## v3.0.3（节点信息只输出一行）

- 安装完成、菜单 3 查看节点配置都只输出一条 Surge 节点行，不再打印大段参数表
- 优先输出 IPv4 节点行；只有纯 IPv6 服务器才输出 IPv6 行，避免客户端误用不可达的 IPv6 地址

## v3.0.2（修复模式选择菜单不显示）

- 修复安装时“协议模式”选项不显示：该函数在 `$( )` 中调用，原先把菜单输出到 stdout 会被命令替换吞掉，只剩 `read -rp` 的提示符
- 菜单与警告改为输出到 stderr，并把选项写进提示行：`[1=default / 2=unshaped / 3=unsafe-raw，回车默认 1]`
- `print_warn` / `print_err` 统一输出到 stderr，避免在命令替换中调用时提示不可见

## v3.0.1（s 快捷命令）

- 脚本入口在进入主菜单前先执行 `create_shortcut`，因此只要运行过一次 `bash <(curl -sL ...)`，即使没有安装 Snell，也会留下 `/usr/local/bin/s`
- `s` 优先执行本机保存的面板脚本；如果面板文件缺失，会自动从仓库重新下载后再打开
- 主菜单增加“快捷命令: s”提示，卸载时一并删除 `/usr/local/bin/s`

## v3.0.0（升级到 Snell v6.0.0rc2）

按官方 [Snell 发布说明](https://kb.nssurge.com/surge-knowledge-base/release-notes/snell) 与 [Snell v6 博客](https://nssurge.com/blog/snell-v6/) 升级到 v6，整体优化/搭建方式继续套用 [Sakimmoe/AnyTLS](https://github.com/Sakimmoe/AnyTLS/blob/main/anytls)。

### v6 变化

- 版本：`SNELL_VERSION="v6.0.0rc2"`，官方仅提供 amd64 / i386 / aarch64（无 armv7l）
- `listen` 使用 v6 多地址写法，不再写 v5 的 `::0:端口` 或 `ipv6 = true/false`：
  - 仅 IPv4：`0.0.0.0:端口`
  - 仅 IPv6：`[::]:端口`
  - 双栈：`0.0.0.0:端口,[::]:端口`
- 新增 `mode`（服务端与客户端必须一致）：
  - `default`：混淆 + AES
  - `unshaped`：仅 AES，约快 10%，特征类似 v3 随机流
  - `unsafe-raw`：明文，仅限内网或已有安全隧道
- 新增 `dns` 与 `dns-ip-preference`，脚本按监听模式自动设置：
  - v4only → `ipv4-only`，v6only → `ipv6-only`，双栈 → `prefer-ipv4`
- v6 移除 QUIC Proxy Mode，UFW 只放行 TCP，不再放行 UDP
- PSK 长度放宽为 12-255（v6 官方限制）
- 客户端 Surge 行：`version=6`，非 default 模式追加 `mode=<同名>`
- 版本比较忽略 rc 后缀（v6 二进制 `-v` 输出为 `6.0.0`，与 `v6.0.0rc2` 视为同一版本）

### 新增的可靠性处理

- v5 → v6 更新时自动迁移配置：备份旧配置与监听栈标记，迁移失败或新版本启动失败时回滚二进制和配置
- 菜单 6 拆为子菜单：切换监听模式 / 切换协议模式，菜单 9（重新应用网络优化/调整 Swap）、10（卸载）、0（退出）位置保持不变

## v2.0.0（v5 重写版）

按 AnyTLS 的整体优化/搭建方式重写，并按 [getsomecat/GetSomeCats 搭建文档](https://github.com/getsomecat/GetSomeCats/blob/Surge/%E7%AE%80%E5%8D%95%E6%90%AD%E5%BB%BASnell%E6%9C%8D%E5%8A%A1.md) 调整搭建细节。v3.0.0 保留其 BBR + fq、Swap 自动调整、UFW、每周清理、快捷命令、更新回滚、nobody 运行等结构。

## 仍保留的注意事项

- `judge()` 失败检测不可靠：部分命令后带 `|| true`，失败也可能显示“完成”
- UFW 使用 `--force reset`，会清空服务器已有防火墙规则
- 卸载不会恢复 DNS、Swap、sysctl 等系统改动
- “检查更新”比较的是脚本内硬编码版本号 v6.0.0rc2，不会在线发现新版本；官方正式版出来后需人工更新脚本
- v6 仍是 RC 测试版，官方可能在正式版前做不兼容协议调整；客户端 Surge 也必须更新到支持 v6 的测试版
- 仅 IPv6 模式使用 `[::]` 监听，Linux 默认 IPv6 套接字可能同时接受 IPv4 映射连接（与 v5 同源限制）
- 设置时区、禁用 systemd-resolved、覆盖 DNS 等属于“代理机专用”的激进改动

## 三种网络模式的配置对照（已按官方帮助/示例核对，未在真实服务器实测）

- 仅 IPv4：`listen = 0.0.0.0:端口` + `dns-ip-preference = ipv4-only`
- 双栈：`listen = 0.0.0.0:端口,[::]:端口` + `dns-ip-preference = prefer-ipv4`
- 仅 IPv6：`listen = [::]:端口` + `dns-ip-preference = ipv6-only`
