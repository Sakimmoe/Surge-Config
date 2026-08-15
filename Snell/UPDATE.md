# Snell 脚本版本升级教程（纯手动修改）

适用场景：官方发布新版本（例如 v6.0.0rc3）后，手动改仓库代码，再在服务器上重新安装。脚本已移除在线更新功能。

行号以仓库 main 分支当前版本为准；改动后行号会变化，按标记文字搜索即可。

## 需要改的文件

| 文件 | 作用 |
| --- | --- |
| `Snell/snell` | 版本号、配置生成、客户端节点行、服务启动参数 |
| `vendor/` | 三个架构的官方安装包（IPv6 备用下载源） |
| `Snell/README.md` | 版本描述、SHA256 |

## 一、改脚本版本号

文件：`Snell/snell`

- 第 23 行：`SNELL_VERSION="v6.0.0rc2"`
- 把 `v6.0.0rc2` 改成新版号，例如 `v6.0.0rc3`

这一行决定下载哪个版本：脚本第 309 行按 `${SNELL_VERSION}` 拼官方下载地址，第 310 行按它拼仓库 `vendor/` 备用地址，所以版本号必须和安装包文件名完全一致。

## 二、替换 vendor/ 安装包

- 删除 `vendor/` 里旧的三个文件：
  - `snell-server-v6.0.0rc2-linux-amd64.zip`
  - `snell-server-v6.0.0rc2-linux-i386.zip`
  - `snell-server-v6.0.0rc2-linux-aarch64.zip`
- 放入新版三个文件，文件名严格保持 `snell-server-v6.0.0rc3-linux-amd64.zip` 这种格式（amd64 / i386 / aarch64）

## 三、改 Snell/README.md

- 第 11 行：功能第 1 条里的 `snell-server v6.0.0rc2` 改成新版号
- 第 75-80 行：「官方安装包 SHA256」一节，把三行哈希和文件名换成新包的值（哈希用你常用的校验工具算）

## 四、新版本配置格式变了才改的代码

官方发布说明里如果新增、删除或改名了配置项，才需要动下面这些地方：

1. 配置文件模板
   - 文件：`Snell/snell`
   - 第 284 行开始：`write_snell_conf()`，生成的配置内容是 `listen / psk / mode / dns / dns-ip-preference`
   - 官方改了配置键就在这里增删对应行
2. 三种网络模式的取值
   - 第 258 行：`make_listen()`，三种模式的监听地址
   - 第 267 行：`make_dns()`，三种模式的 DNS 服务器
   - 第 275 行：`make_dns_pref()`，三种模式的 `dns-ip-preference`
3. 客户端节点行
   - 第 431 行开始：`export_snell_info()`
   - 第 437-438 行：`EXTRA` 变量里的 `version=6`，协议版本变了就改成新版
   - 第 442、445、448 行：IPv4 / IPv6 / 兜底三行节点信息的生成，格式变了改这里
4. 服务启动参数
   - 第 887 行：`ExecStart=/usr/local/bin/snell-server -c ... --loglevel warning`
   - 新版命令行参数有变化就改这一行
5. 安装依赖（一般不用动）
   - 第 769 行：Debian/Ubuntu 的依赖安装列表；新版本有新依赖才加

## 五、推送仓库

改完后提交并推送到 GitHub，确保 `main` 分支是更新后的版本。

## 六、服务器重新安装

1. 先打开 `/etc/snell/snell-server.conf`，记下端口和密码
2. 在服务器上重新运行一键命令（脚本启动时会自动刷新本地的 `s` 面板）
3. 选 1「安装 / 重装 Snell」
4. 端口、密码填回原来的值；协议模式按客户端选择（默认选 1 `default`）
5. 安装完成看输出版本号，再进面板选 8 一键体检确认服务正常

## 注意事项

- 版本号（第 23 行）、vendor 文件名、README 里的哈希三者必须对应同一个版本
- 重装时端口和密码如果和原来不同，客户端 Surge 节点行也要跟着换
- 官方说明没提配置变化时，只改第 23 行、vendor、README 三处即可
