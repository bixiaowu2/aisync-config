---
name: github-code-backup
description: "雷达代码的 GitHub 备份仓库 bixiaowu2/codexmonitor——靠会话手动推，没有自动化；含 deploy key 位置和\"仓库目录 ↔ 服务器 /opt 路径\"的对应关系"
metadata:
  node_type: memory
  type: project
  originSessionId: fa71fad5-e5c6-44fd-b346-fb93b0f7c7b7
  modified: 2026-10-16T17:34:59.289Z
---

雷达**代码**（不是数据）备份到 GitHub 私有仓库 **`bixiaowu2/codexmonitor`**。截至 2026-10-17 是 68 次提交、164 个文件。

**仓库结构 ↔ 服务器路径**（名字不一样，别按名字找）：

| 仓库目录 | 服务器路径 |
|---|---|
| `alpha-monitor/` | `/opt/alpha-radar` |
| `meme-monitor/` | `/opt/meme-radar` |
| `stock-monitor/` | `/opt/stock-radar` |
| `x-monitor/` | x-monitor 相关（`/opt/x-engagement`、`square-post`）|
| `ops/` | `/opt/radar-maintenance` |
| 根目录若干 `*.md` | 架构/数据源/迭代计划等文档 |

**⚠️ 它不是自动的**——服务器上**没有任何 git 仓库、同步脚本或定时器**（AMD 连 `git` 都没装），全靠会话里手动打包 `/opt` 代码再推上去。所以"还在运行吗"这个问题对它不适用：**没人推它就停在原地**。

- 最后成功推送：**2026-10-13 07:32**（10-11 ~ 10-13 期间很密集，之后彻底断）
- 到 10-16，服务器上已有 **29 个代码文件**晚于那次推送被修改（`x_publisher.py`、`meme_publish.py`、`radar.py`、`square_publish.py` …）——即 GitHub 上那份已明显落后

**推送凭据（还有效，实测 `git ls-remote` 成功）**：
- 服务器 ARM：`~/.ssh/codexmonitor_ed25519`（deploy key，注释 `codexmonitor-deploy@bixiaowuhome`），`~/.ssh/config` 把 `github.com` 指到它
- 本机：`~/.ssh/codexmonitor_ed25519`（同名），SSH 上去身份显示为 `bixiaowu2/codexmonitor`
- **本机 `桌面/githubtoken` 已失效**（2026-10-17 实测 API 返回 Bad credentials，5 月的老 token）——要用 GitHub API 得换新的

**How to apply**：用户问"代码同步到 GitHub 了吗"时，别只看凭据在不在——要 `git ls-remote`/查仓库最后一次 commit 时间，跟服务器代码的 mtime 比。凭据有效 ≠ 同步在跑。

相关：[[radar-backup-chain]]、[[oracle-radar-server]]
