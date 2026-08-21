---
title: "Graphify — local-only memory system setup"
tags: [graphify, session-log]
created: 2026-08-21
type: session-log
status: imported
---

# Local-only memory system setup

## Summary
Analyzed the `claude-code-memory-setup-main` reference project (Obsidian + Graphify +
chat-import pipeline). Identified security concerns in its automated path (unverified
browser extension, oddly-named `graphifyy` package, LLM semantic extraction sending code
to third-party providers including Moonshot, BRAT beta plugins, unattended cron). Built a
simpler, local-only replacement instead.

## Decisions
- See `decisions.md` in this folder for the two durable decisions from this session.

## Open items
- Confirm `/vault-save` and `/vault-resume` appear as slash commands after a fresh
  Claude Code session (skills were added mid-session, may need a reload to register).
- Decide later whether to expand into the reserved `permanent/inbox/fleeting/references`
  taxonomy, or add multi-language parsing improvements to `tools/local_graph.py`.

## Files touched
- `tools/local_graph.py` (new)
- `~/.claude/skills/vault-save/SKILL.md` (new, global)
- `~/.claude/skills/vault-resume/SKILL.md` (new, global)
- Obsidian vault: `CLAUDE.md`, `templates/default-note.md`, `Graphify/decisions.md`, this log
- Project `CLAUDE.md` and `AGENT_LOG.md` (pending update)
