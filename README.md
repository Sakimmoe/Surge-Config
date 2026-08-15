# Surge-Config

个人 Surge 配置备份仓库：主配置、YouTube 去广告模块、Snell 一键脚本。

## 目录结构

```text
Surge-Config/
├── Surge.conf                              # Surge 主配置（MTProto secret 已打码）
├── Modules/
│   ├── YouTubeEnhance-Original.sgmodule    # YouTube 去广告原版（手机端完整，含卡片广告）
│   └── YouTubeEnhance-WebCompat.sgmodule   # 网页兼容版（电脑可播，手机端可能漏广告）
└── Snell/
    ├── snell                               # Snell v5 一键安装/管理脚本（来自 Sakimmoe/Snell）
    └── REVIEW.md                           # 脚本审查意见
```

## Surge 主配置

`Surge.conf` 为个人配置备份。发布到公开仓库前已将 `[MTProto]` 的 `secret` 替换为占位符，本地使用前请填回自己的密钥。

## YouTube 模块

- 原版（手机端完整去广告）：<https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Modules/YouTubeEnhance-Original.sgmodule>
- 网页兼容版：<https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Modules/YouTubeEnhance-WebCompat.sgmodule>

推荐：手机用原版；电脑若必须走手机 Surge 代理，请在 Windows 安装并信任 Surge 的 MITM 根证书。

## Snell 一键脚本

脚本来自 <https://github.com/Sakimmoe/Snell>，支持 Debian / Ubuntu / CentOS，功能包括安装、更新、改端口/密码、切换监听模式、防火墙、Swap 调整等。审查意见见 [Snell/REVIEW.md](Snell/REVIEW.md)。
