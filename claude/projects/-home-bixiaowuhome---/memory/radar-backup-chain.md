---
name: radar-backup-chain
description: 雷达数据的异地备份链路（ARM→AMD→Cloudflare R2）+ 各定时器时间/保留策略；实测确认没有 GitHub 备份那条腿
metadata:
  node_type: memory
  type: project
  originSessionId: fa71fad5-e5c6-44fd-b346-fb93b0f7c7b7
  modified: 2026-10-16T17:14:11.569Z
---

雷达数据的备份分三层，**没有 GitHub 这一层**（2026-10-17 实测：两台服务器都没装 git/gh、没有 GitHub 凭据、没有任何指向 GitHub 的 push；用户记忆中的 "GitHub 备份" 待澄清，本机唯一的 GitHub 同步是 `bixiaowu2/aisync-config`，那是本机配置不是雷达数据）。

**链路**：ARM 东京(雷达主机) → AMD 东京(备份主机) → Cloudflare R2(异地第三层)

| 层 | 定时器 | 频率 | 脚本 | 落地 |
|---|---|---|---|---|
| ARM→AMD 状态层 | `radar-backup-state.timer` (ARM) | 每 15 分钟 (`*:03,18,33,48`) | `/usr/local/bin/radar-backup state` | AMD `/srv/backup/arm/state/` + `state-daily/<日期>/`，保留 14 天 |
| ARM→AMD 数据库层 | `radar-backup-db.timer` (ARM) | 每天 02:23 UTC | `radar-backup db` | AMD `/srv/backup/arm/db/<日期>/`，保留 7 份 |
| AMD→R2 状态层 | `r2-sync-state.timer` (AMD) | 每天 03:41 UTC | `/usr/local/bin/r2-sync state` | R2 `oracleradar-backup/state` |
| AMD→R2 数据库层 | `r2-sync-db.timer` (AMD) | **每周日** 04:23 UTC | `r2-sync db` | R2 `oracleradar-backup/db/<日期>`，保留 4 份 |

**Why 这样设计**：
- ARM→AMD 走 **Tailscale 内网**（`ubuntu@100.121.14.104`），rsync 必须以 `sudo -u ubuntu` 发出——Tailscale SSH 只放行 ubuntu 用户，root 会被拒。
- **活的 SQLite 绝不能 rsync**（会拿到撕裂副本），脚本用 sqlite 在线备份 API 做一致性快照再传。
- R2 免费额度 10 GB：db 单份 1.3 G，所以 db 是**周级**不是日级（脚本里 `SIZE_LIMIT_MB=9500` 留 500M 余量，`DB_KEEP=4`）。

**How to apply**：
- rclone remote 名 `oracleradar:`，桶 `oracleradar-backup`，凭据在 AMD `/home/ubuntu/.config/rclone/rclone.conf`。
- 查状态：AMD 上 `rclone size oracleradar:oracleradar-backup`、`rclone lsf --dirs-only oracleradar:oracleradar-backup/db/`；或 `r2-sync status`。
- 查运行历史：`journalctl -u r2-sync-state -u r2-sync-db`（AMD）、`journalctl -u radar-backup-state -u radar-backup-db`（ARM）。**别只看 timer 的 LAST 字段就下结论**。
- 用户说"备份正常吗"时，注意 **R2 的 state 层是每日一次**，所以 R2 上的状态最多滞后 24h 属正常，不是故障。

相关：[[oracle-radar-server]]、[[radar-ops-constraints]]
