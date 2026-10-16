---
name: cf-tunnel-vpn
description: 自建翻墙工具方案（Cloudflare Tunnel + Xray）+ 关键实测：Oracle 东京 IP 被 GFW 限速，必须走 CF 中转并用优选 IP
metadata:
  node_type: memory
  type: project
  modified: 2026-10-16T17:17:00.422Z
  originSessionId: f3154622-2e70-4b0c-a4ec-f7ee593ffca0
---

用户的自建翻墙工具，2026-10-16 立项。定位是**翻墙工具，不是匿名工具**（用户已明确确认，不要求对抗定向调查）。

**核心实测结论（2026-10-16 晚高峰，北京）——这是整个方案的立论基础：**

| 路径 | 实测 |
|---|---|
| 本机 → 国内阿里云 CDN | 6.2 MB/s（宽带本身正常） |
| 本机 → Oracle 东京 **直连 IPv4** | **6.5 KB/s** |
| 本机 → Oracle 东京 **直连 IPv6** | **2.1 KB/s** |
| 本机 → Cloudflare 直连 | 130–442 KB/s |
| Oracle → Cloudflare（服务器出站） | 12–42 MB/s |
| **经 CF 隧道 + 优选 IP `162.159.0.1`** | **4.5–5.1 MB/s（36–40 Mbps）** |

**Why 重要**：用户上次失败（`桌面/oraclevpn/oracle-clash-verge-computer-zhuji.yaml`，WireGuard 直连 ARM 的 UDP 51820）**不是协议问题**——换 Reality/Trojan/WireGuard 都一样，因为 **Oracle 东京 IP 段被 GFW 针对性限速（v4/v6 都是死路）**，瓶颈在 IP 层不在协议层。任何直连 Oracle 的方案都不可能稳。

**How to apply**：
- **绝不直连 Oracle**，必须经 Cloudflare 中转
- **优选 IP 是决定性的**：同一时刻 CF 不同边缘 IP 从中国访问速度差 14 倍以上（扫了 18 个 IP，最好的 `162.159.0.1`、`108.162.192.1` 达 1.5–1.9 MB/s，有 9 个完全连不上）。客户端必须把 server 设为优选 IP，SNI/Host 仍填真实域名。默认解析到的 IP 常常是烂的。
- 架构：`Hiddify(手机)/Clash Verge(电脑) → CF Edge +优选IP → Cloudflare Tunnel → Oracle Xray(VLESS-WS)`。用 **Tunnel 而不是开端口**，因为 ARM 的 iptables INPUT 末尾是 `REJECT` 一切，开端口要同时改 iptables + VCN 安全列表；Tunnel 是服务器主动出站，零入站端口。
- **不支持 UDP**（Tunnel 是 HTTP 层只转发 TCP）。YouTube/Google 会自动回落 TCP 无感；在线游戏不行。
- 主备：**主力放闲置的 AMD**（`158.101.142.237`，隔离风险），**备用放 ARM**（`140.238.38.82`）。理由：ARM 上跑着币安雷达 + mihomo（存着合约交易 API Key 和订阅凭据），不能被 VPN 流量牵连。

**服务器上的落地物（AMD）**：`/usr/local/bin/xray`（v26.3.27，systemd `xray.service`）、`/usr/local/etc/xray/config.json`（VLESS-WS，只监听 `127.0.0.1:8080`）、`/usr/local/bin/cloudflared`、`cf-quick.service`（Phase 0 临时隧道，日志 `/var/log/cf-quick.log`）。Xray 内存占用仅 11 MB。

**2026-10-16 进展：域名已买（`bitbit.casa`），已从临时隧道转到正式命名隧道**。落地物：`cf-tunnel.service`（`/etc/cloudflared/config.yml`，tunnel `vpn-amd` / id `a257f93a-dbb6-4f37-83d4-5ced75303a7c`，ingress 把 `vpn.bitbit.casa` 转到 `http://127.0.0.1:8080`）。2026-10-17 实测两个服务都在跑、且 Xray 日志里有国内家宽 IP 在访问 google/youtube——链路是通的。

**待办**：原 Phase 0 的 `cf-quick.service` 已可下线（确认客户端不再依赖后 disable）。

相关：[[oracle-radar-server]]、[[radar-ops-constraints]]、[[radar-backup-chain]]

**注意**：用户曾问能否用这个代理实现 X 自动发帖——已说明不成立：雷达跑在 Oracle 服务器上、本来就能直连 X，VPN 解决的是"家里设备访问外网"，两者无关；X 自动发帖的瓶颈是 API 额度（免费额度已用完），且用浏览器自动化发帖有封号风险。
