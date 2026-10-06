---
name: x-account-monitoring
description: Build or review an X account post monitor that deduplicates posts and forwards them to WeCom and DingTalk within a five-minute SLA.
argument-hint: "[project-path]"
user-invocable: false
allowed-tools: [Read, Grep, Glob, Bash]
---

## When to use

Use for requests like “监控的X账户发帖信息，5分钟内自动推送到相关微信，钉钉群”. This covers official X API polling, persistent post-id deduplication, and WeCom/DingTalk webhook delivery. It does not make personal-WeChat automation safe or supported.

## Inputs / context to gather

1. Inspect the target checkout and identify whether an existing `x_monitor/` package, tests, `.env` example, or deployment files already exist.
2. Confirm monitored account count, content filters (original/repost/reply and keywords), image forwarding, deployment environment, and failure-alert expectations.
3. Confirm that “微信” means an enterprise WeCom group webhook. If it means personal WeChat, explain the lack of a stable official group-bot interface before implementing.
4. Keep Bearer Tokens and webhook URLs in local `.env` or secret storage; never request them in chat.

## Procedure

1. Use X API v2 app-only Bearer Token authentication and poll on a configurable interval. A 240-second default meets the five-minute requirement; one-minute polling targets usual 1–2 minute delivery.
2. Persist `post_id` state in SQLite. Suppress historical posts on first run unless `--notify-existing` is explicitly requested.
3. Format each notification with author, body, timestamp, original link, and optional media links.
4. Send to both WeCom markdown and DingTalk markdown webhooks; support DingTalk signing when configured.
5. Mark a post delivered only after all configured notifiers succeed. Leave failed deliveries eligible for retry on the next poll.
6. Expose operational controls equivalent to `--once`, `--notify-existing`, configurable account/token/interval/state, webhook settings, and signing settings.

## Efficiency plan

- Start with local tests and CLI help before attempting live API calls or webhook delivery.
- Use a fixture or fake X client for deterministic first-run suppression and later-post delivery tests.
- Do not spend time debugging live network behavior until endpoint/auth configuration is confirmed; isolate API, store, monitor, and notifier failures.

## Pitfalls and fixes

- Personal WeChat requested -> pivot to WeCom or explicitly document the unsupported/risky path.
- Duplicate notifications -> check SQLite persistence and deduplicate by immutable `post_id`.
- Historical flood on startup -> enable first-run suppression; require explicit `--notify-existing` to replay.
- One notifier fails -> do not mark delivered; retry on the next poll.
- Secrets appear in logs/chat -> move them to `.env`/secret storage and redact output.

## Verification checklist

- `python3 -m unittest discover -s tests -v` passes.
- `python3 -m compileall -q scanner x_monitor` passes when both packages exist.
- `python3 -m x_monitor.cli --help` exposes account, token, interval, state, webhook, signing, `--notify-existing`, and `--once` controls.
- A fake-client test proves first-run suppression and delivery of a later post.
- Treat real webhook delivery and live X polling as separately verified only when actually exercised.
