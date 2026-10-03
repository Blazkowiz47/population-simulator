# Project Integrations

External integrations are optional connections from this project's local memory to shared systems such as Jira, Confluence, GitHub Issues, Linear, Slack/Teams, or manual draft-only updates.

Local project memory remains canonical. Missing tools, missing authentication, or unavailable network access must not block local memory updates.

## Enabled Integrations

| Integration | Provider | Adapter | Status | Capabilities | Policy |
|---|---|---|---|---|---|

## Device Capability Notes

| Node | Integration | Capability | Status | Last checked |
|---|---|---|---|---|

## Draft And Fallback Policy

- If an integration is enabled but unavailable on the current node, skip external publishing and save a draft locally.
- Record skipped publishes in the relevant integration file or draft.
- Never publish secrets, credentials, raw logs, bulky outputs, or private working notes.
- Publish only concise, stakeholder-safe updates unless the user explicitly asks for a fuller draft.

## Adding An Integration

When adding an integration, record:

- Provider and adapter, for example Jira or Confluence via `atlassian-rovo-mcp`
- Enabled capabilities, for example `create-ticket` or `add-comment`
- Linked external objects, for example issues, epics, pages, channels, or documents
- Publish policy, including whether updates are draft-first or auto-published
- Device/node availability and fallback behavior
