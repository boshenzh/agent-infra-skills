# agent-infra-skills

Cross-business agent infrastructure skills for OpenClaw and compatible agent runtimes (Codex, Hermes Agent, Claude Code). Layer 1 in a three-layer architecture:

```
Layer 1 · agent-infra-skills   ← you are here (universal infrastructure)
Layer 2 · <industry>-pp-cli     ← e.g. ocean-pp-cli (industry CLI + wrapper skills)
Layer 3 · <industry>-<company>  ← company-specific scenario playbooks (private)
```

## Skills shipped

| Skill | Wraps | Purpose |
|---|---|---|
| `mirador-watch` | [pimalaya/mirador](https://github.com/pimalaya/mirador) | Hold an IMAP IDLE long-connection on any mailbox; fire a shell command on each new message (typically `curl POST` to an agent's wake webhook). |
| `himalaya` | [pimalaya/himalaya](https://github.com/pimalaya/himalaya) — thin pointer to upstream [openclaw/openclaw/skills/himalaya](https://github.com/openclaw/openclaw/blob/main/skills/himalaya/SKILL.md) | Read / send / search / move / delete IMAP+SMTP mail. |
| `wecomcli-smartsheet` | [WecomTeam/wecom-cli](https://github.com/WecomTeam/wecom-cli) — thin pointer to upstream [skills/wecomcli-smartsheet/SKILL.md](https://github.com/WecomTeam/wecom-cli/blob/main/skills/wecomcli-smartsheet/SKILL.md) | Manage WeCom (企业微信) smartsheet structure (sub-sheets, fields/columns) and data (records add/update/delete/query) via `wecom-cli doc smartsheet_*`. |

## Roadmap

- `telegram-send` — outbound notification to Telegram chat
- `dingtalk-send` / `feishu-send` — group robot notifications

## Install

### OpenClaw

```bash
git clone https://github.com/boshenzh/agent-infra-skills.git
openclaw plugins install -l ./agent-infra-skills
openclaw config set plugins.allow '["agent-infra-skills"]'
```

### Other runtimes (manual)

```bash
git clone https://github.com/boshenzh/agent-infra-skills.git
# Then symlink the relevant SKILL.md into the runtime's skill discovery directory:
#   Codex:        ~/.codex/skills/<name>/SKILL.md
#   Claude Code:  ~/.claude/skills/<name>/SKILL.md
#   Hermes:       (verify upstream docs)
```

The skill bodies are runtime-agnostic — every operation is a plain shell command driving the underlying CLI. Only the `metadata.openclaw.*` block in YAML frontmatter is runtime-specific; other runtimes ignore unknown keys.

## CLI prerequisites

| Tool | Install |
|---|---|
| mirador | `cargo install --git https://github.com/pimalaya/mirador.git --features imap,keyring --locked` |
| himalaya | `brew install himalaya` (or see [upstream install docs](https://github.com/pimalaya/himalaya#installation)) |

## License

Apache-2.0
