# Falconer

**A free, open-source, self-hosted mobile terminal for AI coding agents.**

Supervise Claude Code, Codex, OpenCode and other agents from your phone: get a push when an agent needs you,
approve or deny with one tap, open the terminal, paste a screenshot. You run everything: your phone, your
computer, your tailnet. There is no Falconer cloud.

> Current version: **v0.3.0**. Android 9+ today, iOS later. App GPL-3.0, hook Apache-2.0.

**Demo (4:50):** [docs/media/falconer-demo.mp4](docs/media/falconer-demo.mp4): browse Herdr agents, fix a bug from
the phone, approve a permission card, dictate a prompt, open the terminal, change colors.

## What it is

Falconer has two parts you install:

- **falconer-hook** on your computer (Linux or macOS): watches your coding agents, keeps an inbox of what they
  need, sends encrypted pushes and types your answers into the agent's terminal.
- **Falconer** on your Android phone: receives the pushes, shows the inbox and gives you a full terminal into
  your computer over SSH.

The phone and the computer talk directly over your [Tailscale](https://tailscale.com) network. The phone and
the computer both need a connection to it.

## What you get

- A push when an agent needs you, end-to-end encrypted; approvals show the command being approved.
- Approve, deny or answer from the inbox or the notification; stale or expired approvals (older than 15 minutes)
  are refused by the host.
- A terminal built for the thumb: keys bar, saved shortcuts, chat with image and file attachments, pinch zoom,
  and **Open Herdr** with focus mirroring between the phone and the computer.
- Pairing by QR inside your tailnet, confirmed with `y/N` on the host; the phone's key is limited to
  `restrict,pty,from="<phone ip>"`.
- Agent usage, notification hours, app lock, English and Spanish.

**Supported agents:** Claude Code, Codex CLI, Gemini CLI, Cursor, GitHub Copilot CLI, Kiro CLI, OpenCode and
Aider through their own hooks; Amp and Pi through Herdr.

## How it works

```text
┌──────────────────┐   SSH over Tailscale (pinned host key)   ┌───────────────────────────────────┐
│ phone            │ ───────────────────────────────────────► │ your computer                     │
│ Falconer app     │                                          │ sshd + Herdr/tmux + agents        │
│ (falconer-app)   │ ◄─── end-to-end encrypted push ───────── │ falconer-hook                     │
└──────────────────┘      via ntfy (sees ciphertext only)     └───────────────────────────────────┘
```

- **app** connects over SSH with a key kept in the Android Keystore and a pinned host key, runs `falconer-hook`
  commands for sessions, inbox, answers and usage, and decrypts pushes on the phone.
- **hook** watches the agents (Herdr or tmux and the agents' own hooks), keeps the inbox, sends encrypted pushes
  and types the answers into the agent's pane. It opens no port of its own.
- **relay** (iOS, planned, not started) would forward the already encrypted push to APNs.
- **web** is the static landing page and the `/notify/` web-push page.

## Quick start

On your computer, with Tailscale logged in:

```sh
curl -fsSL https://raw.githubusercontent.com/lesquel/falconer-hook/main/install.sh | sh
falconer-hook setup
falconer-hook pair --alias my-computer
```

Prefer to read the installer first? Download the tag-pinned script, inspect it, then run it:

```sh
curl -fsSLO https://raw.githubusercontent.com/lesquel/falconer-hook/v0.3.0/install.sh
less install.sh
sh install.sh
```

The installer verifies the release checksums and its minisign signature before installing anything.

On the phone, install the APK from the [latest app release](https://github.com/lesquel/falconer-app/releases/latest),
then tap **Scan pairing code**. The full guide (Tailscale, ntfy, uninstall, hardening) is
[`docs/INSTALL.md`](./docs/INSTALL.md).

## Release channels

| Channel | Tags | Where |
|---|---|---|
| Stable | `vX.Y.Z` | `/releases/latest` of each repo; the installer's default |
| Pre-release | `vX.Y.Z-rc.N` | GitHub pre-releases, never "latest"; `install.sh --version vX.Y.Z-rc.N` |

## Repositories

This is the umbrella repository. Components live in their own repositories, referenced as git submodules:

| Path | Repository | What it is |
|---|---|---|
| [`app/`](./app) | [`falconer-app`](https://github.com/lesquel/falconer-app) | Flutter + Rust mobile app (Android first) |
| [`hook/`](./hook) | [`falconer-hook`](https://github.com/lesquel/falconer-hook) | Rust daemon + CLI + TUI that runs on your computer |
| [`relay/`](./relay) | [`falconer-relay`](https://github.com/lesquel/falconer-relay) | Optional self-hosted push relay for iOS (planned, not started) |
| [`web/`](./web) | [`falconer-web`](https://github.com/lesquel/falconer-web) | Landing page and `/notify/` PWA (Astro, static, no trackers) |

Clone everything:

```sh
git clone --recurse-submodules https://github.com/lesquel/falconer.git
# already cloned?
git submodule update --init --recursive
```

## Documentation

| Doc | Content |
|---|---|
| [`docs/INSTALL.md`](./docs/INSTALL.md) | Computer setup: Tailscale, Herdr, ntfy, `falconer-hook` |
| [hook README](https://github.com/lesquel/falconer-hook/blob/main/README.md) | Every command, pairing, security rules, files |
| [app README](https://github.com/lesquel/falconer-app/blob/main/README.md) | Features, security properties, development |
| `*/ARCHITECTURE.md` | Clean Architecture by feature, per repository |

## Security

- Everything travels over SSH inside your tailnet; the app pins each computer's host key and warns if it
  changes.
- Pushes are end-to-end encrypted; ntfy only sees ciphertext.
- A paired phone has a full shell on your computer, by design. Protect the phone (screen lock, app lock) and
  revoke a lost one with `falconer-hook devices revoke <id>`.

The threat model and how to report a vulnerability privately are in [SECURITY.md](./SECURITY.md).

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how the repositories fit together and where to open issues and pull
requests. Everyone taking part follows the [Code of Conduct](./CODE_OF_CONDUCT.md).

## Recommended stack

- **Tailscale** to reach your computer without opening ports.
- **Herdr** to keep agents running and track their state (tmux also works).
- **ntfy** as the push transport (public `ntfy.sh` sees only ciphertext, or self-host it). Optional: without it
  the app pairs with notifications off.

## License

App GPL-3.0; hook, web and this repository Apache-2.0 (each repository's `LICENSE`). Relay: proposed Apache-2.0, no code yet.
