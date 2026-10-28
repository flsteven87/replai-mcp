---
name: replai-setup
description: "Set up and run a Replai AI customer support Agent through Replai's MCP server: connect, choose the Workspace, add knowledge, preview answers, connect a channel, publish, and work the Inbox. Use when the user asks to do this in Replai."
---

# Set up and run a Replai support Agent

Replai gives SaaS teams an AI customer support Agent grounded in their product knowledge.
It answers on a website chat widget or LINE and hands conversations to the team's Inbox when
a person is needed.
This skill covers doing that setup with Replai's remote MCP server.
Guide: https://replai.tw/en/guide/connect-ai-agents

## 1. Connect

MCP server: `https://api.replai.tw/mcp` (Streamable HTTP, OAuth).

Use the Replai MCP connection included with the assistant's installed package. For other clients,
add the server URL in the client's MCP settings with OAuth authentication. The connection guide
above has client-specific registration instructions.

The first connection opens the user's browser. The user signs in to Replai with Google, chooses
one Workspace and approves the requested permissions. This step belongs to the user; do not try
to complete it for them, and never ask for passwords or API keys.

Use only the connected Replai MCP tools. No local software installation or shell execution is needed.

## 2. Confirm the Workspace

Call `connection_context` first. It returns the one Workspace this connection can reach and the
granted operation scopes. Each connection reaches exactly one Workspace; to work in another one,
the user reconnects and chooses it. If a scope you need is missing, tell the user which
permission to approve.

## 3. Create or choose an Agent

Call `agents_list` and reuse an existing Agent when one fits; refer to Agents by ID, because
names can repeat. Otherwise call `agent_create`. Read the draft with `agent_draft_read` and
change it with `agent_draft_edit`. Draft edits never reach customers until publication.

## 4. Add knowledge

- Website: `website_discover`, then `website_import` with the pages to import.
- Text or a single URL: `source_add`.
- Google Drive or GitHub: check `provider_status`; if not connected, `provider_connect_handoff`
  returns a Replai link for the user to open. Follow it with the tool its `readback_tool` names
  (`provider_connect_attempt_read` for GitHub) until it reports connected. Then
  `drive_files_list` / `source_import_drive` or `source_import_repository`.
- Files on the user's computer: `source_upload_handoff` returns an upload link for the user;
  MCP cannot read local paths.
- Imports run asynchronously: follow them with `source_operation_read` and `sources_list`.

## 5. Preview answers

Save representative customer questions with `test_case_create`, run them with
`test_case_run` (it runs the draft and uses the Workspace's monthly test allowance; do not
rerun automatically after a timeout) and read results with `test_result_read`. `knowledge_gaps_list` shows questions the
Agent could not answer.

## 6. Connect a channel

- Website widget: `widget_enable`, then `widget_installation_read` for the install code. The
  code goes before `</body>` on the user's site. `widget_settings_edit` changes its look and copy.
- LINE: `provider_connect_handoff` with provider `line` returns the setup link where the user
  enters the LINE channel credentials. Then `line_connection_verify`. Group behavior:
  `line_behavior_read` / `line_behavior_edit`.

## 7. Publish

Call `publication_review`, show the user the candidate, and publish only when the user agrees:
`publication_publish` with that `review_id`. Publishing changes what customers receive.
`publication_history` lists versions; `publication_restore` puts an earlier version back into
the draft, which then needs publishing again.

## 8. After launch

- Inbox: `inbox_list`, `inbox_read`, `inbox_reply`, `inbox_drafts`, `inbox_draft_approve`,
  `inbox_takeover`, `inbox_return_to_ai`, `inbox_resolve`.
- Insights: `agent_analytics`, `agent_kpis_read`, `workspace_usage`.

## Rules for every write

- Tools that take `request_key` need a new UUID that you keep before calling. After a lost
  response, repeat the identical request with the same key; it returns the original result
  and does not act twice.
- Errors carry `code` and `next_action`. Read current state before trying again.
- Removal tools (`agent_remove`, `channel_remove`, `provider_disconnect`) each have an
  impact read (`agent_removal_impact`, `channel_removal_impact`,
  `provider_disconnect_impact`); show it to the user and remove only on their explicit request.
- The user can revoke this access in Replai under Workspace settings → MCP & CLI.
