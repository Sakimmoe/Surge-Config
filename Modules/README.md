# Modules

YouTube 去广告模块，基于 Maasea 的 Youtube (Music) Enhance 修改。

## 文件

| 文件 | 适用场景 | 说明 |
|---|---|---|
| YouTubeEnhance-Original.sgmodule | 手机 App 端 | 完整去广告（含首页卡片广告），需要 MITM googlevideo.com |
| YouTubeEnhance-WebCompat.sgmodule | 电脑必须走手机代理播放 | 移除 googlevideo MITM，电脑可播放，但手机端可能漏广告 |

## 远程地址

- 原版：<https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Modules/YouTubeEnhance-Original.sgmodule>
- 网页兼容版：<https://raw.githubusercontent.com/Sakimmoe/Surge-Config/main/Modules/YouTubeEnhance-WebCompat.sgmodule>

## 推荐组合

手机使用原版模块；电脑若必须走手机 Surge 代理，请在 Windows 安装并信任 Surge 的 MITM 根证书，两边即可同时正常。

## 其他模块

| 文件 | 来源 | 说明 |
|---|---|---|
| iOSUpdate.sgmodule | [ConnersHua/RuleGo](https://github.com/ConnersHua/RuleGo) | 屏蔽 iOS/iPadOS 系统更新 |
| DNS防泄露.beta.sgmodule | [QingRex/LoonKissSurge](https://github.com/QingRex/LoonKissSurge) | 原模块仅含模块信息头，无实际规则 |
| skip-proxy-lists.sgmodule | [mieqq/mieqq](https://github.com/mieqq/mieqq) | 跳过部分应用的代理检测 |
