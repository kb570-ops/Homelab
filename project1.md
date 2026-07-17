# 01 — Ubuntu Server (Parallels VM)

**Goal:** Deploy a Linux server, manage it headless over SSH,
harden access, and run a full patch cycle with verification.

## 1. Build
Installed Ubuntu 24.04 (ARM64) in a Parallels VM on an M1 Mac.
The VM gets its own IP on the Parallels network (10.211.55.x),
making it reachable like a remote machine. GUI is never used —
console access is reserved for bootstrap and recovery only.

## 2. Remote access
Installed `openssh-server` on the VM. My Mac is the admin
workstation; all operations happen over SSH. Local console =
crash cart.

## 3. Hardening
- Rotated the default `parallels` account password (`passwd`)
- Generated an ed25519 key pair on the Mac; deployed the public
  key with `ssh-copy-id` — login is now a cryptographic challenge,
  not a password
- Disabled password authentication in `/etc/ssh/sshd_config`
  (checked `/etc/ssh/sshd_config.d/` for overrides — none found)
- **Verified:** `ssh -o PubkeyAuthentication=no` is rejected with
  `Permission denied (publickey)` — key-only access confirmed
- Passphrase encrypts the private key locally (macOS keychain);
  account password retained for `sudo` privilege escalation

<img width="648" height="34" alt="image" src="https://github.com/user-attachments/assets/a16bcf2d-2565-4feb-bd69-6509d7e8d68f" />

## 4. Patching + verification
Cleared a 473-package backlog (`apt update && apt upgrade`),
rebooted to load the new kernel. Post-patch checks:
- `uptime` — reboot confirmed (up 13 min)
- `uname -r` — kernel moved 6.14.0-27 → 6.14.0-28
- `apt list --upgradable` — only phased updates remain
- `systemctl --failed` — 0 failed units, clean boot

<img width="895" height="110" alt="image" src="https://github.com/user-attachments/assets/a92e3757-8f50-452e-bb3a-33f39b0df469" />

## Lessons
- Ran `ssh-keygen` on the server instead of the Mac — read the
  prompt before typing; know which machine you're on
- Commented (`#`) config lines are ignored — the setting isn't
  active until uncommented
- Empty grep output means no matches — sometimes silence is the
  passing result
- Manual SSH verification works for one box but doesn't scale —
  motivation for the monitoring project later
