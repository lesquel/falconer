# Falconer — Install

> Everything runs on **your** computer (laptop, workstation or VPS) and your phone. There is no Falconer server.
> The phone and the computer reach each other over your Tailscale network.

## In 4 steps

1. **Host:** install Tailscale and log in (`curl -fsSL https://tailscale.com/install.sh | sh`,
   then `sudo tailscale up`). Do the same on the phone with the same Tailscale account.
2. **Host:** install the hook and run the guided setup:
   ```sh
   curl -fsSL https://raw.githubusercontent.com/lesquel/falconer-hook/main/install.sh | sh
   falconer-hook setup
   ```
   Or open the terminal UI and do everything from there: `falconer-hook`.

   To read the installer before running it, use the tag-pinned copy:
   ```sh
   curl -fsSLO https://raw.githubusercontent.com/lesquel/falconer-hook/v0.2.0/install.sh
   less install.sh
   sh install.sh
   ```
3. **Phone:** install **ntfy** (F-Droid or Play Store) and open it once, then install the Falconer
   APK from the [latest `falconer-app` release](https://github.com/lesquel/falconer-app/releases/latest)
   (most phones: `openmosh-v<version>-arm64-v8a.apk`, the asset keeps its pre-rename name, about 15.5 MB; Android 9+ with a screen lock).
4. **Pair:** `falconer-hook pair --alias my-host` on the host, then **Scan pairing code** in the
   app. The host asks `Pair "<name>" from <node> (<ip>)? [y/N]`; answer `y` within 60 s. The code
   works once and expires in 5 minutes. Without ntfy the phone still pairs, with notifications
   off; add them later with **Enable notifications** on the host screen.

## What you end up with

| Piece | Role | Required |
|---|---|---|
| `sshd` | The app connects here | Yes |
| Tailscale | Private network between phone and host, no public ports. Pairing refuses to run without it | Yes |
| Herdr (or tmux) | Keeps agents alive while the phone is away; Herdr also lets the computer follow the phone's tab | Yes (one of them) |
| `falconer-hook` | Watches agents, sends encrypted pushes, executes approvals | Yes |
| ntfy app on the phone | UnifiedPush distributor on Android (`ntfy.sh` sees only ciphertext, or self-host) | For notifications |

## The installer

`install.sh` detects the OS and architecture, downloads `falconer-hook-<target>.tar.gz` from the
latest GitHub release, verifies `SHA256SUMS` and its **minisign signature** (pinned key
`RWTiZ1vxHn/jWfqAEjobeqloI9cL9pOA0/in+ayyCgPqT0gYDRKR1Ox5`, trusted comment
`falconer-hook <tag>`), puts the binary in `~/.local/bin`, writes the config, installs the user service and runs
`doctor`. It prints every command and never calls `sudo`. The signature is required: install
`minisign` (`apt`/`dnf`/`pacman`/`brew install minisign`) or `rsign` (`cargo install rsign2
--locked`) first, or the installer stops before downloading.

| Flag | Effect |
|---|---|
| `--dry-run` | Print every step, change nothing |
| `--version <tag>` | Install `vX.Y.Z` or a pre-release `vX.Y.Z-rc.N` instead of the latest stable release |
| `--insecure-no-signature` | Check the checksums only, without the signature. Not recommended |
| `--from-source` | Build with `cargo install --git … --locked` |
| `--binary <path>` | Install a binary you already built |
| `--yes`, `--no-service` | Accept the Herdr/Tailscale offers; skip the service |

Platforms: Linux x86_64 and aarch64 (static musl), macOS x86_64 and Apple Silicon.

### Release channels

| Channel | Tags | How to get it |
|---|---|---|
| Stable | `vX.Y.Z` | Default (`/releases/latest`) |
| Pre-release | `vX.Y.Z-rc.N` | `install.sh --version vX.Y.Z-rc.N`; never marked "latest" |

## Setup, agents and the TUI

- `falconer-hook setup` writes the config, installs the service, offers an integration for every
  agent it detects and prints the next step.
- Supported agents: Claude Code, Codex CLI, Gemini CLI, Cursor, GitHub Copilot CLI, Kiro CLI, OpenCode
  and Aider through their own hooks; Amp and Pi through Herdr. `falconer-hook integration list` shows what
  is detected and installed. Herdr detects agents on its own as well.
- `falconer-hook` (no arguments) opens the TUI: status (`doctor`), agents and inbox, devices,
  pairing, integrations, setup and uninstall.
- **Open Herdr** in the app runs `falconer-hook attach herdr`, which mirrors focus: the computer
  follows the tab and workspace picked on the phone, and the other way round.
- `falconer-hook usage` reports token usage per agent, project and model (Codex only: Claude Code
  keeps no local usage summary, so it is not read).

## Lost phone

`falconer-hook devices revoke <id>` (or `x` on the TUI Devices screen, or *Revoke* in the app)
removes the phone's key and push key, ends its live SSH sessions and prints a checklist of what
else to do.

## Uninstall

```sh
falconer-hook uninstall --dry-run   # see exactly what would be removed
falconer-hook uninstall             # service, integrations, paired SSH keys, devices, state, config
falconer-hook uninstall --all       # also deletes the binary
```

It only removes what Falconer created: `authorized_keys` lines tagged `openmosh:` (the tag keeps its
pre-rename name), its own integration entries, and the `falconer` config and state directories. The sshd
drop-in from `harden-ssh` is kept and listed with the command to remove it, because removing it turns password
logins back on.

## Self-hosting ntfy (optional)

If you don't want even ciphertext on `ntfy.sh`, run your own ntfy server inside your tailnet and
select it as the default server **in the ntfy app on the phone**. The phone hands its push endpoint
to the host during pairing, so nothing changes on the host.

## Recommended host hardening

`falconer-hook doctor` warns when `sshd` accepts passwords. Key-only SSH is recommended:

```sh
falconer-hook harden-ssh --dry-run   # see the file it writes
falconer-hook harden-ssh             # asks, then uses sudo; also `s` on the TUI Status screen
```

It writes `/etc/ssh/sshd_config.d/10-falconer.conf`, checks it with `sshd -t` and reloads sshd.
Logins with a password over SSH stop working; key logins and the local console do not change.
