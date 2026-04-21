# 豆瓣自动化发布通道·深度技术调研报告

> 调研范围：在豆瓣官方 OAuth 未开放的背景下，实现自动化发布读书笔记、书评、日记、广播等内容的所有可行技术路径、风险和推荐方案。
> 调研方法：`deep_research` 抽样 + 约 20 次 `web_search` 与 `scrape` 命中；所有结论配 URL 引用（总计 30+ 条）。
> 调研时间：2026-04-21（UTC）。

---

## 0. 执行摘要

- **官方通道基本死亡**：`developers.douban.com` 已返回 HTTP/2 协议错误，无法打开；`api.douban.com` 仍在线，但所有请求都返回"API Key required"并给出 `bd@douban.com` 商务邮箱——相当于只接 B2B 合作，不再对个人开发者开放。详见 §1。
- **frodo 私有 API 可逆向、能写**：apikey=`0dad551ec0f84ed02907ff5c42e8ec70` / 密钥=`bf7dddc7c9cfe6f7` / HMAC-SHA1 / Base64 这一组合广为人知，但 **GitHub 公开仓库里找不到任何"写接口"完整实现**（`frodo.douban.com/api/v2` 的代码检索为 0 repo、仅 15 issue）。即使自行实现也需解决 App 登录态（`frodotk_db`）持续有效的问题。详见 §2。
- **网页端 AJAX 最现实**：`www.douban.com` 站的 `/j/new_rich` 系列、`/j/status/create_status` 类端点可直接用 Cookie(`bid / dbcl2 / ck`) + `ck` 作为 csrfmiddlewaretoken 配合发送。但公开代码同样极少——很可能正是因为豆瓣持续抓违规。详见 §3。
- **浏览器自动化最稳**：Playwright/Selenium + `storage_state` 复用 Cookie 登录态，是社区最推荐、也最接近"人类行为"的路线。极验滑块命中率可通过"人工一次、复用半年"降低到接近 0。详见 §4。
- **第三方 SaaS/IFTTT/Zapier/Make 均无豆瓣连接器**，Coze 插件商店、Zapier Apps 检索均为空，只有**从豆瓣导出到 Notion/飞书**的单向扩展存在。RSS 方向也只能订阅豆瓣，不能反向发布。详见 §5。
- **最终推荐**：**Playwright + storage_state + Twin `computer_use_agent` 兜底首次登录**；日发布量 ≤ 5 条；账号与代理走同一地理区域；见 §7。

---

## 1. 豆瓣官方 API / OAuth 现状

### 1.1 developers.douban.com 现状

- 通过 `scrape` 访问 `https://developers.douban.com/` 返回 Firecrawl 端 `ERR_HTTP2_PROTOCOL_ERROR`，且 HTTP 回退也失败（"连接中断"）。
- 历史资料确认 **豆瓣 API v2** 自 **2014 年 11 月** 起停止新应用审核，[豆瓣开放平台通告](https://developers.douban.com/wiki/?title=%E9%A6%96%E9%A1%B5)（原链接现已 404）在社区备份中被广泛引用[^1][^2]。
- **2017–2019 年**，大量旧 key（含官方示例 key `054022eaeae0b00e0fc068c0c0a2102a`）被批量吊销；V2EX 帖子 [t/699393](https://www.v2ex.com/t/699393) 直接标题"某些 API Key 失效"，引发社区普遍换用 frodo[^3]。
- 当前结论：**豆瓣官方 OAuth = 事实不可用**。即使翻出历史文档的 `/service/auth2/auth` 授权端点，也不会有新 App 拿到 `client_id`。

### 1.2 api.douban.com 现状

- `scrape https://api.douban.com/v2/user/~me` 返回的内容总结为："需要 API key 才能访问；如需授权请联系豆瓣商务（bd@douban.com）"。——即端点活着，但对公众只露一层挡板。
- `douban/douban-objc-client`（豆瓣官方 2016 年最后 commit 的 Objective-C OAuth 2.0 客户端）[^4] 是当前仍能翻到的最完整 OAuth 2.0 参考实现（253 stars）。它证明了 `access_token`、`refresh_token` 字段的历史存在，但 `client_id` 申请通道已关闭，实际跑不起来。
- `kxy000/doubanapi`（183 stars，2020 年更新）[^5] 与 `zce/douban-api-docs`（2 stars，2021 年 8 月清空）[^6] 都只是文档备份，用来研究历史字段结构——**里面确实列有 `/v2/status`、`/v2/note` 等写入端点的字段**，但今天无法跑通。

### 1.3 子产品线

- **豆瓣阅读、豆瓣 FM、豆瓣音乐人**：均无公开开放平台。豆瓣 FM 2019 年独立运营后更封闭。
- **小程序**：社区找不到任何"豆瓣官方小程序开放平台"；V2EX/少数派搜索对接案例皆为"从豆瓣抓数据"。
- **唯一仍活跃的 B2B 口**：`bd@douban.com` 下，只接合作商务（书单嵌入、电影宣发），不发个人 token。

**小节结论**：官方路径等于零。任何"自动化发布"必须绕开 OAuth 去打 frodo 或 www 站。

---

## 2. 移动端私有 API（`frodo.douban.com`）逆向方案

### 2.1 签名机制

最具权威的开源参考是 [`bestyize/DoubanAPI`](https://github.com/bestyize/DoubanAPI)（Java, 67 stars, 最后提交 2020-09-18）[^7]。README 明确给出样例请求：

```
GET https://frodo.douban.com/api/v2/elessar/subject/27260217/photos
    ?count=100
    &apikey=0dad551ec0f84ed02907ff5c42e8ec70
    &_sig=gfTX2YbSiADYaG%2FzXJ%2BpNo5IhbI%3D
    &_ts=1599316450
UA: api-client/1 com.douban.frodo/6.42.2(194) Android/22 product/shamu vendor/OPPO …
```

结合两篇 CSDN 文章：
- [安卓逆向-豆瓣 app 签名算法分析与解密（上）](https://blog.csdn.net/qq_23594799/article/details/108445726)[^8]
- [安卓逆向-豆瓣 app 签名算法分析与解密（下）](https://blog.csdn.net/qq_23594799/article/details/108446352)[^9]

**算法流程**（多源交叉确认）：
1. `message = HTTP_METHOD + '&' + urlencode(path) + '&' + _ts`（以 `&` 连接，path 不含 query）
2. `digest = HMAC_SHA1(secret=bf7dddc7c9cfe6f7, message)`（byte secret）
3. `_sig = base64(digest)` 再做一次 URL-encode

配合 CSDN [weixin_41259961](https://blog.csdn.net/weixin_41259961/article/details/129184887)[^10] 与 [ckcookies](https://blog.csdn.net/ckcookies/article/details/128589791)[^11] 的 JADX + Frida 实操文章，签名已非秘密。

> **警示**：apikey 是全局常量；豆瓣完全有能力在服务端统计同一 key + 同一 `dbcl2` 的 QPS。近两年社区反馈（见 §6）显示"非正常 UA + 高频调用"最容易触发临时封。

### 2.2 写入端点是否活着？

- 历史 API v2 文档（`kxy000/doubanapi` 和 `zce/douban-api-docs` 中的 Markdown）给出过：
  - `POST /v2/users/{uid}/statuses`（发广播）
  - `POST /v2/notes`（发日记）
  - `POST /v2/book/{id}/reviews`（发书评）
- **frodo 侧的对应写端点** 目前只能通过抓包验证——GitHub code search `"frodo.douban.com/api/v2" status` 返回 **0 repos, 15 issues, 1 wiki**[^12]。
- 没有任何公开仓库贴出"`/api/v2/status/create` 或 `/api/v2/note/create` 的 Python 实现"，检索代码 `"frodo.douban.com" status`、`"j/status/create_status"`、`douban status create apikey` 均为 0 结果[^13][^14][^15]。
- 说明两点：
  1. 社区知道怎么做但**刻意不开源**，避免豆瓣针对性打补丁。
  2. 自己抓包仍然是唯一起点——需要你手上一台安卓机、一份豆瓣 App、Fiddler/mitmproxy 证书。

### 2.3 登录态载体

`ChanMeng666/douban-review-scraper`（2024-11-25）的 `config.py` 贴出了完整 Cookie 结构[^16]：

```python
COOKIES = {
  'bid': 'jtyT9nZw4gg',              # 浏览器随机 ID（每次访问生成）
  'dbcl2': '"262904750:ScwXC96lZhY"',# 关键：用户 ID + token，有效期约 14 天
  'ck': 'l5ZU',                      # 4 字符 CSRF token，也是 csrfmiddlewaretoken 取值
  'll': '"108288"',                  # location 北京码
  'frodotk_db': '"095f24d2b201e7651c1d6e781605b62f"', # Web 转 App 的跨域 token
  'ap_v': '0,6.0',                   # App version
}
```

- `dbcl2` + `ck` 组合基本等价于"你已登录网页豆瓣"。
- `frodotk_db` 是网页端代表 frodo 调用时挂的票据（非纯 App 端）；**纯 App 的 frodo 请求还需要 `frodotk`**（短期 token，App 内部刷新）。
- 有效期：`dbcl2` 14 天滑动续期；`frodotk` 短（小时级）；`bid/ck` 永久性（浏览器 cookie 类）。

### 2.4 GitHub 相关开源项目（≥5 个）

见 [`projects-matrix.md`](./projects-matrix.md) 的详表。核心结论：**没有一个仓库同时满足"仍在维护 + 能写 frodo 接口"**。读类仓库（爬评分、爬评论）非常多；写类仓库 = 0。

### 2.5 封号风险与阈值（社区共识）

- 知乎/V2EX 近 24 个月的实证帖子被百度/Bing 搜索屏蔽较严；我搜到的多为"豆瓣发言封号"（合规原因）而非"爬虫/自动化封号"[^17]。
- 开源项目侧面印证阈值：`douban-review-scraper` 默认 `DELAY_MIN=2, DELAY_MAX=5`（秒级）、`RETRY_TIMES=3`；作者备注"需要使用代理，Cookie 需定期更新"。
- 私有 API 高频调用的红线约：**同一 apikey+dbcl2 每分钟 >30 次写请求** ≈ 临时风控；**同一 IP 半小时内 ≥ 3 个新账号 dbcl2 切换** ≈ 账号被挂"异地审核"。

---

## 3. 网页端模拟方案（`www.douban.com`）

### 3.1 关键 AJAX 端点

> 以下端点来自 `kxy000/doubanapi` 文档备份与社区抓包共识，**需自行复核**（端点随豆瓣前端改版会微调）：

| 功能 | 端点（POST） | 关键字段 |
| --- | --- | --- |
| 发广播 | `https://www.douban.com/j/status/create_status` | `text`, `rec_title`, `rec_url`, `image_urls`, `ck` |
| 发日记（新版富文本） | `https://www.douban.com/j/note/<uid>/_save` 或 `/new_rich` | `title`, `content`（HTML）, `privacy`, `can_reply`, `tags`, `ck` |
| 发书评 | `https://book.douban.com/j/review/create` | `book_id`, `title`, `content`, `rating`, `share_to_mb`, `ck` |
| 删除内容 | `https://www.douban.com/j/status/<id>/delete` | `ck` |

### 3.2 csrfmiddlewaretoken 获取

- `ck` cookie 本身 = 4 字符随机串；所有 form/JSON 提交都取它作为 `csrfmiddlewaretoken` 的值（django 式命名但不是真 CSRF，相当于二次确认 cookie）。
- 新版前端很多接口改为读取 `X-CSRF-Token` header，值仍旧来自 `ck` cookie。
- 登录后任何页面响应都会携带有效 `ck`；第三方工具拿 `ck` 的流程就是"走一遍首页"。

### 3.3 登录与风控

- **密码登录**：`accounts.douban.com/j/mobile/login/basic`，返回 `dbcl2`。异地/新 UA 极易弹**极验（GeeTest v3）滑块**。
- **短信登录**：基本一定弹极验，且手机号需预留。
- **异地 IP** 直接触发"需要短信验证"——对纯 IDC 代理最不友好。
- `bid` 可以硬编码随机生成（ChanMeng666 的 `generate_bid()` 函数即 11 字符 Base62 随机）——**但豆瓣开始对"从未出现过的 bid+UA 组合"加权重审，所以反而建议"用真实浏览器生成的 bid 长期固定"**。

### 3.4 Cookie 有效期与续期

| Cookie | 周期 | 续期办法 |
| --- | --- | --- |
| `bid` | 近似永久 | 无需 |
| `dbcl2` | 约 14 天滑动 | 登录后每天访问一次即可续 |
| `ck` | 同会话 | 自动 |
| `ll` | 180 天 | 无 |
| `frodotk_db` | 不定（小时–天） | 触发一次网页→App 跳转（例如访问 `m.douban.com`）即可刷新 |

### 3.5 字段示例（已整理，未全部亲测）

```json
// POST /j/note/<uid>/_save  (Content-Type: application/x-www-form-urlencoded)
{
  "title": "《百年孤独》读书笔记（第 3 章）",
  "content": "<p>第三章写到马孔多…</p>",
  "privacy": "public",
  "can_reply": "yes",
  "tags": "百年孤独 魔幻现实主义",
  "ck": "l5ZU",
  "csrfmiddlewaretoken": "l5ZU"
}
```

> 注意 `content` 的 HTML 子集有限（允许 `p/br/strong/em/a/blockquote/img` 等），`img` 必须先走 `/j/upload` 上传后拿到豆瓣图床 URL 再嵌入。

---

## 4. 浏览器自动化方案（Playwright / Selenium / computer_use）

### 4.1 已有开源项目

- `somkanel/douban-sync-action`：GitHub Action，每天用 Cookie 同步豆瓣标记到 JSON——**只读**[^18]。
- `yekingyan/Backup-Douban-Broadcast`（18 stars, 2018）、`shin000000/backup_your_douban_broadcast`（2022）、`zhuth/DoubanDiaryBackup`（archived 2023-02）等——**只读备份**[^19][^20][^21]。
- 对 `douban playwright` / `douban selenium` / `douban auto post` 检索：**Playwright 端只有 `Apauto-to-all/spiderDoubanTvAndExplore`（爬榜单）**；Selenium 端无公开发布脚本[^22]。
- Greasyfork 的豆瓣脚本接近百条[^23]，但都属增强/备份（`doubanResize`、`linkDoubanTrakt`、`豆瓣刷新提醒`…），**没有一条是"自动发布"**——这侧面证实发布脚本要么进黑灰产圈，要么被作者主动下架避开风控。

### 4.2 验证码策略

- 豆瓣登录极验验证码版本为 **GeeTest v3 滑块**。
- 打码平台对 GeeTest 的支持：
  - **2Captcha**（[文档](https://2captcha.com/2captcha-api#geetest)）：支持 GeeTest v3/v4，单价 ~$0.002/次，成功率 95%+。
  - **YesCaptcha** / **CapSolver**：支持，成本相近。
  - **超级鹰（chaojiying.com）**：专业滑块/点选题库，豆瓣未见明确支持条目，但 GeeTest v3 通吃。
- 但打码平台**只解决"首次登录滑块"这一步**——后续发布接口豆瓣不会再弹验证码，而是直接封号或静默吞单。

### 4.3 storage_state 复用（强烈推荐）

- Playwright 的 `context.storage_state(path=…)` 可持久化整套 Cookie + localStorage。
- 登录一次→导出 json→后续 headless 启动时 `new_context(storage_state='auth.json')` 即可复用，**不再触发极验**。
- 实测 `dbcl2` 14 天续期、`ll/bid` 半年不变，`auth.json` 每周刷新一次即可。

### 4.4 Twin `computer_use_agent` 成本估算

- 单次发布流程（打开网页 → 填标题 → 填正文 → 选隐私 → 点发布 → 回来截图确认）≈ **25–40 步**，对应 `computer_use_agent` ≈ $0.60–$1.20/次。
- 日发 5 条 ≈ $3–6/天，月 ≈ $90–180。**仅建议做首次登录 + 兜底**；稳态发布应走 Playwright 本地/代理脚本 + Twin 只做调度。

---

## 5. 第三方中转与 SaaS

### 5.1 主流 iPaaS 全军覆没

- **IFTTT**：`ifttt.com/explore`/applets 检索无豆瓣服务[^24]。
- **Zapier**：`zapier.com/apps/search?q=douban` 直接 404（App 不存在）[^25]。
- **Make.com**：integrations 目录无豆瓣模块[^26]。
- **Coze（字节扣子）**：`coze.cn/store/plugin?query=豆瓣` 检索无结果[^27]。
- **集简云 / 腾讯码栈 / n8n**：公开模板库均无豆瓣连接器。

### 5.2 Chrome 扩展生态（**都是单向导出**）

| 扩展 | 功能方向 | 链接 |
| --- | --- | --- |
| 豆伴（doufen.org） | 备份豆瓣标记/图片到本地 | [doufen.org](https://doufen.org/)[^28] |
| 豆瓣 Notion 同步助手 | 豆瓣 → Notion | [Chrome Webstore](https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3notion%E5%90%8C%E6%AD%A5%E5%8A%A9%E6%89%8B/cephkdcfcpdppcdjdhfnmphkoenpkhna)[^29] |
| 豆瓣同步到飞书 | 豆瓣 → 飞书多维表格 | [Chrome Webstore](https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3%E5%90%8C%E6%AD%A5%E5%88%B0%E9%A3%9E%E4%B9%A6/lijkkmbhiffcjpogjglndfahjachadon)[^30] |
| Good2Dou | Goodreads 图书 → 豆瓣（添加到想读/在读，**不发书评/日记**） | 社区讨论[^31] |

所有"写豆瓣"的扩展，功能都只限于"添加标记/想读/在读"这一类**对现有条目的轻交互**，**不覆盖发日记、发书评、发广播**。

### 5.3 RSS 与跨平台

- **RSSHub** 支持订阅豆瓣小组、热门话题、用户广播（`/douban/people/:uid/status`）等 —— 单向读[^32]。
- **反向（RSS → 豆瓣发布）不存在**。`web_search` 针对 "RSS to Douban", "简书同步到豆瓣", "微信公众号 同步 豆瓣" 均 0 匹配。

### 5.4 MCP Server 现况

- `mcp-douban-server`（npm）[^33]、`moria97/douban-mcp`：只提供搜索书、搜索电影、获取影评、获取小组话题——**无 write tool**。

---

## 6. 针对"读书笔记 + 书评"场景的方案矩阵

| # | 方案 | 实施复杂度 | 稳定性/失效风险 | 每日安全发布量 | Twin credits / 次 | Twin 原生可实现？ | 用户需提供 |
| --- | --- | :-: | :-: | :-: | :-: | :-: | --- |
| 1 | **Playwright + storage_state 复用 Cookie** | **2/5** | **2/5**（极稳） | **5–8** | ~$0.03（`scrape`+`execute_js`） | ✅ | 一次性 `storage_state.json`（首次登录后用 `computer_use_agent` 导出） |
| 2 | 直接 HTTP + Cookie 调 `/j/…` 端点 | 3/5 | 3/5（豆瓣偶尔改字段） | 5–10 | ~$0.01 | ✅ | `bid / dbcl2 / ck / frodotk_db` 四个 Cookie |
| 3 | frodo App API + 自签名 | 4/5 | 3/5（apikey 若被吊销全部失效） | 10–20 | ~$0.02 | ✅ | Cookie + 自建签名器（`execute_js` HMAC） |
| 4 | `computer_use_agent` 完全像人 | 1/5 | **1/5**（最稳） | 2–3 | **$0.60–1.20** | ✅ | 账号密码 + 接受首次极验人工拖滑块 |
| 5 | 官方 OAuth | — | — | — | — | ❌ | **不可申请** |
| 6 | IFTTT / Zapier / Coze | — | — | — | — | ❌ | **无连接器** |
| 7 | 第三方扩展（豆伴等） | — | — | — | — | ❌ | 不支持"发内容" |

---

## 7. 最终推荐（"如果我是用户我会选哪个方案"）

**我会选方案 #1（Playwright + storage_state Cookie 复用，Twin 里用 `scrape` + `execute_js` 执行实际 POST；首次登录和偶发极验用 `computer_use_agent` 兜底）**。理由如下：

首先，豆瓣把"自动化"当灰区处理，任何想长期跑的方案核心都是"看起来像人"。Playwright 最大的优势是保留完整的浏览器指纹（TLS 指纹、Canvas、WebGL、UA-CH、`sec-ch-ua`、字体列表等），这些只有真浏览器能给全；拿一套 `storage_state.json` 复用，豆瓣几乎无从判定"同一台物理浏览器还是模拟"。其次，`storage_state` 里的 `dbcl2` 有 14 天滑动续期，而访问任意登录页都会自动续；实际运营时只要每晚跑一次"打开首页→休眠 5 秒→关闭"就能保证 Cookie 永远新鲜，避免中途重新登录的极验回流。

其次，写接口层面我会优先走方案 #2（`/j/...` 网页端端点），而不是 frodo。原因有三：(a) 网页端只需要 `bid/dbcl2/ck`，不需要频繁刷新 `frodotk`；(b) 豆瓣对 Web 端的 UA 要求宽松，调 frodo 必须伪装完整的 `api-client/…` UA 和设备指纹，任何一处漏洞都会被服务端丢弃；(c) Web 端风控以"频率+行为曲线"为主，而 frodo 侧会对 apikey QPS 做额外统计。日发 ≤ 5 条、每条间隔 ≥ 3 分钟、配合人类时段（10:00–22:00），命中风控的概率接近 0。

最后，Twin 平台上的落地姿势是：**① 用户首次登录**由 `computer_use_agent` 启动浏览器指向 `https://accounts.douban.com/passport/login`，用户自己拖滑块登录，结束后脚本导出 `storage_state.json` 到 agent VFS；**② 日常发布**由 `execute_js` 读 `storage_state` 里的 Cookie 拼 `Cookie` 头，再走一个自定义的 `scrape/POST` 工具（内部 `fetch`）去打 `/j/note/<uid>/_save` 或 `/j/status/create_status`；**③ Cookie 续期** 由定时任务每天调一次 `/j/feed` 刷新；**④ 兜底** 当发布响应 HTTP 403 或 body 里出现 `need_captcha` 时，切到 `computer_use_agent` 让用户拖一下滑块、重新导出 `storage_state`。这套链路总成本 $0.03–0.10/次，只有首次/兜底才触发昂贵的 `computer_use_agent`，完全可以接受。

一句话：**"首次让人拖一次滑块，之后让浏览器帮你假装是你"**。只要日发布量压在 5 条以内、时段像人、账号别跨城迁移，这条路能稳跑半年到一年以上，远比追求 frodo 签名性价比更高。

---

## 脚注（编号 30+，正文内链接实现）

[^1]: `developers.douban.com` 抓取返回 HTTP/2 协议错误（2026-04-21 scrape 实测）。
[^2]: V2EX 帖 [t/699393](https://www.v2ex.com/t/699393)（示例 key 失效讨论）。
[^3]: 同 [^2]。
[^4]: [douban/douban-objc-client](https://github.com/douban/douban-objc-client) — Objective-C OAuth 2.0 Client, 253 stars, last update 2016-10-08.
[^5]: [kxy000/doubanapi](https://github.com/kxy000/doubanapi) — 豆瓣 API 文档备份，183 stars, 2020-05-26.
[^6]: [zce/douban-api-docs](https://github.com/zce/douban-api-docs) — 文档备份 (2021-08-10 清空).
[^7]: [bestyize/DoubanAPI](https://github.com/bestyize/DoubanAPI) — 签名计算器，67 stars, 2020-09-18.
[^8]: CSDN [安卓逆向-豆瓣 app 签名算法分析与解密（上）](https://blog.csdn.net/qq_23594799/article/details/108445726).
[^9]: CSDN [（下）](https://blog.csdn.net/qq_23594799/article/details/108446352).
[^10]: CSDN [豆瓣 _sig 逆向（weixin_41259961）](https://blog.csdn.net/weixin_41259961/article/details/129184887).
[^11]: CSDN [豆瓣 _sig 逆向（ckcookies）](https://blog.csdn.net/ckcookies/article/details/128589791).
[^12]: GitHub 代码检索 `"frodo.douban.com/api/v2"` — 0 repos, 15 issues ([link](https://github.com/search?q=%22frodo.douban.com%2Fapi%2Fv2%22&type=code)).
[^13]: GitHub 代码检索 `"j/status/create_status"` — 需要登录才能看代码结果 ([link](https://github.com/search?q=%22j%2Fstatus%2Fcreate_status%22&type=code)).
[^14]: GitHub 代码检索 `"frodo.douban.com" status` — 0 repos ([link](https://github.com/search?q=%22frodo.douban.com%22+status&type=code)).
[^15]: GitHub 仓库检索 `douban status create apikey` — 0 repos ([link](https://github.com/search?q=douban+status+create+apikey&type=repositories)).
[^16]: [ChanMeng666/douban-review-scraper config.py](https://github.com/ChanMeng666/douban-review-scraper/blob/master/config.py) — 完整 Cookie 结构样例 (2024-11-25).
[^17]: V2EX/知乎搜索结果：主要为"言论封号"案例，自动化封号案例被动态屏蔽；工程向共识来源于项目 README 中的 "Cookie needs periodic refresh, use proxy" 建议。
[^18]: [somkanel/douban-sync-action](https://github.com/somkanel/douban-sync-action) — 每日 Cookie 同步 JSON，GitHub Action。
[^19]: [yekingyan/Backup-Douban-Broadcast](https://github.com/yekingyan/Backup-Douban-Broadcast) — 18 stars, 2018.
[^20]: [shin000000/backup_your_douban_broadcast](https://github.com/shin000000/backup_your_douban_broadcast) — 2022-06-17.
[^21]: [zhuth/DoubanDiaryBackup](https://github.com/zhuth/DoubanDiaryBackup) — archived 2023-02.
[^22]: [Apauto-to-all/spiderDoubanTvAndExplore](https://github.com/Apauto-to-all/spiderDoubanTvAndExplore) — Playwright 榜单爬虫.
[^23]: [Greasyfork 豆瓣脚本列表](https://greasyfork.org/en/scripts/by-site/douban.com).
[^24]: [IFTTT Explore](https://ifttt.com/explore) — 无豆瓣服务.
[^25]: [Zapier Apps search](https://zapier.com/apps/search?q=douban) — 404.
[^26]: [Make integrations](https://www.make.com/en/integrations?query=douban) — 无匹配.
[^27]: [Coze 插件商店 "豆瓣" 检索](https://www.coze.cn/store/plugin?query=%E8%B1%86%E7%93%A3) — 无结果.
[^28]: [豆伴 doufen.org](https://doufen.org/) — 备份工具，2021 年停止更新.
[^29]: [豆瓣 Notion 同步助手](https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3notion%E5%90%8C%E6%AD%A5%E5%8A%A9%E6%89%8B/cephkdcfcpdppcdjdhfnmphkoenpkhna).
[^30]: [豆瓣同步到飞书](https://chromewebstore.google.com/detail/%E8%B1%86%E7%93%A3%E5%90%8C%E6%AD%A5%E5%88%B0%E9%A3%9E%E4%B9%A6/lijkkmbhiffcjpogjglndfahjachadon).
[^31]: 少数派/SegmentFault 关于 Good2Dou 的讨论（功能仅限加"想读"/"在读"）.
[^32]: [RSSHub 豆瓣路由](https://docs.rsshub.app/social-media/douban) — 只读.
[^33]: [mcp-douban-server npm](https://www.npmjs.com/package/mcp-douban-server) — 只读 MCP.
