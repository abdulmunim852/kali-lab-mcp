# Kali Lab MCP plugin for Codex

A reusable Codex plugin that connects to the **official Kali `mcp-kali-server` package** on an existing Kali VM. Codex starts Kali's `mcp-server` bridge over SSH, so the Kali API can stay on `127.0.0.1:5000`. Windows needs OpenSSH, which is built into current Windows versions. This setup does not need WSL, Python, Docker, a public listener, Pinggy, or another internet tunnel.

The plugin is in [`plugins/kali-lab`](plugins/kali-lab). The repository marketplace is in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json), a repo marketplace format supported by Codex.

## First-time setup

Ask Codex to set up Kali Lab. The Windows SSH setup skill handles Windows-side key/configuration steps using PowerShell. It reads the public key from the Windows `.pub` file and returns a complete Kali Bash script directly in chat. The script installs `mcp-kali-server` and `openssh-server`, enables SSH, creates or repairs `~/.ssh`, appends the exact public key to `authorized_keys` only if missing, and applies `700`/`600` permissions. Paste that whole script into a Kali terminal as the normal Kali user.

Then start the API in another Kali terminal as the normal user:

```bash
kali-server-mcp
```

Leave it running while Codex uses the bridge. The API defaults to `127.0.0.1:5000`; keep it bound to localhost. Do not run it as root. The official package exposes powerful tools, including arbitrary command execution. Only connect it to Codex if you understand and accept that access.

At the start of every new Kali work session, Codex first runs `whoami && hostname -I` through Kali MCP and compares the returned username and IP list with the Windows SSH config. It asks you for the IP only if the check fails, returns no usable address, or the configured address is missing from Kali's IP list. If a bridged DHCP address changed, provide the current IP and Codex updates the SSH entry.

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

## Skills

- `kali-windows-ssh-setup` — configure Windows OpenSSH and read the actual public key safely.
- `kali-vm-setup` — return the complete Kali Bash setup script and start the API as the normal user.
- `kali-authorized-scan` — run and report authorized scans and tool calls.
- `kali-connection-troubleshooting` — diagnose IP, SSH, host key, API, and plugin issues.

## Other files

- `plugins/kali-lab/plugin.json` — plugin identity and Codex display metadata.
- `plugins/kali-lab/assets/kali-lab-icon.svg` — white-background icon with an original black dragon mark.
- `plugins/kali-lab/mcp.json` — local stdio bridge launched over SSH.
- `.claude-plugin/marketplace.json` — repository plugin catalog.
- `.codex/config.toml` — project-scoped plugin enablement and approval policy.

## References

- [Kali's official package page](https://www.kali.org/tools/mcp-kali-server/)
- [Kali package README](https://gitlab.com/kalilinux/packages/mcp-kali-server/-/raw/kali/master/README.md)
- [Codex plugin packaging](https://developers.openai.com/plugins/build/plugins)
