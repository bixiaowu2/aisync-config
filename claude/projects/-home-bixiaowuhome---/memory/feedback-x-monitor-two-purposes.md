---
name: x-monitor-two-purposes
description: x-monitor 的 4 个核心号是跟踪币安 Alpha 信号用的，与 KOL 语料号用途不同，不可混用
metadata:
  node_type: memory
  type: feedback
  originSessionId: 1eaaa207-154d-499d-837b-685c9803c7f8
  modified: 2026-10-15T12:55:51.986Z
---

x-monitor 里有两类完全不同的号，**目的、渠道、配置都必须分开**：

- **核心号**（`X_ACCOUNTS` = binancezh, cz_binance, heyibinance, binancewallet）：用途是**跟踪币安 Alpha 信息**，产出是钉钉/Telegram 信号。
- **KOL 语料号**（`X_EXTRA_ACCOUNTS_FILE` → `x_accounts.json`）：用途是**积累写 X 文章的语料/学风格**，产出只进 `corpus` 表。

**Why:** 2026-10-15 我在扩充语料号时，用户主动纠正："x-monitor那4个号用来跟踪币安alpha信息的，不是用来写X文章的。用途目的不同" —— 担心我把信号跟踪和发文取材混为一谈，进而动到核心号。

**How to apply:**
- 改发帖/选题/风格相关的东西时，**不要动核心号的配置和投递**。
- KOL 语料号一律在 `routes.json` 里显式写 `[]`（= 只存档不推送）。**绝不能让它们继承 `global_targets()`** —— 那里面有钉钉，会直接打爆 3000/月配额（见 [[radar-ops-constraints]]）。
- 注意 `targets_for()` 的 fallback 链是 `routes[账号] → routes["*"] → global_targets()`，所以"没配路由"= 走钉钉，是个危险的默认值。
- 相关：[[kol-handle-verification]]
