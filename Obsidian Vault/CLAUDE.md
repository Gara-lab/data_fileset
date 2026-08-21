# Vault — instructions for Claude Code

This vault is cross-project, local-only memory for Claude Code sessions. It replaces
any chat-extractor / browser-extension / cron pipeline with two manually-triggered
skills: `vault-save` and `vault-resume` (installed globally in `~/.claude/skills/`).
Nothing writes here automatically — a note only appears when you ask for `/vault-save`.

## Folder structure

- `permanent/` — atomic, durable notes (one concept per note). Fill in over time as
  ideas prove worth keeping long-term; empty is fine to start.
- `inbox/` — raw, unsorted capture. Triage into `permanent/` or a project folder later.
- `fleeting/` — short-lived scratch notes, safe to delete once acted on.
- `references/` — external material worth keeping (links, excerpts, docs).
- `templates/` — note templates, e.g. `default-note.md`.
- `<ProjectName>/` — one folder per project, named after the project's root folder name.
  - `architecture/` — structural/design notes for that project.
  - `features/` — planned or implemented features.
  - `logs/` — session logs, one file per `/vault-save`, named `YYYY-MM-DD-<slug>.md`.
  - `decisions.md` — running list of durable decisions for that project, kept short.

## Note conventions

- Frontmatter on every note: `title`, `tags`, `created`, `type`, `status`.
- Use `[[wikilinks]]` for links to other notes in this vault.
- Filenames in kebab-case.

## Skills that manage this vault

- `/vault-save` — run at the end of a session. Writes a dated log note under
  `<ProjectName>/logs/` and appends any new durable decisions to
  `<ProjectName>/decisions.md`.
- `/vault-resume` — run at the start of a session. Reads `<ProjectName>/decisions.md`
  and the most recent logs, then summarizes current state.

## Never do

- Don't add an automated chat-export or cron pipeline into this vault — that trade-off
  (third-party extractor + browser extension) was deliberately rejected in favor of the
  manual skills above. See `Graphify/decisions.md` for the reasoning.
- Don't delete notes without asking.
