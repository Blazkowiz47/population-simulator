---
name: remember
description: Use only when the user invokes /remember or asks to record project activity, a run, decision, learning, status change, blocker, next action, or compact daily memory. Do not use for unrelated reminders or task scheduling.
---

# Remember Project Knowledge

Follow the project `AGENTS.md` memory directives. This skill is a project-local shortcut; it does not override project instructions.

## Steps

1. Identify today's local date and canonical node.
2. Classify the input as activity, run, decision, learning, or status. Ask only when ambiguity would materially change the destination.
3. If the stable `<node>` is unknown, ask before creating or writing a new node-specific note.
4. Prefer exactly one `memory/notes/YYYY-MM-DD-<node>.md` daily note. Never create topic-suffixed daily-note filenames; use `memory/scratch/` for topic documents.
5. Continue writing `memory/notes/YYYY-MM-DD.md` only when that legacy file is already active for the date or the user explicitly asks to keep it.
6. Write once to the smallest canonical destination:
   - activity or compact daily context → today's note
   - notable run → `memory/runs.md`, with a short note link when useful
   - durable decision → `memory/decisions.md`
   - durable learning → `memory/learnings.md`
   - status, blocker, next action, or latest result → `memory/index.md`
7. Read an enabled integration file only when the capture may publish externally; missing adapters or authentication never block local memory.
8. Keep the update compact and link to evidence paths instead of pasting bulky output.
