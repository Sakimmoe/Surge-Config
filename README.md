# Surge-Config

个人 Surge 配置备份仓库：主配置、YouTube 去广告模块、Snell 一键脚本。

## 目录结构

```text
Surge-Config/
├── Surge.conf                              # Surge 主配置（MTProto secret 已打码）
├── Modules/
│   ├── README.md                           # 模块说明
│   ├── YouTubeEnhance-Original.sgmodule    # YouTube 去广告原版
│   └── YouTubeEnhance-WebCompat.sgmodule   # 网页兼容版（电脑可播，手机端可能漏广告）
└── Snell/
    ├── README.md                           # Snell 脚本说明
    ├── snell                               # Snell v5 一键脚本（AnyTLS 风格重写）
    └── REVIEW.md                           # 审查与变更记录
```

## Surge 主配置

`Surge.conf` 为个人配置备份。发布到公开仓库前已将 `[MTProto]` 的 `secret` 替换为占位符，本地使用前请填回自己的密钥。

## 子目录说明

- YouTube 模块：见 [Modules/README.md](Modules/README.md)
- Snell 一键脚本：见 [Snell/README.md](Snell/README.md)
