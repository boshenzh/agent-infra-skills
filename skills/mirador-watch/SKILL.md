---
name: mirador-watch
description: "Set up or operate an IMAP mailbox watcher that fires an agent webhook on each new email. Use when the user asks to monitor an inbox, trigger an agent on incoming mail, or wire IMAP IDLE to a webhook. Does NOT read, send, or search mail — use the himalaya skill for those."
author: "boshenzh"
license: "Apache-2.0"
argument-hint: "<start|stop|status|logs|doctor|test-hook> [account]"
allowed-tools: "Read Write Edit Bash"
homepage: https://github.com/pimalaya/mirador
metadata:
  openclaw:
    requires:
      bins:
        - mirador
    install:
      - kind: cargo
        bins: [mirador]
        git: https://github.com/pimalaya/mirador.git
        features: [imap, keyring]
        locked: true
---

# Mirador inbox watcher

Drives [pimalaya/mirador](https://github.com/pimalaya/mirador) to hold an IMAP IDLE connection and shell out on every new message. **Companion skill**: [openclaw/openclaw → `skills/himalaya`](https://github.com/openclaw/openclaw/blob/main/skills/himalaya/SKILL.md) for reading/sending mail in the agent turn after the watcher fires.

## Setup checklist

When the user wants to set up a new watcher, follow this order:

- [ ] 1. Verify `mirador --version` reports `+imap +keyring`. If missing, see `references/install.md`.
- [ ] 2. Store mailbox credential in OS secret store. See `references/install.md` for the platform default.
- [ ] 3. Create `~/.config/mirador/config.toml` from `references/config-template.toml` and fill `<placeholders>`.
- [ ] 4. Validate: `mirador doctor <account>`. Expect `Account <account> is well configured!`.
- [ ] 5. Dry-run the hook: `mirador-watch test-hook <account>`. Confirm the webhook target receives a synthetic event.
- [ ] 6. Start watching: `mirador-watch start <account>`. Tail the log for `entering idle mode`.

Provider-specific config examples for Aliyun enterprise mail, Gmail, 163, exmail.qq: `references/providers.md`.

## Commands

| Subcommand | Implementation | Notes |
|---|---|---|
| `start <account>` | `nohup mirador watch <account> INBOX > ~/Library/Logs/mirador/<account>.log 2>&1 &` then write PID to `~/Library/Logs/mirador/<account>.pid` | One process per account |
| `stop <account>` | `kill -INT $(cat ~/Library/Logs/mirador/<account>.pid)` then `rm` the pid file | SIGINT for clean IDLE LOGOUT |
| `status` | `pgrep -fl "mirador watch"` | Lists running watchers |
| `logs <account>` | `tail -50 ~/Library/Logs/mirador/<account>.log` | Last 50 log lines |
| `doctor <account>` | `mirador doctor <account>` | Validates config + IMAP connect, does NOT test the hook |
| `test-hook <account>` | Parse `on-message-added.cmd` from `~/.config/mirador/config.toml`, substitute placeholders with dummy values (`{id}=test-1`, `{subject}=test`, `{sender.address}=tester@example.com`), execute. | Verify webhook path before going live |

For production deployment with launchd/systemd, load `references/production.md` only when the user explicitly asks about auto-start/auto-restart.

## Gotchas

These are concrete corrections to mistakes the agent will make without being told:

- **Install `--locked` is required.** Without it, cargo resolves `toml 0.9.x` and breaks `pimalaya-tui 0.2.2`. Yanked-keccak warning is non-fatal. Full command: `cargo install --git https://github.com/pimalaya/mirador.git --features imap,keyring --locked`.

- **Config field `email` does NOT exist in mirador.** Valid top-level keys under `[accounts.<name>]` are only `default`, `folder`, `backend`, `on-message-added`. Copying from himalaya configs will fail `mirador doctor` with `unknown field 'email'`.

- **`on-message-added.cmd` has only 8 placeholders**: `{id}`, `{subject}`, `{sender}`, `{sender.name}`, `{sender.address}`, `{recipient}`, `{recipient.name}`, `{recipient.address}`. **No `{body}`, no `{date}`, no `{message-id}`.** If the hook needs body, the wake event carries `{id}` and the next agent turn fetches via `himalaya message read <id>`.

- **Aliyun enterprise mail IDLE drops at ~5 minutes**, not the standard 29 minutes. Mirador reconnects automatically — log will show "starting new IMAP IDLE loop…" every ~5 min. Not an error. Gmail and 163 stay closer to 29 min.

- **Aliyun + 163 don't advertise `AUTH=PLAIN` or `AUTH=LOGIN`** in CAPABILITY, only `XOAUTH/XOAUTH2/EXTERNAL`. Mirador tries those, fails (no OAuth tokens), then falls back to the bare `LOGIN` command which works. Debug log shows three "trying auth mechanism…" failures before "login succeeded!" — this is normal.

- **`WARN imap_codec: Rectified missing 'text' to "..."` and `missing required UNSEEN OK untagged response`** appear on every `SELECT INBOX` against Aliyun. Non-fatal — the imap-codec parser auto-corrects. Same library backs himalaya so himalaya logs identical warnings. Do not chase these.

- **One mirador process watches one folder.** N folders / N accounts = N processes. Use the pid-file pattern in `start`/`stop` above to track them. Single config file can hold multiple `[accounts.<name>]` sections, but each needs its own watcher process.

- **Background with `&` is PoC only.** For production, use launchd (macOS) or systemd-user (Linux) — see `references/production.md`. A bare `&` dies on terminal close.

- **`on-message-added.notify.*` triggers the macOS system notification center** in addition to running `cmd`. If you only want the webhook fired (no desktop notification), omit the `notify.summary` / `notify.body` keys entirely.

## When NOT to use this skill

- User wants to **read / send / search / move / forward** mail → use the [`himalaya`](https://github.com/openclaw/openclaw/blob/main/skills/himalaya/SKILL.md) skill instead.
- Mailbox provider exposes a native push webhook (Gmail Watch + Pub/Sub, Microsoft Graph subscriptions) AND user has the cloud infra to receive it → that's more reliable than IDLE long-poll.
- User wants a single-shot "check the inbox now" → just call himalaya directly, no watcher needed.

## Cross-runtime note

The body of this skill makes no OpenClaw-specific calls — every step is a plain shell command driving the `mirador` CLI. The `metadata.openclaw` block above only hints OpenClaw's auto-install. Codex / Hermes Agent / Claude Code users: install mirador via `references/install.md` and the rest applies verbatim.
