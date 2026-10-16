---
name: oracle-radar-server
description: 用户的 Oracle 云服务器上跑着四套监控雷达 + 跨机器 ai-context 中继站，以及 SSH / Tailscale 连接方式
metadata:
  node_type: memory
  type: project
  originSessionId: 1eaaa207-154d-499d-837b-685c9803c7f8
  modified: 2026-10-16T17:17:13.141Z
---

用户在 Oracle Cloud ARM 服务器上运行一套自建监控雷达系统，用户用中文交流。

**服务器**：`140.238.38.82`（主机名 `instance-20260607-1415`，ap-tokyo-1，**时区 UTC**，日志日期比北京早一天）。
登录：`ssh -i /home/bixiaowuhome/桌面/ssh-key-2026-06-07arm.key ubuntu@140.238.38.82`（ubuntu 用户 passwordless sudo）。
私钥与项目文件在本机 `/home/bixiaowuhome/桌面/`。

**四套雷达**（服务名是遗留的，真正干活的是另一批脚本，别搞混）：

| 系统 | 遗留服务名 | 实际脚本 |
|---|---|---|
| 妖币/币安Alpha | `alpha-radar.service`（已 disabled） | `/opt/alpha-radar/demon_scan.py` `demon_track.py` `demon_review.py` |
| meme | `meme-radar.service`（长驻采集，只采集不推送） | `/opt/meme-radar/meme_scan.py` `meme_track.py` `meme_publish.py` |
| 股票 | `stock-radar.service`（长驻采集） | `/opt/stock-radar/stock_scan.py` `stock_track.py` `stock_ta.py` |
| X 监控 | `x-monitor.service` | `/home/ubuntu/x-monitor-validation-20260926/x-monitor-no-api/x_monitor.py`（Playwright） |

推送统一走 scan/track 系列脚本的定时器，**不是**长驻服务。

**跨机器协作**：用户有**两台电脑**，各自开着 Claude Code 会话，都 SSH 到这台服务器。跨会话 `SendMessage` 只支持**同机**，所以跨机器靠服务器上的共享文件中继站 `/home/ubuntu/ai-context/`（追加式 journal + STATE.md + handoff.md，脚本 `bin/ctx`）。**开工前先读 STATE.md 和 handoff.md，做完事用 `bin/ctx "..."` 记日志**（两台机器 SSH 进来时 `hostname` 都是服务器名，需用 `AI_CONTEXT_WHO` 覆盖）。相关：[[radar-ops-constraints]]

**Tailscale 组网**（2026-10-15 本机装好并登录，账号 `bixiaowu@`）：本机与云服务器已在同一 tailnet，可走内网 IP 直连、不必用公网 IP。

| 节点 | tailnet IP | 备注 |
|---|---|---|
| `bixiaowuhome-computer` | 100.106.137.105 | 用户本机（Ubuntu 22.04，snap 版 tailscale，`sudo tailscale up` 登录） |
| `instance-20260607-1415` | 100.77.74.3 | = 上面那台 `140.238.38.82`（tokyo），实测可打通 IPv6 直连 |
| `instance-20260531-2335` | 100.121.14.104 | **另一台 Oracle 实例**（AMD/x86_64，Ubuntu 24.04）。公网 `158.101.142.237`，`ssh -i /home/bixiaowuhome/桌面/ssh-key-2026-05-31amd.key ubuntu@158.101.142.237`。**已确认它的角色=备份机**（收 ARM 的雷达备份，再转 Cloudflare R2，见 [[radar-backup-chain]]）+ 翻墙出口（见 [[cf-tunnel-vpn]]）|
| `muse` | 100.90.45.57 | 常显示 idle，`tailscale ping` 不通 |
| `bixiaowu` | 100.96.158.65 | 用户另一台设备，常离线 |

注意：tailnet 只解决**网络可达**，不改变 `SendMessage` 仅限同机的限制，`ai-context` 中继站仍然是跨机协作的正路。
