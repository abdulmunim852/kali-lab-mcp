---
name: kali-windows-ssh-setup
description: Configure or repair Windows OpenSSH access to the user's Kali VM for the Kali Lab plugin.
---

# Windows SSH setup for Kali Lab

Use this skill when the user is setting up Kali Lab for the first time or repairing Windows SSH access.

## Workflow

1. At the start of a Kali work session, first use Kali MCP `execute_command` with `whoami && hostname -I`. Show the command and complete output. Read the configured `HostName` and `User` from `%USERPROFILE%\.ssh\config` with PowerShell and compare the username and configured IP against the returned username and IPv4 list (accept a match to any returned IPv4). Ask the user for the current IP only if SSH/MCP fails, the command output has no usable IPv4, or the configured address is absent from the returned list. If it differs, ask for the current address, then update only the `kali-lab` entry and preserve other entries. On a first-time setup with no working SSH alias, ask for the IP needed to create it.
2. Use PowerShell yourself when available. Inspect `%USERPROFILE%\.ssh\kali_mcp` before creating a key. Never overwrite it. If absent, create an Ed25519 key. Never display, copy, or commit the private key.
3. Read the public key from `%USERPROFILE%\.ssh\kali_mcp.pub` using PowerShell. Validate that it is one line beginning with `ssh-ed25519`. Do not retype, infer, or substitute the key contents.
4. Create or update `Host kali-lab` with the confirmed username, current IP, key path, `IdentitiesOnly yes`, and normal host-key checking. Preserve unrelated SSH config entries. Set Windows ACLs for the private key and SSH config so OpenSSH accepts them; do not apply permissive ACLs.
5. Give the actual public key to the user by embedding it in the ready-to-run Kali Bash script described in `kali-vm-setup`. Do not merely tell them to paste a key line or describe steps in prose.
6. After the user runs the script, verify from PowerShell with `ssh -o BatchMode=yes kali-lab "whoami"`. Show the command and complete result. If it fails, diagnose the specific failure before proceeding.
7. For a changed or new host key, compare the Windows fingerprint with the fingerprint displayed by `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` on Kali. Never disable host-key verification.

The user should run `kali-server-mcp` as the normal Kali account, not root. Refer to `kali-vm-setup` for the exact Kali-side script and API startup steps.
