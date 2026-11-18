# Contributing to Falconer

Thanks for helping. Falconer is split into several repositories; this one (the umbrella) ties them together as
git submodules and holds the shared docs.

## How the repositories fit together

| Repository | What it is | Language |
|---|---|---|
| [falconer-app](https://github.com/lesquel/falconer-app) | Android app: inbox, approvals, terminal | Flutter + Rust |
| [falconer-hook](https://github.com/lesquel/falconer-hook) | Daemon, CLI and TUI on your computer | Rust |
| [falconer-relay](https://github.com/lesquel/falconer-relay) | iOS push relay (planned, not started) | Rust |
| [falconer-web](https://github.com/lesquel/falconer-web) | Landing page and `/notify/` PWA | Astro + TypeScript |
| [falconer](https://github.com/lesquel/falconer) | Umbrella: submodules and shared docs | Markdown |

The app talks to the hook over SSH by running `falconer-hook` commands, so a change to a command's output is a
contract change between the two repositories: update both, and say so in the pull request.

## Get the code

```sh
git clone --recurse-submodules https://github.com/lesquel/falconer.git
cd falconer
git submodule update --init --recursive   # if you cloned without --recurse-submodules
```

Each submodule is a normal clone of its repository: create branches, commit and open pull requests inside it.

## Where to open issues and pull requests

Open them in the repository that owns the code. Use the umbrella only for the shared docs or when you are not
sure where something belongs. Each repository has its own guide:

- [falconer-app CONTRIBUTING](https://github.com/lesquel/falconer-app/blob/main/CONTRIBUTING.md)
- [falconer-hook CONTRIBUTING](https://github.com/lesquel/falconer-hook/blob/main/CONTRIBUTING.md)
- [falconer-relay CONTRIBUTING](https://github.com/lesquel/falconer-relay/blob/main/CONTRIBUTING.md)
- [falconer-web CONTRIBUTING](https://github.com/lesquel/falconer-web/blob/main/CONTRIBUTING.md)

## Ground rules

- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:` …), one logical change per commit.
- Tests and docs change together with the behavior.
- Never commit secrets, real IP addresses, hostnames or personal data. Use examples such as `100.88.77.66` and
  `my-computer.tailnet-example.ts.net`.

## Security and conduct

Report vulnerabilities privately as described in [SECURITY.md](./SECURITY.md), never in a public issue.
Everyone taking part follows the [Code of Conduct](./CODE_OF_CONDUCT.md).
