# Snell 脚本审查与变更记录

## 双栈模式出站也改为仅 IPv4

- 用户要求彻底关闭 IPv6 出站：双栈模式（IPv6 入站 + IPv4 出站）的 `dns-ip-preference` 从 `prefer-ipv4` 改为 `ipv4-only`
- 现在三种模式统一 `ipv4-only` 出站：入站可以收 v6，出站只走 IPv4
- DNS 统一使用 IPv4（1.1.1.1 / 8.8.8.8）

## 仅 IPv6 模式改为 IPv6 入站 + IPv4 出站

- 之前“仅 IPv6”模式把出站也设为 IPv6（`dns-ip-preference = ipv6-only`），与 AnyTLS 的 v6 节点行为不一致
- 现在“仅 IPv6”模式：`listen = [::]:端口`（IPv6 入站），`dns = 1.1.1.1,8.8.8.8` + `dns-ip-preference = ipv4-only`（彻底关闭 IPv6 出站）
- 双栈模式同样为 `ipv4-only`（IPv6 入站 + 仅 IPv4 出站）

## IPv6 节点行去掉方括号

- 用户实测：Surge 的 Snell 节点行里 IPv6 地址带方括号（`[2a0e:...]`）连不通，去掉方括号正常
- `export_snell_info()` 的 IPv6 节点行改为不带方括号；服务器配置里的 `[::]` 监听地址不受影响
- README 与 AI_UPDATE.md 已注明该规则，防止后续升级时改回去

## 新增 AI_UPDATE.md（AI 升级任务说明）

- 新增 `Snell/AI_UPDATE.md`：一份自包含的升级任务书，Snell 出新版时把整段内容发给 AI，即可让它完成仓库端升级（版本号、vendor、SHA256、配置函数检查、语法检查、推送）
- README 与根目录 README 已加入入口说明

## 新增 UPDATE.md 升级教程

- 按需求新增 `Snell/UPDATE.md`：仓库端改 `SNELL_VERSION`、替换 `vendor/`、更新 SHA256、按需修改配置生成函数 → 推送 → 服务器菜单 1 重装
- README 里原来的“手动更新 Snell（服务端）”替换为指向 UPDATE.md 的入口

## 移除更新功能（改为手动更新）

- 删除菜单 2「更新」、`--upgrade` 入口、`update_snell` / v5→v6 迁移 / 回滚函数、`snell -v/--upgrade` 快捷参数
- 删除不再使用的版本记录与相关函数：`.version`、`get_installed_version`、`get_remote_version`、`cleanup_old`、`get_ip`、`get_conf_mode` 等
- 菜单重排：2 查看节点配置、3 更改端口、4 更改密码、5 监听/协议模式、6 重启、7 运行状态、8 一键体检（新增）、9 重新应用网络优化 / 调整 Swap、10 卸载、0 退出（9/10/0 位置保持不变）
- 服务端升级改为手动：备份 → 下载官方包 → 替换二进制 → 按需改配置 → 验证 → 失败回滚，完整步骤见 README

## v3.0.5（Debian/Ubuntu 长期使用的清理与日志优化）

- 清理脚本针对 Debian/Ubuntu 调整：
  - `apt-get autoremove --purge -y`，残留依赖与旧内核清得更干净
  - systemd journal 从 7 天 / 50M 收紧为 3 天 / 30M（Snell 日志也在 journal 里）
  - 补充清理 v5→v6 迁移备份 `*.v5bak` 等历史残留
- systemd 服务增加 `--loglevel warning`，从源头减少 Snell 日志写入
- 旧安装自动适配：每次进入面板时检测服务文件，缺 `--loglevel` 就补上并重启；启动失败自动回滚
- 菜单 9「重新应用网络优化 / 调整 Swap」现在会顺带重写每周清理任务，老服务器不用重装即可刷新
- Debian 11 / Ubuntu（apt 系）为脚本的主要支持目标

## 节点名称修复

- 精简输出为单行后，行首节点名被写死为 `Snell_端口`，用户输入的节点名称被忽略
- 现在安装时填写的节点名称会直接作为 Surge 策略名输出（IPv4 / IPv6 行一致），并自动过滤逗号、换行等会破坏配置行的字符

## 依赖精简与固定节点名

- 删除“请输入节点名称”步骤，节点输出行固定使用 `IPv4` / `IPv6` 作为 Surge 策略名
- 安装依赖精简：
  - apt：只装 `curl unzip ufw iproute2 cron ca-certificates`，不再装 `wget` / `tar`
  - dnf / yum：只装 `curl unzip cronie ca-certificates`，不再装未使用的 `firewalld` / `tar`
  - 下载改用 curl 优先，wget 仅作为可选回退（有就用到，没有也不安装）

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
- 新增 `dns` 与 `dns-ip-preference`：三种模式统一 `ipv4-only` 出站，DNS 统一用 IPv4
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
- 脚本不再提供在线更新；官方出新版后按 README 的手动步骤备份、替换二进制、按需改配置并验证
- v6 仍是 RC 测试版，官方可能在正式版前做不兼容协议调整；客户端 Surge 也必须更新到支持 v6 的测试版
- 仅 IPv6 模式使用 `[::]` 监听，Linux 默认 IPv6 套接字可能同时接受 IPv4 映射连接（与 v5 同源限制）
- 设置时区、禁用 systemd-resolved、覆盖 DNS 等属于“代理机专用”的激进改动

## 三种网络模式的配置对照（已按官方帮助/示例核对，未在真实服务器实测）

- 仅 IPv4：`listen = 0.0.0.0:端口` + `dns-ip-preference = ipv4-only`
- 双栈：`listen = 0.0.0.0:端口,[::]:端口` + `dns-ip-preference = ipv4-only`（IPv6 入站 + 仅 IPv4 出站）
- 仅 IPv6：`listen = [::]:端口` + `dns-ip-preference = ipv4-only`（IPv6 入站 + 仅 IPv4 出站）
