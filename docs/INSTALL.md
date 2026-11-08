# OpenMosh — Host setup

> The repositories are private for now, so the one-line `curl | sh` and release downloads need GitHub auth. Use `--from-source` or `--binary` meanwhile.
> Everything here runs on **your** host (laptop, workstation or VPS). Nothing runs on an OpenMosh server, because there is none.

## What you end up with

| Piece | Role | Required |
|---|---|---|
| `sshd` | The app connects here | Yes |
| Tailscale | Private network between phone and host, no public ports. Pairing refuses to run without it | Yes |
| Herdr (or tmux) | Keeps agents alive while the phone is away | Yes (one of them) |
| `openmosh-hook` | Watches agents, sends encrypted pushes, executes approvals | Yes |
| ntfy | Push transport. `ntfy.sh` (public, sees only ciphertext) or self-hosted | Yes |
| ntfy app on the phone | UnifiedPush distributor on Android | Yes |

## Quick install

```sh
curl -fsSL https://raw.githubusercontent.com/lesquel/openmosh-hook/main/install.sh | sh
```

The installer will:
1. Detect the OS and architecture and download the matching `openmosh-hook` release binary to `~/.local/bin`.
2. Check for `sshd`, Herdr or tmux, and Tailscale. Offer to install Herdr and Tailscale using their official installers.
3. Write `~/.config/openmosh/config.toml` (host alias, retention, sources).
4. Run `openmosh-hook service install` (systemd user service on Linux, launchd agent on macOS).
5. Run `openmosh-hook doctor` and print the next step: "Open OpenMosh on your phone and add this host".

It never asks for root unless a dependency's own installer does, and it prints every command before running it.

| Flag | Effect |
|---|---|
| `--dry-run` | Print every step, change nothing |
| `--from-source` | Build with `cargo install --git https://github.com/lesquel/openmosh-hook --locked` (uses your git credentials while the repo is private) |
| `--binary <path>` | Install a binary you already built |

From a local clone:

```sh
cd hook && cargo build --release --locked
sh install.sh --binary target/release/openmosh-hook
```

## Manual setup (today)

1. Install and log in to Tailscale on the host and on the phone:
   ```sh
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
2. Make sure SSH works with key authentication (password login disabled is recommended).
3. Install Herdr:
   ```sh
   curl -fsSL https://herdr.dev/install.sh | sh
   ```
   Or use tmux from your package manager.
4. Install the **ntfy** app on the phone (F-Droid or Play Store). OpenMosh uses it as the UnifiedPush distributor.
5. Install `openmosh-hook` (see Quick install) and run:
   ```sh
   openmosh-hook service install
   openmosh-hook doctor
   ```
6. Pair the phone with a QR (Tailscale must be up on both ends, same Tailscale user):
   ```sh
   openmosh-hook pair --alias my-host
   ```
   In the app: **Add host → Scan QR**. The QR expires in 5 minutes and works once.
   Alternative for a host you already reach over SSH: **Add host → Connect → Enable notifications** (SSH pairing).

## Self-hosting ntfy (optional)

If you don't want even ciphertext on `ntfy.sh`, run your own ntfy server (a single binary or Docker container) inside your tailnet and select it as the default server **in the ntfy app on the phone**. The phone hands its push endpoint to the host during pairing, so nothing changes on the host.

## Uninstall

```sh
openmosh-hook service uninstall
rm -rf ~/.local/bin/openmosh-hook ~/.config/openmosh ~/.local/state/openmosh
```
