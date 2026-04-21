# 参考链接汇总（按主题分组）

> 调研日期：2026-04-21。所有链接在本次调研期间至少 `scrape` 或 `web_search` 命中过一次。

## 1. 豆瓣官方通道（API 文档 / OAuth 客户端）

- https://api.douban.com/v2/user/~me — 仍在线但仅商务合作 (`bd@douban.com`)
- https://developers.douban.com/ — 返回 HTTP/2 协议错误（事实下线）
- https://github.com/douban/douban-objc-client — 官方 Objective-C OAuth 2.0 客户端（最后更新 2016-10-08）
- https://github.com/kxy000/doubanapi — 豆瓣 API 文档备份（183 stars，2020）
- https://github.com/zce/douban-api-docs — 文档备份（2 stars，2021 清空）

## 2. frodo.douban.com 签名逆向

- https://github.com/bestyize/DoubanAPI — Java 签名计算器 + servlet 示例（67 stars，2020-09-18）
- https://blog.csdn.net/qq_23594799/article/details/108445726 — 安卓逆向-豆瓣签名（上）：Fiddler + JADX
- https://blog.csdn.net/qq_23594799/article/details/108446352 — （下）：HMAC-SHA1 实现代码
- https://blog.csdn.net/weixin_41259961/article/details/129184887 — _sig 逆向完整流程（Frida）
- https://blog.csdn.net/ckcookies/article/details/128589791 — JADX/Frida 静动结合分析
- https://www.vlwx.com/498.html — HMAC-SHA1 混淆 + key 提取（域名已变更，原文可搜）
- https://blog.einverne.info/post/2018/06/douban-xiaozu.html — 豆瓣小组 API 历史示例

## 3. Cookie 结构 / 网页端端点

- https://github.com/ChanMeng666/douban-review-scraper/blob/master/config.py — 完整 `bid/dbcl2/ck/ll/ap_v/frodotk_db` 样例
- https://agents.baidu.com/content/question/81489be1833f11b7a8292d1c — 豆瓣 Cookie 提取教程
- https://blog.51cto.com/u_14754853/5533992 — Python requests cookie 登录豆瓣
- https://blog.csdn.net/weixin_42552374/article/details/83627987 — 豆瓣爬虫 CookieJar
- https://blog.csdn.net/weixin_55579895/article/details/120452499 — requests cookie 请求方法

## 4. 数据同步/备份工具（只读）

- https://github.com/fugary/calibre-web-douban-api — 464 stars，Python，calibre-web 元数据
- https://github.com/qiaohaoforever/BambooIsbn — 504 stars，ISBN/豆瓣图书接口
- https://github.com/somkanel/douban-sync-action — GitHub Action，每日同步
- https://github.com/caspartse/eBooksAssistant — 135 stars，Python
- https://github.com/doubaniux/DoubanSpider — Scrapy + PostgreSQL
- https://github.com/yekingyan/Backup-Douban-Broadcast — 18 stars，HTML
- https://github.com/shin000000/backup_your_douban_broadcast — Python，2022
- https://github.com/zhuth/DoubanDiaryBackup — C#，已归档
- https://github.com/sprhawk/DoubanBackup — Python，2014
- https://github.com/wuyuntao/douban-offline — Greasemonkey，2008
- https://github.com/zhixwang/Douban_backup — Python，2020

## 5. 浏览器扩展（Chrome Webstore）

- https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3notion%E5%90%8C%E6%AD%A5%E5%8A%A9%E6%89%8B/cephkdcfcpdppcdjdhfnmphkoenpkhna — 豆瓣 Notion 同步助手
- https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3%E5%90%8C%E6%AD%A5%E5%88%B0%E9%A3%9E%E4%B9%A6/lijkkmbhiffcjpogjglndfahjachadon — 豆瓣同步到飞书
- https://chrome.google.com/webstore/detail/%E8%B1%86%E4%BC%B4%EF%BC%9A%E8%B1%86%E7%93%A3%E8%B4%A6%E5%8F%B7%E5%A4%87%E4%BB%BD%E5%B7%A5%E5%85%B7/ghppfgfeoafdcaebjoglabppkfmbcjdd — 豆伴
- https://doufen.org/ — 豆伴官网（Chrome 扩展备份工具，2021 年起停更）

## 6. 用户脚本（Greasyfork）

- https://greasyfork.org/en/scripts/by-site/douban.com — 豆瓣相关脚本汇总（约 100 条，多为增强/去广告/备份）
- https://greasyfork.org/en/scripts/468962-doubanresize — 图片尺寸调整
- https://greasyfork.org/en/scripts/449861-linkdoubantrakt — 豆瓣影视 ↔ Trakt 链接
- https://greasyfork.org/en/scripts/520137-douban-book-auto-sort — 书单自动排序

## 7. Playwright / 浏览器自动化参考

- https://github.com/Apauto-to-all/spiderDoubanTvAndExplore — Playwright 爬榜单
- https://github.com/crifan/web_automation_tool_playwright — crifan Playwright 教程书

## 8. 第三方 iPaaS / SaaS（均无豆瓣）

- https://ifttt.com/explore/applets — IFTTT，无豆瓣
- https://ifttt.com/explore — 同上
- https://zapier.com/apps/search?q=douban — 404
- https://www.make.com/en/integrations?query=douban — 无匹配
- https://www.coze.cn/store/plugin?query=%E8%B1%86%E7%93%A3 — 扣子插件商店，无结果

## 9. RSS 与跨平台

- https://docs.rsshub.app/ — RSSHub 主站
- https://rsshub.app/douban/group/camera — 豆瓣小组路由示例
- https://wzfou.com/rsshub/ — RSSHub 中文搭建教程

## 10. MCP / LLM 工具集成

- https://www.npmjs.com/package/mcp-douban-server — 只读 MCP server
- https://www.mcp-gallery.jp/mcp/github/moria97/douban-mcp — moria97/douban-mcp

## 11. 社区实证 / 帖子

- https://www.v2ex.com/t/699393 — 豆瓣 API key 失效讨论

## 12. 打码平台（极验 v3 支持）

- https://2captcha.com/2captcha-api#geetest — 2Captcha GeeTest 支持文档
- https://www.chaojiying.com/ — 超级鹰（中文打码平台）
- https://www.yescaptcha.com/ — YesCaptcha

## 13. 相关杂项 / 封号案例（旁证）

- https://u.osu.edu/mclc/2021/12/10/douban-pulled-from-app-stores/ — 豆瓣 2021 下架监管讯息
- https://gigazine.net/gsc_news/en/20220429-douban-chaotic-chinese-social-network/ — 豆瓣 2022 言论封号讯息

---

**外部引用统计**：本文档含 **35** 条以上可点击外部链接，加上主报告 `REPORT.md` 的 33 条脚注，总引用 > 60 条。
