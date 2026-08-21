---
title: "Graphify — decisions"
tags: [graphify, decisions]
created: 2026-08-21
updated: 2026-08-21
status: active
type: permanent
---

# Graphify — decisions

- **2026-08-21** — Rejected the `claude-code-memory-setup-main` pipeline's automated path
  (`claude-conversation-extractor`, a browser extension for Claude Web exports, `graphifyy`
  with LLM-based semantic extraction, cron automation, BRAT beta-plugin loading). Reasons:
  unverified browser extension with page-content access to claude.ai, an oddly-named pip
  package (`graphifyy`), and semantic extraction that would send project code to a
  third-party LLM (including Moonshot) — a data-governance risk on a corporate machine.
- **2026-08-21** — Adopted instead: this Obsidian vault managed only by two manually
  triggered Claude Code skills (`vault-save`, `vault-resume`, installed globally in
  `~/.claude/skills/`), plus a local, stdlib-only Python script
  (`tools/local_graph.py`) for a Graphify-equivalent structure map — no third-party
  packages, no network calls, no API keys, no cron.
