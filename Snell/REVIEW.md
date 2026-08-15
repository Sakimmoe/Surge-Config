# Snell 脚本审查与变更记录

## v2.0.0（重写版）

按 [Sakimmoe/AnyTLS](https://github.com/Sakimmoe/AnyTLS/blob/main/anytls) 的整体优化/搭建方式重写，并按 [getsomecat/GetSomeCats 搭建文档](https://github.com/getsomecat/GetSomeCats/blob/Surge/%E7%AE%80%E5%8D%95%E6%90%AD%E5%BB%BASnell%E6%9C%8D%E5%8A%A1.md) 调整搭建细节。

采用的内容：

- 搭建方法：官方 `dl.nssurge.com` 下载 v5.0.1 → 解压到 `/usr/local/bin` → `/etc/snell/snell-server.conf` → systemd 以 nobody 运行（含 `CAP_NET_BIND_SERVICE`、syslog 输出、`LimitNOFILE`）
- 套用 AnyTLS：BBR + fq、TCP Fast Open、大缓冲区、Swap 自动调整、UFW、每周清理、快捷命令、更新备份回滚、菜单结构
- 客户端输出格式参考文档：`snell, IP, 端口, psk=..., version=5, reuse=true, ecn=true, tfo=true`

修复旧版问题：

- `listen = [::]:端口` → 官方 v5 写法 `listen = ::0:端口`
- 配置文件权限收紧：`/etc/snell/snell-server.conf` 640（root:nogroup），节点信息 600
- systemd 增加 User=nobody / Group 回退（CentOS 无 nogroup 时用 nobody）
- 保留 PSK 16-180 位字符校验、IPv4/IPv6/双栈输出、`--upgrade` 快捷入口
- 菜单：9 = 卸载，10 = 重新应用网络优化 / 调整 Swap，0 = 退出
- 官方下载源无 IPv6：纯 IPv6 服务器自动回退到仓库 `vendor/` 内备份的官方二进制
- 架构支持补齐：amd64 / i386 / aarch64 / armv7l 自动选择（`uname -m` 自动匹配）

## 仍保留的注意事项（继承自 AnyTLS 模板）

- `judge()` 失败检测不可靠：部分命令后带 `|| true`，失败也可能显示“完成”
- UFW 使用 `--force reset`，会清空服务器已有防火墙规则
- 卸载不会恢复 DNS、Swap、sysctl 等系统改动
- “检查更新”比较的是脚本内硬编码版本号 v5.0.1，不会在线发现新版本（官方已出 v6 RC）
- 双栈与仅 IPv6 的 v5 配置写法相同（`listen = ::0`），实际差异取决于服务端实现
- 设置时区、禁用 systemd-resolved、覆盖 DNS 等属于“代理机专用”的激进改动

## 三种网络模式的配置对照（已按官方文档/示例核对，未在真实服务器实测）

- 仅 IPv4：`listen = 0.0.0.0:端口` + `ipv6 = false`
- 双栈：`listen = ::0:端口` + `ipv6 = true`（v5 依赖 IPv6 套接字的系统兼容行为同时接受 IPv4）
- 仅 IPv6：`listen = ::0:端口` + `ipv6 = true`（v5 下可能同时接受 IPv4 映射连接，属官方 v5 限制，v6 已支持多地址监听）
