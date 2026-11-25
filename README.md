# Kali Lab MCP plugin for Codex

A reusable Codex plugin that connects to the **official Kali `mcp-kali-server` package** on an existing Kali VM. Codex starts Kali's `mcp-server` bridge over SSH, so the Kali API can stay on `127.0.0.1:5000`. Windows needs OpenSSH, which is built into current Windows versions. This setup does not need WSL, Python, Docker, a public listener, Pinggy, or another internet tunnel.

The plugin is in [`plugins/kali-lab`](plugins/kali-lab). The repository marketplace is in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json), a repo marketplace format supported by Codex.

## One-time Kali VM setup

Run on Kali:

```bash
sudo apt update
sudo apt install -y mcp-kali-server openssh-server
sudo systemctl enable --now ssh
whoami
hostname -I
```

Use the `kali` username and the current reachable IPv4 address. Start the API as the normal `kali` user:

```bash
kali-server-mcp
```

Leave it running while Codex uses the bridge. The API defaults to `127.0.0.1:5000`; keep it bound to localhost. The official package exposes powerful tools, including arbitrary command execution. Only connect it to Codex if you understand and accept that access.

## One-time Windows SSH setup

For the first connection, ask Codex to perform the Windows-side setup using PowerShell. The plugin's onboarding skill tells Codex to:

1. Ask you for Kali's current IPv4 address (from `hostname -I`) and username.
2. Generate an SSH key only if one does not already exist.
3. Show you the **public key line** in chat so you can paste it into Kali's `/home/kali/.ssh/authorized_keys`.
4. Create or update the Windows `kali-lab` SSH host entry, preserving other SSH config entries.
5. Verify key-based SSH and repair Windows OpenSSH Agent/ACL settings if needed.

Do not share or commit the private key. The VM IP and SSH key are user-specific settings and are not stored in this repository.

At the start of every new Kali work session, the plugin asks for Kali's current IPv4 and checks it against the Windows SSH config. If the VM received a new bridged DHCP address, provide the new IP and Codex updates the SSH entry.

## Install from this GitHub repository

After pushing this repository to GitHub, add its marketplace once in PowerShell (replace the placeholders):

```powershell
codex plugin marketplace add OWNER/REPOSITORY
```

Restart Codex. Open the **Plugins Directory**, select the **Kali MCP** marketplace, and install **Kali Lab**. Trust this repository and start a new chat. The project config enables the plugin server with approval prompts for tool calls.

For the current local checkout, restart Codex and install **Kali Lab** from the **Kali MCP** repo marketplace in the Plugins Directory.

## Usage

Keep `kali-server-mcp` running on Kali, then ask Codex to call `server_health`. For a scan, identify the authorized lab target and request the specific tool, for example:

> Use Kali Lab's `nmap_scan` to run a basic scan against my lab VM at 192.168.56.101.

The plugin instructions require Codex to show each Kali tool call, its arguments, and its full result in chat. Only scan systems you own or have explicit authorization to test.

## Files

- `plugins/kali-lab/plugin.json` — plugin identity and Codex display metadata.
- `plugins/kali-lab/mcp.json` — local stdio bridge launched over SSH.
- `plugins/kali-lab/skills/kali-setup/SKILL.md` — setup, transparency, use, and troubleshooting workflow.
- `.claude-plugin/marketplace.json` — repository plugin catalog.
- `.codex/config.toml` — project-scoped plugin enablement and approval policy.

## References

- [Kali's official package page](https://www.kali.org/tools/mcp-kali-server/)
- [Kali package README](https://gitlab.com/kalilinux/packages/mcp-kali-server/-/raw/kali/master/README.md)
- [Codex plugin packaging](https://developers.openai.com/plugins/build/plugins)
