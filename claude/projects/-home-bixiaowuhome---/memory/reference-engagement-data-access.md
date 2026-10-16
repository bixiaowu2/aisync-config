---
name: engagement-data-access
description: 互动数据的可读通道 —— 币安广场读不到、X 官方 API 要付费额度，只有 X embed 接口免费可读
metadata:
  node_type: memory
  type: reference
  originSessionId: 1eaaa207-154d-499d-837b-685c9803c7f8
  modified: 2026-10-15T23:36:56.561Z
---

想拿"帖子表现"做数据反馈时，三条通道的实测结论（2026-10-15/16）：

| 通道 | 结论 |
|---|---|
| **币安广场 OpenAPI** | **只有发布端点**。`/content/list`、`/content/detail`、`/content/statistics`、`/content/history` 等 14 个候选全 **404**。技能文档也明说"只支持发布"。 |
| **币安广场网页版** | **读不到**。`/en/square`、帖子页、用户主页一律 **CloudFront 403**（用真实浏览器 + mihomo 代理也一样）。curl 则返回 HTTP 202 空响应（反爬挑战）。 |
| **X 官方 API** | 凭据可用（`get_me` 成功），但**读推文要付费额度**：`get_users_tweets` 报 **HTTP 402 `credits depleted`**（不带 user_auth 是 401）。 |
| **X 单条推文 embed 接口** | ✅ **免费可读，无需登录**：`https://cdn.syndication.twimg.com/tweet-result?id=<推文ID>&token=a` 返回权威 `favorite_count`/`conversation_count`/`created_at`/`text`。 |
| **X 游客视图** | 能读但**硬性封顶 5 条**，滚到底就是登录墙；搜索完全不可用。 |

**所以**：广场的浏览/互动数**只能用户自己在 App 里看**；X 的互动可以自动跟 —— 用游客视图取帖子 ID + embed 接口取精确数值。

**已装的工具**（都在服务器上）：
- `/opt/bn-docs/tools/analyze_tweet.py` —— 给推文链接即出全文 + 精确互动 + 结构拆解
- `/opt/bn-docs/tools/verify_x_handles.sh` —— 批量核验 handle 是否真实可抓（见 [[kol-handle-verification]]）
- `/opt/x-engagement/track.py` + `x-engagement.timer` —— 每 6 小时跟踪用户 X 号（@bidafa1801919）新帖互动，存成时间序列

相关：[[x-monitor-two-purposes]] [[radar-ops-constraints]]
