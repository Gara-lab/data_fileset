---
title: "Vault & Local Memory System — Overview"
tags: [graphify, obsidian, documentation]
created: 2026-08-21
updated: 2026-08-21
status: active
type: reference
---

# Vault & Local Memory System — Overview

This is the human-readable guide to what's been set up. For the machine-facing rules
Claude Code itself follows, see [[CLAUDE]] (this vault's root `CLAUDE.md`).

## Why this exists

Claude Code forgets everything between sessions, and re-explaining a project's state
every time wastes time and tokens. A reference setup (`claude-code-memory-setup-main`,
kept in the Graphify project folder for record) solved this with Obsidian + a
"Graphify" codebase-graph tool + an automated chat-export pipeline. After reviewing it,
the automated pipeline was judged too risky to adopt as-is (a browser extension with
broad access to claude.ai, an oddly-named third-party package, code potentially sent to
third-party LLMs, unattended cron jobs). The full reasoning is logged in
[[Graphify/decisions|Graphify/decisions.md]].

What's built instead is a **minimal, local-only, manually-triggered** version that
covers the same goal — continuity across sessions, plus a structural map of a codebase —
without any of that risk surface.

## Current state

- ✅ Vault folder structure in place (see below).
- ✅ Two global Claude Code skills installed (`vault-save`, `vault-resume`) — confirmed
  working as `/vault-save` and `/vault-resume` slash commands in any project, in any new
  session.
- ✅ A local, dependency-free codebase structure map tool (`tools/local_graph.py`) inside
  the Graphify project.
- ✅ First real round trip done: a session log exists at
  [[Graphify/logs/2026-08-21-memory-system-setup]].
- ⏳ Not yet used: the `permanent/`, `inbox/`, `fleeting/`, `references/` folders are
  reserved for future use but currently empty — nothing requires you to fill them in.

## What you can do

### `/vault-save` — save the current session's progress
Type `/vault-save` in Claude Code (in any project) when you want to record what
happened. Claude will:
- Write a dated note to `<ProjectName>/logs/` in this vault summarizing the session.
- Add any new durable decision to `<ProjectName>/decisions.md`.

Nothing is saved automatically — this only happens when you ask for it.

### `/vault-resume` — catch up at the start of a session
Type `/vault-resume` when opening a project fresh. Claude will read that project's
`decisions.md` and its most recent logs in this vault, then summarize where things
stand before you ask anything else.

### Running the structure map
From the Graphify project folder, run:
```
python tools/local_graph.py .
```
This regenerates `graphify-out/graph.json` (a machine-readable map of files, functions/
classes, and import relationships) and `graphify-out/GRAPH_REPORT.md` (a readable
summary — file counts, most-imported files, per-file def counts). Nothing runs
automatically; re-run it yourself after significant code changes. It only understands
Python precisely (via the `ast` module); other languages get a best-effort regex scan,
which is less accurate.

## Vault folder structure

| Folder | Purpose |
|---|---|
| `permanent/` | Durable, atomic notes — one idea per note. Empty until you start using it. |
| `inbox/` | Raw, unsorted capture — sort into `permanent/` or a project folder later. |
| `fleeting/` | Scratch notes, safe to delete once acted on. |
| `references/` | External material worth keeping. |
| `templates/` | Note templates, e.g. `default-note.md`. |
| `<ProjectName>/` | One folder per project (e.g. `Graphify/`), auto-created by `/vault-save` on first use. |
| `<ProjectName>/architecture/` | Structural/design notes for that project. |
| `<ProjectName>/features/` | Planned or implemented features. |
| `<ProjectName>/logs/` | One file per `/vault-save`, dated. |
| `<ProjectName>/decisions.md` | Running list of durable decisions for that project. |

## Where things live on disk

- Vault: `C:\Users\L_FORM14\OneDrive - ohm-energie.com\Documents\Obsidian Vault`
- Skills (apply to every project): `C:\Users\L_FORM14\.claude\skills\vault-save\SKILL.md`
  and `...\vault-resume\SKILL.md`
- Structure-map tool (Graphify project only, copy elsewhere if you want it per-project):
  `tools/local_graph.py`

## Limitations & possible next steps

- No chat archive and no auto-generated per-function notes — the original reference
  project had both, but both required the risky automated pipeline. Revisit only if you
  decide that trade-off is worth it later, as its own explicit decision.
- `tools/local_graph.py`'s non-Python parsing is a simple regex scan, not a real parser —
  it can miss or misidentify defs/imports in JS/TS/Go/etc. Fine for a rough map, not a
  precise one.
- The `permanent/inbox/fleeting/references` folders are reserved but unused; adopting a
  fuller Zettelkasten workflow later is additive, not a restructure.
