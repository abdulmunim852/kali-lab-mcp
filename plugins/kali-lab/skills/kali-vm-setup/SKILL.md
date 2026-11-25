---
name: kali-vm-setup
description: Prepare the Kali VM for Kali Lab, generate a copyable setup script containing the user's actual SSH public key, and start Kali's MCP API.
---

# Kali VM setup

Use this skill for first-time Kali package, SSH key authorization, and API setup.

## Produce the Kali Bash script

After Windows setup has read and validated the actual public key from `%USERPROFILE%\.ssh\kali_mcp.pub`, return a complete Bash script directly in chat for the user to paste into a Kali terminal. Do not create a script file. Do not return generic directions in place of the script, and never include the private key.

Embed the exact public key line in a `PUBKEY='...'` assignment. OpenSSH public key characters do not contain single quotes; if an unexpected quote appears, stop and do not produce an unsafe shell snippet. The script should:

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ "$(id -u)" -eq 0 ]; then
  echo "Run this as your normal Kali user, not root." >&2
  exit 1
fi

PUBKEY='ssh-ed25519 REPLACE_WITH_THE_EXACT_LINE_READ_FROM_WINDOWS'
case "$PUBKEY" in
  ssh-ed25519\ *) ;;
  *) echo "Invalid Ed25519 public key." >&2; exit 1 ;;
esac

sudo apt update
sudo apt install -y mcp-kali-server openssh-server
sudo systemctl enable --now ssh

install -d -m 700 "$HOME/.ssh"
chmod 700 "$HOME/.ssh"
touch "$HOME/.ssh/authorized_keys"
chmod 600 "$HOME/.ssh/authorized_keys"
if ! grep -Fqx -- "$PUBKEY" "$HOME/.ssh/authorized_keys"; then
  printf '%s\n' "$PUBKEY" >> "$HOME/.ssh/authorized_keys"
fi
chmod 600 "$HOME/.ssh/authorized_keys"

echo "Kali SSH and MCP packages are ready for user: $(id -un)"
```

Replace the placeholder with the actual public key read from Windows before showing the script. Never leave the placeholder in the script returned to the user. Keep existing `authorized_keys` entries; only append the key when that exact line is absent. The script uses `sudo` for packages and service setup, while creating the authorized key under the logged-in normal user's home.

Tell the user to paste the whole script into a Kali terminal and run it. Then ask them to report its output or continue when it completes. Show and explain any result they provide.

## Start the API

After setup, have the user run this in a separate Kali terminal as the same normal user:

```bash
kali-server-mcp
```

The API should remain running at `127.0.0.1:5000` while Codex uses the SSH bridge. Do not run it as root or bind it to `0.0.0.0`. Keep the terminal open. Verify connectivity from Codex with `server_health` after confirming the current Kali IPv4 address according to `kali-connection-troubleshooting`.

## Reporting

Show the exact Kali script before the user runs it. After they run it, report the complete output they provide. Do not claim setup succeeded until output or a subsequent connection check confirms it.
