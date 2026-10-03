<!-- BEGIN SUSHRUT MEMORY DIRECTIVES -->
## Sushrut Project Memory

This project participates in Sushrut's knowledge-base memory system. These directives govern memory tracking only. Preserve and follow all other project-specific instructions in this file.

### Project Memory Files

- Keep `memory/index.md` as the fast project overview.
- Keep today's project note updated during active work.
- Prefer `memory/notes/YYYY-MM-DD-<node>.md` for new notes when a stable device/server node name is known.
- Keep exactly one daily note per project/date/node. Put topic-specific or multi-document work in `memory/scratch/`; never encode a topic after the node suffix.
- Ask the user for the stable `<node>` name if it is not known before creating or writing a new node-specific note.
- Continue reading `memory/notes/YYYY-MM-DD.md` for legacy project memory.
- Continue writing `memory/notes/YYYY-MM-DD.md` only when that legacy file is already the active note for the target date or the user explicitly asks to keep the legacy convention.
- Treat `memory/notes/YYYY-MM-DD.md` as the legacy/default-node note and `memory/notes/YYYY-MM-DD-<node>.md` as explicit-node notes.
- Track devices, servers, and paths in `memory/devices.md`.
- Track experiments and long-running jobs in `memory/runs.md`.
- Track durable findings in `memory/learnings.md`.
- Track decisions and their rationale in `memory/decisions.md`.
- Use `memory/scratch/` for uncertain project-only captures and in-flight notes that are not ready for `notes/`, `runs.md`, `learnings.md`, `decisions.md`, or `index.md`.
- Scratch-only work should stay in `memory/scratch/`; do not create or update today's project note just because the user asked to work on scratch.
- Use `memory/integrations/` for optional external connections such as Jira, Confluence, GitHub Issues, Linear, Slack/Teams, or manual draft-only publishing.
- Use consolidated project-local skills for project-memory shortcuts when present:
  - Codex skills: `.agents/skills/<skill>/SKILL.md`
  - Claude skills: `.claude/skills/<skill>/SKILL.md`
- Treat `memory/commands/` as a legacy fallback only when project-local memory skills are missing.

### Project Memory Skills

If the user starts a prompt with a project memory shortcut, or explicitly asks to use or record project memory, use the matching local skill:

- `remember` - infer and record activity, runs, decisions, learnings, or status changes in the smallest canonical memory file.
- `recall` - retrieve relevant read-only context from locally registered project memory when available, otherwise from synchronized knowledge-base memory.
- `scratch` - capture an uncertain or in-flight project-local note.
- `organise-scratch` - route project-local scratch notes into the right memory files.
- `check-initialisation` - verify and align the project memory structure.

These skills are shortcuts. They do not override project-specific instructions or this `AGENTS.md`.

### When To Update Memory

- After every meaningful experiment, update today's note.
- After every meaningful analysis result, update today's note and `memory/learnings.md`.
- After every meaningful code change or commit, record what changed and why.
- When scratch work produces a durable result, promote it to today's note and the appropriate structured memory file; otherwise keep it in `memory/scratch/`.
- When using a new device, server, environment, or repo path, update `memory/devices.md`.
- When starting or finishing a long-running job, update `memory/runs.md`.
- When project status, blocker, latest useful result, or next action changes, update `memory/index.md`.
- Read an enabled integration file when a memory update may publish externally. Never let missing external tools or authentication block local memory updates.

### What To Record

- Node name when using a node-specific note
- Device/server
- Repo path
- Branch and commit when relevant
- Command/config
- Dataset or input source
- Output path
- Result summary
- Blocker
- Next action

### Style Rules

- Keep memory compact and chronological.
- Do not paste giant logs, full outputs, large tables, or raw dumps.
- Summarize the durable lesson and link to paths where evidence lives.
- Prefer clear next actions over vague observations.
- If unsure whether something matters, add it briefly to today's note when it is small; use `memory/scratch/` when it needs a separate in-flight note.
- If an enabled integration cannot publish from the current device, save the intended external update as a local draft and mark the publish status as skipped for this node.
<!-- END SUSHRUT MEMORY DIRECTIVES -->
