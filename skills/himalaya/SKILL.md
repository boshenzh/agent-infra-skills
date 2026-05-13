---
name: himalaya
description: "Pointer skill — use himalaya CLI to list/read/search/compose/reply/forward/organize IMAP+SMTP email. The canonical SKILL.md is maintained upstream by the OpenClaw project; install that one. This file exists so non-OpenClaw runtimes can discover the dependency."
author: "boshenzh"
license: "Apache-2.0"
homepage: https://github.com/pimalaya/himalaya
argument-hint: "see upstream"
allowed-tools: "Bash"
metadata:
  openclaw:
    requires:
      bins:
        - himalaya
    install:
      - kind: brew
        bins: [himalaya]
---

# Himalaya (pointer)

This is a thin pointer. The canonical himalaya SKILL.md is maintained upstream at:

**`openclaw/openclaw` → `skills/himalaya/SKILL.md`**
https://github.com/openclaw/openclaw/blob/main/skills/himalaya/SKILL.md

Under OpenClaw, that skill ships bundled — you do not need this file. Under Codex / Hermes Agent / Claude Code, install the himalaya CLI (`brew install himalaya`) and copy the upstream SKILL.md into your runtime's skill directory (e.g. `~/.codex/skills/himalaya/SKILL.md`).

The mirador-watch skill in this same plugin assumes himalaya is available for reading individual messages after a watcher fires. If himalaya is not installed, mirador-watch's "trigger → read" loop has no read step.

## Why a pointer instead of duplicating

Himalaya is generic email tooling, not specific to this plugin. Vendoring its SKILL.md here would create a maintenance fork. The upstream is the source of truth; this pointer exists only for discovery.
