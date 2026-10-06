---
name: ubuntu-upgrade-recovery-triage
description: Triage a failed Ubuntu release upgrade or no-boot report from recovery root without destructive repair guesses.
argument-hint: "[host-or-working-directory]"
user-invocable: false
allowed-tools: [Read, Grep, Glob, Bash]
---

## When to use

Use when the user reports a failed Ubuntu upgrade, boot failure, `degraded` state, or access only to root/recovery mode. Do not use it to immediately run repairs, retry `do-release-upgrade`, remove kernels, or modify APT sources without diagnostics.

## Inputs / context to gather

1. Ask for the exact prompt: recovery root shell such as `root@host:~#`, or `(initramfs)`.
2. Record the requested source and target release, whether data must be preserved, and whether a recent backup exists.
3. In a recovery root shell, collect these read-only outputs before selecting a repair:
   ```bash
   cat /etc/os-release
   df -h / /boot /boot/efi
   dpkg --audit
   tail -n 60 /var/log/apt/term.log
   ```
4. In a running but unhealthy system, use privileged checks: `sudo dpkg --audit`, `sudo apt-get check`, `systemctl --failed`, and `/var/log/dist-upgrade/` or `/var/log/apt/`.

## Procedure

1. Classify the prompt. Stop normal APT instructions for `(initramfs)`; investigate filesystem/root-device recovery instead. Continue this procedure only for a usable recovery root shell or booted system.
2. Establish the actual release from `/etc/os-release`, not from the kernel version. Check free capacity on `/`, `/boot`, and `/boot/efi`.
3. Inspect package and service evidence under sufficient privilege. A non-root `dpkg --audit` result is inconclusive.
4. Read the latest APT/dist-upgrade errors and identify the specific branch: insufficient space, interrupted package configuration, failed package dependency, or boot/kernel issue.
5. Propose the smallest repair only after that evidence is available. Keep source changes and release retries until after package state is understood.
6. After any authorized repair, rerun package audit/check and failed-unit inspection, then verify normal boot before calling the release recovery complete.

## Efficiency plan

- Get the prompt type and four diagnostic outputs in one user turn; they determine the repair branch and prevent generic command loops.
- Use terminal diagnostics when browser/computer-use tools are unavailable; `unsupported Codex auth method: apikey` was non-blocking for shell investigation.
- Stop at diagnosis when outputs are absent. Preserve evidence and request it rather than speculating.

## Pitfalls and fixes

- `dpkg: error: unable to check lock file ... Permission denied` -> rerun with root privileges; do not call package state healthy or broken from that result.
- `systemctl is-system-running` returns `degraded` -> inspect `systemctl --failed` before retrying a release upgrade.
- Mixed package versions after an interrupted upgrade -> do not blindly alter sources or rerun the upgrade.
- Unknown recovery prompt -> distinguish `root@host:~#` from `(initramfs)` before giving package commands.
- User asks for quick repair but provides no output -> request prompt and diagnostics; do not run `autoremove`, delete kernels, format partitions, or apply generic boot fixes.

## Verification checklist

- Exact prompt type is known.
- Release, disk space, package audit/check, failed services, and relevant upgrade logs were captured under adequate privilege.
- Any repair is tied to a specific observed failure, not a generic upgrade recipe.
- After repair, package check/audit and service state are rechecked; normal boot is confirmed separately.
