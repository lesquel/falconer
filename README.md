# OpenMosh

**A free, open-source, self-hosted mobile terminal for AI coding agents.**

Supervise Claude Code, Codex, OpenCode and other agents from your phone: get a push when an
agent needs you, approve or deny with one tap, open the terminal, paste a screenshot.
You run everything: your phone, your host, your network. There is no OpenMosh cloud.

> Status: **v0.1 in development**. Android first, iOS later. Product and technical design documents are internal and not published.

## How it works

```text
phone (openmosh-app) ──SSH over Tailscale──► your host (Herdr/tmux + agents + openmosh-hook)
        ▲                                                    │
        └──────── encrypted push via ntfy (ciphertext) ◄─────┘
```

## Repositories

This is the umbrella repository. Components live in their own repos, referenced as git submodules:

| Path | Repo | What it is |
|---|---|---|
| [`app/`](./app) | `openmosh-app` | Flutter + Rust mobile app (Android first) |
| [`hook/`](./hook) | `openmosh-hook` | Rust daemon + CLI that runs on your host |
| [`relay/`](./relay) | `openmosh-relay` | Optional self-hosted push relay for iOS (Phase 2) |

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
| `*/ARCHITECTURE.md` | Clean Architecture by feature, per repo |

## Recommended stack for users

- **Tailscale** to reach your host without opening ports.
- **Herdr** to keep agents running and track their state (tmux also works).
- **ntfy** as the push transport (public `ntfy.sh` sees only ciphertext, or self-host it).

## License

Proposed: app GPL-3.0, hook and relay Apache-2.0. Pending confirmation.
