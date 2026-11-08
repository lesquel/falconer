# OpenMosh

**A free, open-source, self-hosted mobile terminal for AI coding agents.**

Supervise Claude Code, Codex, OpenCode and other agents from your phone: get a push when an
agent needs you, approve or deny with one tap, open the terminal, paste a screenshot.
You run everything: your phone, your host, your network. There is no OpenMosh cloud.

> Current version: **v0.2.0**. Android (9+) today, iOS later. Free and open source: app GPL-3.0,
> hook Apache-2.0.

## What you get

- A push when an agent needs you, end-to-end encrypted; approvals show the command being approved.
- Approve, deny or answer from the inbox or the notification; stale or expired approvals
  (older than 15 minutes) are refused by the host.
- A terminal built for the thumb: keys bar, saved shortcuts, chat with image and file attachments,
  pinch zoom, and **Open Herdr** with focus mirroring between the phone and the computer.
- Pairing by QR inside your tailnet, confirmed with `y/N` on the host; the phone's key is limited
  to `restrict,pty,from="<phone ip>"`.
- Agent usage, notification hours, app lock, English and Spanish.

Supported agents: Claude Code, Codex CLI, Gemini CLI, Cursor, GitHub Copilot CLI, Kiro CLI,
OpenCode and Aider; Amp through Herdr state only.

## How it works

```text
phone (openmosh-app) ──SSH over Tailscale──► your host (Herdr/tmux + agents + openmosh-hook)
        ▲                                                    │
        └──────── encrypted push via ntfy (ciphertext) ◄─────┘
```

- **app** connects over SSH with a key kept in the Android Keystore and a pinned host key, runs
  `openmosh-hook` commands for sessions, inbox, answers and usage, and decrypts pushes on the phone.
- **hook** watches the agents (Herdr or tmux and the agents' own hooks), keeps the inbox, sends
  encrypted pushes and types the answers into the agent's pane. It opens no permanent port.
- **relay** (iOS, not started) would forward the already encrypted push to APNs.
- **web** is the static landing page.

## Quick install

```sh
# host
curl -fsSL https://raw.githubusercontent.com/lesquel/openmosh-hook/main/install.sh | sh
openmosh-hook setup
openmosh-hook pair
```

Then install the APK from the [latest app release](https://github.com/lesquel/openmosh-app/releases/latest)
and scan the code. The full guide (Tailscale, ntfy, uninstall, hardening) is
[`docs/INSTALL.md`](./docs/INSTALL.md).

## Release channels

| Channel | Tags | Where |
|---|---|---|
| Stable | `vX.Y.Z` | `/releases/latest` of each repo; the installer's default |
| Pre-release | `vX.Y.Z-rc.N` | GitHub pre-releases, never "latest"; `install.sh --version vX.Y.Z-rc.N` |

## Repositories

This is the umbrella repository. Components live in their own repos, referenced as git submodules:

| Path | Repo | What it is |
|---|---|---|
| [`app/`](./app) | [`openmosh-app`](https://github.com/lesquel/openmosh-app/tree/main) | Flutter + Rust mobile app (Android first) |
| [`hook/`](./hook) | [`openmosh-hook`](https://github.com/lesquel/openmosh-hook/tree/main) | Rust daemon + CLI that runs on your host |
| [`relay/`](./relay) | [`openmosh-relay`](https://github.com/lesquel/openmosh-relay/tree/main) | Optional self-hosted push relay for iOS (Phase 2, not started) |
| [`web/`](./web) | [`openmosh-web`](https://github.com/lesquel/openmosh-web/tree/main) | Landing page (Astro, static, no trackers) |

Clone everything:

```sh
git clone --recurse-submodules https://github.com/lesquel/OpenMosh.git
# already cloned?
git submodule update --init --recursive
```

## Documentation

| Doc | Content |
|---|---|
| [`docs/INSTALL.md`](./docs/INSTALL.md) | Host setup: Tailscale, Herdr, ntfy, `openmosh-hook` |
| [hook README](https://github.com/lesquel/openmosh-hook/blob/main/README.md) | Every command, pairing, security rules, files |
| [app README](https://github.com/lesquel/openmosh-app/blob/main/README.md) | Features, security properties, development |
| `*/ARCHITECTURE.md` | Clean Architecture by feature, per repo |

## Recommended stack for users

- **Tailscale** to reach your host without opening ports.
- **Herdr** to keep agents running and track their state (tmux also works).
- **ntfy** as the push transport (public `ntfy.sh` sees only ciphertext, or self-host it).
  Optional: without it the app pairs with notifications off.

## License

App GPL-3.0, hook Apache-2.0 (each repo's `LICENSE`). Relay: proposed Apache-2.0.
