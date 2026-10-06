thread_id: 01a0e007-f3de-77f0-8921-8e861edf5c73
updated_at: 2026-09-30T12:10:48+00:00
rollout_path: /home/bixiaowuhome/.codex/sessions/2026/09/27/rollout-2026-09-27T07-23-32-01a0e007-f3de-77f0-8921-8e861edf5c73.jsonl
cwd: /home/bixiaowuhome/Documents/Codex/来自-bixiaowu/2026-09-27/kef

# Oracle WireGuard deployment followed by Clash Verge regression and incomplete recovery

Rollout context: The user asked whether a VPN was suitable for their Oracle instance, authorized deployment, requested support for six devices including two computers, then asked to adapt the computer profiles for Clash Verge. The Oracle server work succeeded, but an unverified performance tuning experiment worsened the user’s experience. The cloud server was rolled back successfully. Later, local Clash Verge profile switching repeatedly failed due to service/core IPC issues; the original subscription was restored and web access was verified, but the final GUI repair was not verified.

## Task 1: WireGuard deployment and six-device provisioning

Outcome: success

Preference signals:

- The user explicitly authorized deployment with “按你的建议办” and requested six devices, two of them computers. This supports directly implementing the approved server change and creating unique per-device credentials.
- The user later clarified that already configured phones and computers should not be changed during rollback. Preserve existing client profiles unless regeneration is explicitly requested.

Key steps:

- Inspected the Oracle host over SSH and confirmed Ubuntu 24.04 ARM64, 4 CPUs, about 24 GiB RAM, public IPv4 `140.238.38.82`, and an existing active `x-monitor.service`.
- Installed WireGuard and `qrencode`; configured `wg0` on UDP 51820 with IPv4 forwarding, iptables forwarding/NAT, and systemd enablement.
- Validated a real WireGuard handshake with a temporary client, then removed temporary keys/material.
- Added five independent peers to the original client, resulting in six devices total at `10.66.66.2` through `10.66.66.7`; independent key/address matching was later verified against `wg show`.
- Produced individual `.conf` and QR artifacts plus ZIP packages in the workspace.

Reusable knowledge:

- WireGuard is a low-resource fit for this Oracle host and coexists with the Chromium/Xvfb monitor. The service itself has no significant resident userspace process; bandwidth is the primary shared constraint.
- IPv4 SSH later timed out while UDP 51820 remained reachable. IPv6 SSH using `ssh -6` to the host’s IPv6 address worked and enabled rollback.

## Task 2: Performance tuning and rollback

Outcome: partial

Key steps:

- Reviewed mature WireGuard/Mihomo references and inspected the host: Oracle NIC MTU 9000 caused `wg0` to auto-use 8920; BBR was available.
- Applied MTU 1420 to server/client configs, enabled BBR and TCP MTU probing, and rebuilt client packages.
- The user reported that performance felt worse. The agent restored the prior server baseline: automatic MTU 8920, TCP `cubic`, and `tcp_mtu_probing=0`; client artifacts were also restored without MTU overrides.
- Rollback validation showed `wg-quick@wg0=active`, enabled at boot, `x-monitor.service=active`, and all six peers retained.

Failures and how to do differently:

- The tuning was not validated from the user’s actual device/network before being presented as an improvement. Treat MTU/BBR changes as experiments, benchmark representative real paths, and preserve a rollback before changing both server and clients.

## Task 3: Clash Verge/Mihomo conversion and timeout investigation

Outcome: partial

Preference signals:

- The user said the two computers use Clash Verge and asked whether the configuration should be changed. Future agents should generate the client-native Mihomo YAML and verify the actual GUI/core rather than only the raw WireGuard files.
- The user repeatedly asked for progress and reported problems quickly, indicating a need for concise status updates and explicit verification evidence.

Key steps:

- Converted the two computer WireGuard profiles to `oracle-clash-verge-computer-1.yaml` and `oracle-clash-verge-computer-2.yaml`.
- Verified both YAML files with Mihomo v1.19.31. Isolated proxy tests eventually passed for both and returned Oracle exit IP `140.238.38.82`, though one first attempt failed due DNS/TLS timeout.
- Diagnosed intermittent failures around `remote-dns-resolve: true`; logs repeatedly contained `dns resolve failed`, TLS EOF, and timeout errors.
- The user ultimately preferred returning to the original subscription rather than continuing the Oracle YAML experiment.

Failures and how to do differently:

- Passing an isolated Mihomo test does not prove Clash Verge is using that profile. The GUI later returned to a remote subscription, so always inspect the active profile, runtime config, selected proxy group, and real browser traffic.
- Do not overwrite an active subscription/runtime configuration during experiments; use a separate local profile and preserve complete backups.

## Task 4: Restore original subscription and repair Clash Verge switching

Outcome: partial

Key steps:

- Restored the prior subscription profile from `profiles/RpvSdagCU6Zb.yaml.before-oracle-force-20260927-1918` and verified the restored profile contained 106 nodes.
- Verified web connectivity through the restored subscription: Google HTTP 204 in about 0.8 seconds and YouTube HTTP 204 in about 2.8 seconds.
- Determined that the missing detailed country groups were due to profile choice: `RpvSdagCU6Zb` had six broad groups, while `RlsfKOhEJpzZ` had 31 detailed country/service groups. The nodes were not deleted.
- Repeatedly investigated GUI errors. The service socket initially had root-only access, producing `权限不够 (os error 13)`. A systemd drop-in was added, and the service socket eventually became `root:bixiaowu` mode 660 with `/tmp/verge` mode 2770.
- After permissions were fixed, the GUI error changed to missing core socket `没有那个文件或目录 (os error 2)`. The final attempted restart of `clash-verge`/`verge-mihomo` produced no verification output, so successful profile switching was not proven.

Failures and how to do differently:

- The final state is unresolved until a fresh user-owned `verge-mihomo` is running, `/tmp/verge/verge-mihomo.sock` exists and is accessible, and the GUI successfully switches to `RlsfKOhEJpzZ` while showing its detailed groups.
- Do not claim completion from service activity or socket permissions alone; validate the actual GUI profile switch and browser connectivity.

References:

- Workspace: `/home/bixiaowuhome/Documents/Codex/来自-bixiaowu/2026-09-27/kef`
- Clash Verge data: `/home/bixiaowuhome/.local/share/io.github.clash-verge-rev.clash-verge-rev`
- Service unit: `/etc/systemd/system/clash-verge-service.service`
- Permission drop-in: `/etc/systemd/system/clash-verge-service.service.d/permissions.conf`
- Service socket: `/tmp/verge/clash-verge-service.sock`
- Core socket expected by the user process: `/tmp/verge/verge-mihomo.sock`
- Key errors: `Connection failed, I/O error: 权限不够 (os error 13)` and `Connection failed, I/O error: 没有那个文件或目录 (os error 2)`
