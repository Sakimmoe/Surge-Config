# Snell 脚本版本升级教程（改脚本 + 服务器重装）

适用场景：官方发布新版本（例如 v6.0.0rc3）后，把仓库脚本升级到新版，然后服务器重新安装。

脚本已经移除在线更新功能，所以升级必须按这个流程手动操作。

## 一、仓库端修改

### 1. 拿到官方新版下载链接

官方发布说明：

<https://kb.nssurge.com/surge-knowledge-base/release-notes/snell>

复制新版号，例如 `v6.0.0rc3`。注意官方只提供 amd64 / i386 / aarch64，没有 armv7l。

### 2. 修改脚本里的版本号

打开 `Snell/snell`，找到：

```bash
SNELL_VERSION="v6.0.0rc2"
```

改成：

```bash
SNELL_VERSION="v6.0.0rc3"
```

### 3. 下载三个官方安装包

```bash
curl -fLO https://dl.nssurge.com/snell/snell-server-v6.0.0rc3-linux-amd64.zip
curl -fLO https://dl.nssurge.com/snell/snell-server-v6.0.0rc3-linux-i386.zip
curl -fLO https://dl.nssurge.com/snell/snell-server-v6.0.0rc3-linux-aarch64.zip
```

### 4. 替换仓库里的 vendor/ 备用源

`vendor/` 是纯 IPv6 服务器下载失败时的官方二进制备用源：

- 删除旧的 `snell-server-v6.0.0rc2-linux-*.zip`
- 放入新的 `snell-server-v6.0.0rc3-linux-*.zip`

### 5. 更新 README 里的 SHA256

```bash
sha256sum snell-server-v6.0.0rc3-linux-*.zip
```

把 `Snell/README.md` 底部“官方安装包 SHA256”一节替换成新的三行。

### 6. 检查其他版本号描述

- `Snell/README.md` 功能第 1 条里的 `snell-server v6.0.0rc2`
- 本教程里的示例链接（不影响使用，但建议保持一致）

### 7. 如果新版本改了配置格式

先看官方发布说明有没有新增/删除配置项，有变化就同步修改 `Snell/snell` 里生成配置的函数：

- `write_snell_conf()`：生成 `listen / psk / mode / dns / dns-ip-preference`
- `make_listen()` / `make_dns()` / `make_dns_pref()`：三种网络模式的对应值
- `export_snell_info()`：客户端节点行，比如协议版本变了要把 `version=6` 改成新版本

### 8. 提交并推送

```bash
git add -A
git commit -m "Update Snell to v6.0.0rc3"
git push origin main
```

## 二、服务器端重新安装

### 1. 先记下当前端口和密码

重装时端口和密码要填回一样的，客户端才不用改：

```bash
cat /etc/snell/snell-server.conf
```

或者在面板里看：`s` → 2 查看节点配置。

### 2. 重新运行脚本

```bash
bash <(curl -sL https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Snell/snell)
```

脚本启动时会自动刷新本地的 `s` 快捷面板。

### 3. 选 1「安装 / 重装 Snell」

- 端口：填回原来的端口
- 密码：填回原来的密码（回车会生成新密码，客户端也要跟着改）
- 协议模式：和客户端保持一致，默认选 1 `default`

重装会先停止旧服务，下载新版失败时会自动重启旧服务，不会弄丢旧二进制。

### 4. 验证新版

安装完成看输出的版本号，再确认服务正常：

```bash
systemctl status snell --no-pager
ss -tlnp | grep snell
journalctl -u snell -n 20 --no-pager
```

也可以在面板里选 8 一键体检。

### 5. 客户端

- 端口、密码没变：Surge 里原来的节点行不用改
- 变了：用安装完成后输出的 IPv4 / IPv6 节点行替换

## 三、常见问题

### 为什么不能用菜单更新了？

脚本已移除在线更新功能，因为版本升级往往需要同步换配置，手动改更可控。

### 重装会丢配置吗？

重装会按你输入重新生成配置，所以先记下端口和密码。监听模式会自动按服务器网络检测，协议模式重新选一次。

### 只想换二进制、不想动配置怎么办？

可以只手动替换二进制：备份 → 下载 → 替换 `/usr/local/bin/snell-server` → 启动验证；或者重装时把所有参数填成和原来一样。
