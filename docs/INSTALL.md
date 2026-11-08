# OpenMosh — Install

> The repositories are private for now. Until they are public, release downloads need the GitHub
> CLI logged in to an account with access (`gh auth login`); the installer uses it automatically.
> Everything runs on **your** host (laptop, workstation or VPS) and your phone. There is no
> OpenMosh server.

## In 4 steps

1. **Host:** install Tailscale and log in (`curl -fsSL https://tailscale.com/install.sh | sh`,
   then `sudo tailscale up`). Do the same on the phone with the same Tailscale account.
2. **Host:** install the hook and run the guided setup:
   ```sh
   curl -fsSL https://raw.githubusercontent.com/lesquel/openmosh-hook/main/install.sh | sh
   openmosh-hook setup
   ```
   Or open the terminal UI and do everything from there: `openmosh-hook`.
3. **Phone:** install **ntfy** (F-Droid or Play Store) and open it once, then install the OpenMosh
   APK from the [latest `openmosh-app` release](https://github.com/lesquel/openmosh-app/releases/latest)
   (most phones: `openmosh-<version>-arm64-v8a.apk`, about 15.5 MB; Android 9+ with a screen lock).
4. **Pair:** `openmosh-hook pair --alias my-host` on the host, then **Scan pairing code** in the
   app. The host asks `Pair "<name>" from <node> (<ip>)? [y/N]`; answer `y` within 60 s. The code
   works once and expires in 5 minutes. Without ntfy the phone still pairs, with notifications
   off; add them later with **Enable notifications** on the host screen.

## What you end up with

| Piece | Role | Required |
|---|---|---|
| `sshd` | The app connects here | Yes |
| Tailscale | Private network between phone and host, no public ports. Pairing refuses to run without it | Yes |
| Herdr (or tmux) | Keeps agents alive while the phone is away; Herdr also lets the computer follow the phone's tab | Yes (one of them) |
| `openmosh-hook` | Watches agents, sends encrypted pushes, executes approvals | Yes |
| ntfy app on the phone | UnifiedPush distributor on Android (`ntfy.sh` sees only ciphertext, or self-host) | For notifications |

## The installer

`install.sh` detects the OS and architecture, downloads `openmosh-hook-<target>.tar.gz` from the
latest GitHub release, verifies `SHA256SUMS` and its **minisign signature** (pinned key
`RWTiZ1vxHn/jWfqAEjobeqloI9cL9pOA0/in+ayyCgPqT0gYDRKR1Ox5`, trusted comment
`openmosh-hook <tag>`), puts the binary in `~/.local/bin`, writes the config, installs the user service and runs
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

- `openmosh-hook setup` writes the config, installs the service, offers an integration for every
  agent it detects and prints the next step.
- Supported agents: Claude Code, Codex CLI, Gemini CLI, Cursor, GitHub Copilot CLI, Kiro, OpenCode
  and Aider (Amp: approvals only, no hook mechanism). `openmosh-hook integration list` shows what
  is detected and installed. Herdr detects agents on its own as well.
- `openmosh-hook` (no arguments) opens the TUI: status (`doctor`), agents and inbox, devices,
  pairing, integrations, setup and uninstall.
- **Open Herdr** in the app runs `openmosh-hook attach herdr`, which mirrors focus: the computer
  follows the tab and workspace picked on the phone, and the other way round.
- `openmosh-hook usage` reports token usage per agent, project and model (Codex only: Claude Code
  keeps no local usage summary, so it is not read).

## Lost phone

`openmosh-hook devices revoke <id>` (or `x` on the TUI Devices screen, or *Revoke* in the app)
removes the phone's key and push key, ends its live SSH sessions and prints a checklist of what
else to do.

## Uninstall

```sh
openmosh-hook uninstall --dry-run   # see exactly what would be removed
openmosh-hook uninstall             # service, integrations, openmosh: SSH keys, devices, state, config
openmosh-hook uninstall --all       # also deletes the binary
```

It only removes what OpenMosh created: `authorized_keys` lines tagged `openmosh:`, its own
integration entries, and the `openmosh` config and state directories.

## Self-hosting ntfy (optional)

If you don't want even ciphertext on `ntfy.sh`, run your own ntfy server inside your tailnet and
select it as the default server **in the ntfy app on the phone**. The phone hands its push endpoint
to the host during pairing, so nothing changes on the host.

## Recommended host hardening

`openmosh-hook doctor` warns when `sshd` accepts passwords. Key-only SSH is recommended:

```sh
openmosh-hook harden-ssh --dry-run   # see the file it writes
openmosh-hook harden-ssh             # asks, then uses sudo; also `s` on the TUI Status screen
```

It writes `/etc/ssh/sshd_config.d/10-openmosh.conf`, checks it with `sshd -t` and reloads sshd.
Logins with a password over SSH stop working; key logins and the local console do not change.
