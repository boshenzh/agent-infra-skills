---
name: wecomcli-smartsheet
description: "Pointer skill — manage WeCom (企业微信) smartsheet structure (sub-sheets, fields/columns) and data (records add/update/delete/query) via the `wecom-cli doc smartsheet_*` API. The canonical SKILL.md is bundled with the wecom-cli installation at `<install>/skills/wecomcli-smartsheet/SKILL.md`, and also lives upstream at https://github.com/WecomTeam/wecom-cli/blob/main/skills/wecomcli-smartsheet/SKILL.md — install that one. This file exists so non-OpenClaw runtimes can discover the dependency. Use when the user asks to read or write 企微 smartsheets, add/rename/delete columns, manage sub-sheets, or works with 客户线索表 / 运价表 / 待审核 type sheets."
author: "boshenzh"
license: "Apache-2.0"
homepage: https://github.com/WecomTeam/wecom-cli
argument-hint: "see upstream"
allowed-tools: "Bash"
metadata:
  openclaw:
    requires:
      bins:
        - wecom-cli
---

# wecomcli-smartsheet (pointer)

This is a thin pointer. The canonical SKILL.md is maintained upstream by the wecom-cli project at:

**`WecomTeam/wecom-cli` → `skills/wecomcli-smartsheet/SKILL.md`**
https://github.com/WecomTeam/wecom-cli/blob/main/skills/wecomcli-smartsheet/SKILL.md

When wecom-cli is installed, that skill ships bundled — `wecom-cli` itself drops the SKILL.md into your agent's discovery path (Cursor / Claude Code / Windsurf etc.) automatically.

## When you don't need this file

Under OpenClaw with `wecom-cli` already installed: the upstream skill is auto-discoverable, this pointer is redundant.

Under Codex / Claude Code / Cursor / Windsurf with `wecom-cli` installed: `wecom-cli setup skills` (or equivalent) drops the upstream SKILL.md into the right place, this pointer is redundant.

This pointer exists for the rare case of an agent runtime that doesn't auto-discover bundled CLI skills — it makes the dependency explicit in `agent-infra-skills` plugin metadata.

## API surface — what wecom-cli supports

Per the upstream skill, `wecom-cli doc smartsheet_*` covers:

**Structure management (sub-sheets and fields/columns):**
- `smartsheet_get_sheet` — list all sub-sheets in a doc
- `smartsheet_add_sheet` — add a sub-sheet to a doc
- `smartsheet_update_sheet` — rename a sub-sheet
- `smartsheet_delete_sheet` — delete a sub-sheet (irreversible)
- `smartsheet_get_fields` — list columns in a sub-sheet
- `smartsheet_add_fields` — add columns (text / checkbox / option / etc.)
- `smartsheet_update_fields` — rename columns (cannot change type)
- `smartsheet_delete_fields` — delete columns (irreversible)

**Data management (records):**
- `smartsheet_get_records` — query records from a sub-sheet
- `smartsheet_add_records` — add records (no images/files)
- `+smartsheet_add_records_auto_file` — add records with local image/file paths (local helper, `+` prefix)
- `smartsheet_update_records` — update records by record_id
- `+smartsheet_update_records_auto_file` — update records with local image/file paths
- `smartsheet_delete_records` — delete records (irreversible; 极速版 smartsheets unsupported)

**Crucial:** `wecom-cli` does NOT have an API to create a top-level smartsheet doc. Docs are created by the operator in 企微 UI; the CLI manages structure and data *inside* an existing doc.

## Why a pointer instead of duplicating

`wecom-cli` is generic 企业微信 tooling, not specific to this plugin. Vendoring its SKILL.md here would create a maintenance fork (the upstream evolves with wecom-cli releases). The upstream is the source of truth; this pointer exists only for discovery.

The `freight-onboard` skill in `freight-skills` uses this API directly through `wecom-cli doc smartsheet_*` calls — it does not depend on this pointer file, but is documented to load the canonical upstream SKILL.md as a reference.
