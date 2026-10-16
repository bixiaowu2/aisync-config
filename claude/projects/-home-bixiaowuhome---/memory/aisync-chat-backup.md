---
name: aisync-chat-backup
description: AI 客户端聊天记录备份工具 aisync（Claude 桌面版 + Codex）→ restic 走 Cloudflare R2；GUI 手动运行，没有定时任务
metadata:
  node_type: memory
  type: project
  originSessionId: fa71fad5-e5c6-44fd-b346-fb93b0f7c7b7
  modified: 2026-10-16T17:35:26.901Z
---

**aisync** = 用户自建的 AI 客户端**聊天记录**备份工具（不含服务器数据）。备份范围：Claude 桌面版 / Claude Code 的 `~/.claude/projects/**` + Codex 的 `~/.codex/sessions/**`。

- 程序：`~/下载/aisync`（108 MB 的 Linux GUI 单文件），**手动启动，没有 cron / systemd timer**
- 配置：`~/aisync/aisync.conf`
  - 数据正文 → `restic` → **Cloudflare R2**，`RESTIC_REPOSITORY="rclone:r2:aisync/aisync"`（密码在 `~/aisync/.restic-pass`）
  - 索引/清单 → **GitHub** `git@github-aisync:bixiaowu2/aisync-config.git`（clone 在 `~/aisync/repo`）
  - 所以 aisync 是"内容进 R2、catalog 进 GitHub"，跟 [[github-code-backup]] 是两回事
- 本机 rclone remote `[r2]` 与服务器上的 `oracleradar` remote **是同一个 Cloudflare R2 账号**（endpoint 都是 `a76483e257d388c4e8bf5473b074368d.r2.cloudflarestorage.com`），只是桶不同

**How to apply**：用户问"aisync 还在跑吗"——它是**手动**触发的，别拿"上次运行时间早"当故障。判断方法是看 `~/aisync/repo` 的 git log 最新一条时间和 `manifest.json` 的 `updated` 字段。

相关：[[radar-backup-chain]]
