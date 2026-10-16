---
name: claude-desktop-linux-setup
description: 两台机器 Claude 桌面客户端的安装方式与自动更新路径（bixiaowu 用户级+自写脚本；本机系统 deb+社区 apt 源）
metadata:
  node_type: memory
  type: project
  originSessionId: d13d465a-6d92-4e10-9d8c-e4daadf0ca1a
  modified: 2026-10-16T12:00:47.057Z
---

桌面客户端用的是**社区重打包版** `claude-desktop-unofficial`（上游 `aaddrick/claude-desktop-debian`），不是 Anthropic 官方 Linux 版。该包自带的更新器只能"检测"到新版本、无法自行安装，所以必须靠 apt 源或自写脚本。

两台机器布局不同，**脚本不能互相照搬**：

- **bixiaowu**（Tailscale `100.96.158.65`，Ubuntu 26.04）：用户级安装，无需 sudo。
  - 载荷：`~/.local/share/claude-desktop-unofficial/claude-desktop-unofficial-tmp/`
  - 手写启动器 `~/.local/bin/claude-desktop`（带 `--no-sandbox --ozone-platform=x11 --disable-gpu`）——**不要覆盖**
  - 自写更新脚本 `~/.local/bin/claude-desktop-update`（支持 `--check/--wait/--yes/--auto/--rollback`），由 systemd user timer 每天 01/07/13/19:17 调用
  - 同机还装了 `cc-switch 4.0.0`，**没装桌面客户端以外的 Claude 包**

- **bixiaowuhome-Computer**（本机，Ubuntu 22.04）：系统 deb 安装。
  - 载荷 `/usr/lib/claude-desktop-unofficial/`，启动器 `/usr/bin/claude-desktop-unofficial`
  - 2026-10-16 补加社区 apt 源 `https://pkg.claude-desktop-debian.dev`（密钥指纹 `87494CF73ACC0F23AA9557B86E29E413B912E0F1`）
  - 自动更新交给 `unattended-upgrades`：用 `Origins-Pattern "o=claude-desktop-debian"`（**不要**改 `Allowed-Origins`，那会覆盖掉系统默认的安全源）
  - 定时器 `apt-daily-upgrade.timer` 约 06:19 跑

**Why:** 该包无 `prerm`、`postinst` 也不杀进程，所以升级不会中断正在运行的 App，新版下次启动才生效。

**How to apply:** 提到"更新 Claude 桌面版"时先确认是哪台机器——本机走 `apt`，bixiaowu 走那个脚本。相关：[[oracle-radar-server]]
