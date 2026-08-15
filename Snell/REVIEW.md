# Snell 脚本审查与变更记录

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
