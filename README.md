# Surge-Config

个人 Surge 配置仓库：主配置、YouTube 去广告模块、Snell 一键脚本。

## 目录结构

```text
Surge-Config/
├── Surge.conf                              # Surge 主配置
├── Modules/
│   ├── README.md                           # 模块说明
│   ├── YouTubeEnhance-Original.sgmodule    # YouTube 去广告原版
│   └── YouTubeEnhance-WebCompat.sgmodule   # 网页兼容版
├── Snell/
│   ├── README.md                           # Snell 脚本说明
│   ├── UPDATE.md                           # 版本升级教程
│   ├── AI_UPDATE.md                        # AI 升级任务说明
│   ├── REVIEW.md                           # 审查与变更记录
│   └── snell                               # Snell v6 一键脚本
└── vendor/                                 # 官方 snell-server 二进制（IPv6 备用下载源）
```

## 使用

- Surge 主配置：`Surge.conf` 融合自 [官方配置示例](https://gist.githubusercontent.com/Zeaphyou/864aebea248ca1bb8000e0e5623b65f3/raw/c36413c715f43f22772d3c2353358e1ff936b2e6/Surge.conf) 与 [EAlyce/conf](https://github.com/EAlyce/conf)，使用前在 `[Proxy]` 填入你的 Snell 节点
- 分流原则：国外网站走代理、国内网站走直连
- YouTube 模块：见 [Modules/README.md](Modules/README.md)
- Snell 一键脚本：见 [Snell/README.md](Snell/README.md)
- Snell 版本升级：见 [Snell/UPDATE.md](Snell/UPDATE.md)
