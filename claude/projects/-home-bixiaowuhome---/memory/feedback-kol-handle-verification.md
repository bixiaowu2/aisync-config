---
name: kol-handle-verification
description: X handle 清单会腐坏（实测 25% 是坏号），任何来源给的 handle 都必须用浏览器实测核验
metadata:
  node_type: memory
  type: feedback
  originSessionId: 1eaaa207-154d-499d-837b-685c9803c7f8
  modified: 2026-10-15T12:56:06.686Z
---

**规则：任何来源（GitHub awesome 列表、Reddit 帖、KOL 榜单、我自己的记忆）给出的 X handle，都必须实测核验后才能入注册表。**

**Why:** 2026-10-15 实测 `x_accounts.json` 里 20 个 handle，**5 个是坏的（25%）**：

| handle | 实测真相 |
|---|---|
| `spotonchain` | 已迁移，只剩提示页 → 真号 `hupzy_agent` |
| `ansem` | HTTP 404，号不存在 → 真号 `blknoiz06`（140 万粉） |
| `murad` | **冒牌号**（804 粉）→ 真号 `muststopmurad` |
| `thedatanerd` | 账号已被 X 封禁 |
| `arkhamintel` | 受保护账号（仅批准关注者可见），永远抓不到 |

最坑的是：**坏 handle 不会报错**。它只是静默返回 0 条卡片，健康检查里只记一个 `RuntimeError`，
和"这个号最近没发帖"在表面上无法区分 —— 不专门核验就会一直以为在正常工作。

**How to apply:**
- 核验方法（已跑通）：把 `browser-profile` 复制一份到 `/tmp/probe-profile`（**必须复制**，
  正在运行的服务占着原 profile，Chrome 的 ProcessSingleton 会拒绝并发），删掉 `Singleton*` 文件，
  然后用 Playwright 打开 `x.com/<handle>`，看三样：
  1. HTTP 状态（404 = 号不存在）
  2. `page.locator("article").count()`（0 = 抓不到）
  3. 正文里有没有 `Account suspended` / `These posts are protected` / `has been migrated to`
- 判**冒牌号**看粉丝数和 bio（真号特征明显：Murad "Crypto Investor since 2013"、Ansem 140 万粉）。
- 别只信"能打开"：`tetranode` 能打开但只有 733 粉，是抢注号。
- 相关：[[x-monitor-two-purposes]]
