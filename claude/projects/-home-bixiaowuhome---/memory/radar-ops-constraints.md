---
name: radar-ops-constraints
description: Oracle 雷达的四条硬约束：钉钉企业配额、X 手动发帖、meme 不发币安广场、币安 API 凭据（Web3 API 因服务器在东京而用不了）
metadata:
  node_type: memory
  type: project
  originSessionId: 1eaaa207-154d-499d-837b-685c9803c7f8
  modified: 2026-10-16T12:07:03.660Z
---

在这套雷达上做推送/发帖相关改动时，先确认这三条约束：

**1. 钉钉有企业级月配额，会耗尽。**
钉钉开发者后台「Webhook 和 Stream 用量」上限 **3000 次/月**（企业级，所有机器人和 Stream 长连接共用）。
2026-10 月初就是被**旧的 `alpha-radar.service`** 打爆的——它的通知 worker 是 **1 秒循环**、每轮都调 `dingtalk.deliver_one()`，`dingtalk_outbox` 每天入队 3400–3900 条，累计 `sent` 4656 条。
**Why**：这批发送结果**只写数据库、不写 journald**，所以按系统日志统计会完全漏掉它（我就因此误判过一次）。
**How to apply**：判断钉钉用量时不要只看 journald，要查 sqlite 的 `dingtalk_outbox`（历史备份在 `/var/lib/radar-maintenance/backups/alpha-daily-*.sqlite`）。配额耗尽后所有机器人返回 `errcode 90030`，下月 1 日才重置。现役雷达已把钉钉改成小时汇总降频。

**2. X 发帖是手动的，不是自动。**
用户没加支付卡，X API 免额度已用完。`/opt/alpha-radar/x_publisher.py` 的 `post_tweet()` **只发单条、没有 `in_reply_to_tweet_id`**，所以 Thread 串一直是人工复制粘贴的。
**How to apply**：不要假设能自动串推。草稿生成后推 Telegram 给用户手动发即可。

**3. meme 币不发币安广场，只发 Telegram + X。**
**Why**：meme 多是未上币安的 DEX 代币 → cashtag 无法归因，内容挖矿拿不到收益；且帖子含合约地址和推广措辞，有账号受限风险（`square-post` skill 里记的 `30008`/`2000001`/`2000002`）。
妖币雷达的广场帖不受影响（筛的是币安已上币标的，cashtag 有效）。相关：[[oracle-radar-server]]

**2026-10-16 实测确认了 cashtag 的确切机制**（用户自己在 App 里验的，帖子里挂了 `$ROSE`/`$CLAUDIA`/`$FTR` 三个 meme ticker）：
- **`$ROSE` 可点** → 但跳到的是**币安上市的 Oasis Network ROSE**，不是那个 SOL meme 币 —— **指错了**
- `$CLAUDIA` / `$FTR` **不可点**（死文字）→ 币安没有同名上市币

**即：币安广场的 cashtag 只对"币安体系里有资产页的币"生效**（上市币，或 `alpha_contracts()` 那 683 个 Alpha 合约）。
未上市的 ticker 挂 `$` **要么点不动、要么落到同名上市币上** —— 后者更糟，会把读者引到别的币。
**所以 `safe_cashtag`（只在合约对得上 Alpha 名录时才加 `$`）的"宁缺毋滥"是对的，不要为了赚内容挖矿而放宽它。**
**推论**：广场的内容挖矿收益只能来自**币安已上市 / Alpha 标的**的内容；meme/DEX 小币的帖子在广场是零收益。
因此 `square-smartlive.timer`（聪明钱扫货 → 广场）已于 10-16 停用（`disable --now`，服务文件保留可随时恢复）。

**4. 币安 API 凭据（两把，用途不同）**
- `/etc/binance-trade.env`（600, root）—— **合约交易 API Key**，`ipRestrict=True`（只允许服务器出口 IP）。**现货提现已关**、现货只读；**合约可交易**（`canTrade=True`）。用于 `/opt/radar-maintenance/binance_account.py` 只读快照，`position_track.py --run` 会自动读真实持仓。
- `/etc/binance-web3.env`（600, root）—— Binance **Web3 API** Key + Secret（`BX-` 前缀）。签名那套 `/build/api/v1/*` 从服务器直连返回 `40304`（合规限制）。
  **Why**：服务器在 **Oracle ap-tokyo-1（东京）**，而文档把**日本列为有条件限制地区**（需 ①币安账号登录认证 ②KYC 国家不在禁止名单 ③服务器 IP 在日本 才解禁；API key 不满足①）。7 个业务域端点全试过，全是同一个 40304（不是"没勾业务域"那种 40104）。AMD 那台也是 ap-tokyo-1，同样受限。
  **How to apply**：**已用 Clash 代理绕过**（2026-10-15）。服务器装了 **mihomo**（`/etc/systemd/system/mihomo.service`，enabled），只把 `binance.com` 走代理，节点池用 filter **只保留香港/新加坡/台湾**（美/加/英/荷在币安禁止名单、日本就是原 40304 的原因）。订阅链接在 `/etc/clash.env`（600），配置生成+测试脚本 `/opt/mihomo/setup.py`（`--stop` 一键拆除）。实测直连 40304 → 走代理 `code:0 success`，连测 3 次通过。**要从脚本里调 Web3 API，就走 `http://127.0.0.1:7890` 这个代理。** 注：用户给了两条订阅，sub1 域名解析失败（只有 sub2 可用，39 节点），mihomo 每小时会为 sub1 记一条无害的 error。
