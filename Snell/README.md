## 快速开始

```bash
bash <(curl -sL https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Snell/snell)
```

## 功能

1. 安装 / 重装 Snell（snell-server v5.0.1 官方二进制）
2. 更新 snell-server（备份 + 失败自动回滚）
3. 查看节点配置
4. 更改端口
5. 更改密码
6. 切换监听模式（IPv4 / 双栈 / IPv6）
7. 重启服务
8. 查看运行状态
9. 卸载
10. 重新应用网络优化 / 调整 Swap
0. 退出

## 客户端配置（Surge）

```ini
Snell_26216 = snell, 服务器IP, 26216, psk=密码, version=5, reuse=true, ecn=true, tfo=true
```

## 与旧版差异

- `listen` 使用官方 v5 写法 `::0:端口`（旧版为 `[::]:端口`）
- systemd 以 nobody 运行，配置权限收紧为 640，节点信息 600
- 整体结构套用 AnyTLS：BBR + fq、TCP Fast Open、Swap 自动调整、UFW、每周清理、快捷命令、更新回滚
- 官方下载源 `dl.nssurge.com` 只有 IPv4，纯 IPv6 服务器会自动改用仓库内 `vendor/` 的官方二进制备用源
- 架构自动选择：amd64 / i386 / aarch64 / armv7l（对应官方四种 Linux 版本）

## 注意事项

- 会启用 UFW 并重置现有防火墙规则
- 会禁用 systemd-resolved 并覆盖 DNS 为 1.1.1.1 / 8.8.8.8
- 内存 ≥ 1G 时会关闭全部 Swap 并安装开机禁用服务
- 卸载不会还原以上系统改动
- 仅适合“服务器只跑代理”的场景
