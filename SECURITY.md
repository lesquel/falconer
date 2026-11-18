# Security policy

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub, in the repository where the problem lives. Do not open a
public issue.

| Component | Report privately |
|---|---|
| Mobile app | [falconer-app](https://github.com/lesquel/falconer-app/security/advisories/new) |
| Host daemon and CLI | [falconer-hook](https://github.com/lesquel/falconer-hook/security/advisories/new) |
| iOS push relay (planned) | [falconer-relay](https://github.com/lesquel/falconer-relay/security/advisories/new) |
| Landing page and `/notify/` | [falconer-web](https://github.com/lesquel/falconer-web/security/advisories/new) |
| Docs, or not sure which | [falconer](https://github.com/lesquel/falconer/security/advisories/new) |

Include the version, the steps to reproduce and the impact you expect. Fixes ship in the next release of the
affected repository.

## Supported versions

Only the latest stable release (`vX.Y.Z`, shown as "latest" on each repository's releases page) gets security
fixes.

## Threat model in short

- **Transport.** The app reaches your computer over SSH inside your Tailscale network. Pairing refuses to run
  without Tailscale and the computer opens no port of its own.
- **Host keys are pinned.** The app stores each computer's SSH host key at pairing and warns if it changes.
- **Phone keys.** The phone's SSH key lives in the Android Keystore. On the computer it is added to
  `authorized_keys` as `restrict,pty,from="<phone ip>"`, after a `y/N` confirmation on the computer.
- **Pushes are end-to-end encrypted.** The hook encrypts each push for the phone; ntfy (public or self-hosted)
  only sees ciphertext. Notifications carry no commands or hostnames in clear.
- **Approvals are checked on the computer.** Stale or expired approvals are refused.
- **A paired phone has a full shell, by design.** Anyone who can unlock the phone and the app can run commands
  on your computer. Use a screen lock and the app lock, and revoke a lost phone with
  `falconer-hook devices revoke <id>`, which removes its key and ends its live sessions.
- **Releases are signed.** The hook installer verifies `SHA256SUMS` and its minisign signature before installing.

Out of scope: attacks that require control of your Tailscale account, your computer or your unlocked phone.
