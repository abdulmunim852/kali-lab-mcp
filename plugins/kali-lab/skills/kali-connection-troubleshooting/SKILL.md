---
name: kali-connection-troubleshooting
description: Diagnose Kali Lab SSH, host key, changing IP, API, and plugin connection problems.
---

# Kali Lab connection troubleshooting

Use this skill when SSH, the official Kali API, or the MCP bridge cannot connect.

## Check the VM address and SSH

At the start of each new Kali work session, first run Kali MCP `execute_command` with `whoami && hostname -I`; show the command and complete output, then compare the returned username and IPv4 list with the `User`/`HostName` in the Windows `kali-lab` SSH entry. The configured address should match any returned IPv4. Ask the user for the IP only if the command fails, produces no usable IPv4, or the configured address is absent. If it is absent, ask for the current address before updating only `HostName` with PowerShell. Never assume a bridged DHCP address remains stable when the check cannot run.

If SSH times out, show/run from PowerShell:

```powershell
Test-NetConnection <CURRENT_KALI_IPV4> -Port 22
```

Use the result to distinguish a stale IP, unreachable VM/network, and SSH service issue. Check that Kali's SSH service is enabled and running. Do not expose SSH or the Kali API to the public internet as a troubleshooting shortcut.

## Host key failures

If host key verification fails, get Kali's fingerprint with:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Compare it with the Windows SSH fingerprint before replacing a `known_hosts` entry. Never turn off host-key checking or accept a changed key without comparison.

## SSH works, but MCP does not

- Verify SSH key login from PowerShell with `ssh -o BatchMode=yes kali-lab "whoami"`.
- Verify `kali-server-mcp` is running as the normal Kali user and listening on `127.0.0.1:5000`.
- Verify the SSH bridge invokes Kali's `mcp-server --server http://127.0.0.1:5000`.
- Use `server_health` to check the MCP connection, showing the exact tool call and complete returned result.
- Keep the API on loopback. Prefer the existing SSH bridge; do not add Pinggy, a public tunnel, or an internet-facing listener for this local setup.

## Plugin not visible

Check that the repository marketplace is trusted, Kali Lab is installed and enabled, and Codex has been restarted or a new chat opened. Explain any step the user must do in the Codex UI.
