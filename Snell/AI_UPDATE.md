# Snell 版本升级任务说明（发给 AI 使用）

当你看到这份说明时，说明 Snell 官方发布了新版本，需要你（AI）帮我完成仓库 `Sakimmoe/Surge-Config` 的 Snell 脚本升级。

## 背景

- 仓库地址：<https://github.com/Sakimmoe/Surge-Config>
- 涉及文件：
  - `Snell/snell`：Snell v6 一键管理脚本（bash）
  - `vendor/`：官方 snell-server 安装包（纯 IPv6 服务器下载失败时的备用源）
  - `Snell/README.md`：脚本说明与 SHA256
  - `Snell/UPDATE.md`：手动升级教程（其中的行号可能过期，以实际文件为准）
  - `Snell/REVIEW.md`：审查与变更记录，升级后请追加说明
- 脚本已移除在线更新功能，不要重新加回菜单更新项。

## 你的任务

1. 先读取仓库里 `Snell/snell`、`Snell/README.md`、`Snell/UPDATE.md`，了解当前版本与结构。
2. 访问官方发布说明，获取最新版本号和下载链接：
   <https://kb.nssurge.com/surge-knowledge-base/release-notes/snell>
   如有新版介绍，也参考官方博客：<https://www.nssurge.com/blog/snell-v6/>
3. 修改 `Snell/snell` 中的 `SNELL_VERSION`（搜索这个变量即可，当前大约在第 23 行）为新版本号。
4. 下载官方 amd64 / i386 / aarch64 三个安装包；如果官方不再提供某个架构，删除 `vendor/` 里对应的旧文件，并在说明中注明。
5. 校验 SHA256 后替换 `vendor/` 下的三个安装包，旧版文件删除。文件名必须与 `SNELL_VERSION` 完全一致，格式为 `snell-server-<版本>-linux-<架构>.zip`。
6. 更新 `Snell/README.md`：
   - 功能列表里的版本号描述
   - 底部“官方安装包 SHA256”列表
7. 阅读官方发布说明，判断以下内容是否有变化，有变化才改代码：
   - 配置项变化 → 修改 `write_snell_conf()`，以及 `make_listen()` / `make_dns()` / `make_dns_pref()`
   - 协议版本变化（例如不再是 `version=6`）→ 修改 `export_snell_info()` 中的 `EXTRA`
   - 服务端启动参数变化 → 修改 systemd 单元的 `ExecStart`
8. 如有需要，同步更新 `Snell/UPDATE.md` 的行号说明，并在 `Snell/REVIEW.md` 追加本次变更记录。
9. 语法检查（`sh -n` / `dash -n`），保持 LF 行尾，然后提交并推送到 `main` 分支。

## 注意事项

- 不要改动：菜单 9 / 10 / 0 的位置、节点输出名（`IPv4` / `IPv6`）、依赖精简原则、`--loglevel warning`。
- 不要改动：Snell 节点行里的 IPv6 地址不加方括号（实测加方括号会连不通），服务器配置里的 `[::]` 监听地址除外。
- 不要改动：出站策略按服务器真实网络决定——仅 v4 服务器 `ipv4-only`；双栈 `prefer-ipv4`（IPv4 出站优先，显式 IPv6 目标如 MTProto `ipv6=true` 的 Telegram DC 仍可走 v6）；纯 IPv6 `ipv6-only`。这是刻意设计，与 AnyTLS 脚本行为一致。
- 不要重新加回“在线更新”功能。
- 如果官方新版本与旧版差异较大（例如配置格式不兼容），务必说明差异并给出迁移建议，必要时先询问我，不要擅自改动配置生成逻辑。
- 服务器端升级由我手动执行（重跑一键命令 → 菜单 1 重装 → 端口/密码填回原值）。你只需要保证仓库代码正确。

## 完成后请回复

- 新版版本号
- 改了哪些文件、每个文件改了什么
- 是否涉及配置项 / 协议版本 / 启动参数变化
- 推送结果
