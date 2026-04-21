# 豆瓣相关开源项目对比矩阵

> 抓取时间：2026-04-21。所有 star 数、更新时间均为 GitHub 页面直读。"✅/❌/⚠" 表示能否用于自动化**发布**（非读取）。

## A. 签名与逆向类（frodo API）

| # | 仓库 | 语言 | stars | 最近提交 | 维护？ | 能发内容？ | 备注 |
| - | --- | --- | --: | --- | :-: | :-: | --- |
| 1 | [bestyize/DoubanAPI](https://github.com/bestyize/DoubanAPI) | Java | 67 | 2020-09-18 | ❌ | ⚠ 仅签名，未封装写接口 | 最权威的 `_sig` 实现参考 |
| 2 | [kxy000/doubanapi](https://github.com/kxy000/doubanapi) | Markdown | 183 | 2020-05-26 | ❌ | ❌ 纯文档 | 历史 v2 写接口字段文档 |
| 3 | [zce/douban-api-docs](https://github.com/zce/douban-api-docs) | Docs | 2 | 2021-08-10 | ❌ | ❌ 已清空 | 曾整理 API，2021 年删光 |

## B. 官方/历史 OAuth 客户端

| # | 仓库 | 语言 | stars | 最近提交 | 能发？ | 备注 |
| - | --- | --- | --: | --- | :-: | --- |
| 4 | [douban/douban-objc-client](https://github.com/douban/douban-objc-client) | Objective-C | 253 | 2016-10-08 | ❌ 无新 client_id 可申请 | 官方 2.0 OAuth 参考 |

## C. 数据同步/备份类（读）

| # | 仓库 | 语言 | stars | 最近提交 | 能发？ | 备注 |
| - | --- | --- | --: | --- | :-: | --- |
| 5 | [fugary/calibre-web-douban-api](https://github.com/fugary/calibre-web-douban-api) | Python | 464 | 2024-06-08 | ❌ 只读 | calibre-web 图书元数据 |
| 6 | [qiaohaoforever/BambooIsbn](https://github.com/qiaohaoforever/BambooIsbn) | JS | 504 | 2024-11-21 | ❌ 只读 | ISBN 信息聚合（含豆瓣） |
| 7 | [somkanel/douban-sync-action](https://github.com/somkanel/douban-sync-action) | Python | 低 | 2024 | ❌ 只读 | GitHub Action，Cookie 同步到 JSON |
| 8 | [caspartse/eBooksAssistant](https://github.com/caspartse/eBooksAssistant) | Python | 135 | 2024-07-21 | ❌ 只读 | 豆瓣/微信读书元数据 |
| 9 | [ChanMeng666/douban-review-scraper](https://github.com/ChanMeng666/douban-review-scraper) | Python | 低 | 2024-11-25 | ❌ 只读 | **Cookie 结构样例最完整** |
| 10 | [doubaniux/DoubanSpider](https://github.com/doubaniux/DoubanSpider) | Scrapy | 低 | — | ❌ 只读 | Scrapy + PG 存储 |

## D. 广播/日记备份类

| # | 仓库 | 语言 | stars | 最近提交 | 能发？ | 备注 |
| - | --- | --- | --: | --- | :-: | --- |
| 11 | [yekingyan/Backup-Douban-Broadcast](https://github.com/yekingyan/Backup-Douban-Broadcast) | HTML/JS | 18 | 2018-08-24 | ❌ | 备份广播到本地 HTML |
| 12 | [shin000000/backup_your_douban_broadcast](https://github.com/shin000000/backup_your_douban_broadcast) | Python | 0 | 2022-06-17 | ❌ | "quick and dirty" 备份 |
| 13 | [zhuth/DoubanDiaryBackup](https://github.com/zhuth/DoubanDiaryBackup) | C# | 12 | 2023-02-01 (archived) | ❌ | 已归档 |
| 14 | [sprhawk/DoubanBackup](https://github.com/sprhawk/DoubanBackup) | Python | 3 | 2014-04-12 | ❌ | — |

## E. Playwright/Selenium 端

| # | 仓库 | 语言 | stars | 最近提交 | 能发？ | 备注 |
| - | --- | --- | --: | --- | :-: | --- |
| 15 | [Apauto-to-all/spiderDoubanTvAndExplore](https://github.com/Apauto-to-all/spiderDoubanTvAndExplore) | Python (Playwright) | 低 | 近 1 年 | ❌ | 爬电视剧榜单 |
| 16 | [JimSunJing/somegreasyjs](https://github.com/JimSunJing/somegreasyjs) | JS userscript | 1 | 2023-03-10 | ❌ | Greasyfork 备份广播 |
| 17 | [xiaoroa/DBroadcast](https://github.com/xiaoroa/DBroadcast) | JS | 0 | — | ⚠ "check and post douban broadcast" 但极少 star、极少代码 | 唯一自称能发广播的公开仓库，谨慎使用 |

## F. MCP Server（LLM 工具端）

| # | 仓库/包 | stars/popularity | 能发？ | 备注 |
| - | --- | --- | :-: | --- |
| 18 | [mcp-douban-server (npm)](https://www.npmjs.com/package/mcp-douban-server) | 低 | ❌ | 搜索书/电影/小组话题 |
| 19 | moria97/douban-mcp | 低 | ❌ | 同上 |

## 汇总统计

- 共收录 **19 个**公开资源。
- 公开仓库中**明确声称能"发布"** 的：仅 `xiaoroa/DBroadcast`（0 star，极简 JS，自行评估）。
- 所有 ≥ 100 star 的项目都是**只读或文档**。
- 2024 年后仍在维护的：`fugary/calibre-web-douban-api`、`qiaohaoforever/BambooIsbn`、`ChanMeng666/douban-review-scraper`、`somkanel/douban-sync-action`、`caspartse/eBooksAssistant` —— **全部只读**。

**结论**：想自动化发豆瓣，无法站在巨人肩膀上，必须自研。
