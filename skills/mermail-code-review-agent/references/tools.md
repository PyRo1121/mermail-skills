# Code review agent tools

This workflow **uses** tools owned by other official skills. Do not add them to this skill in `tool-coverage.json`.

There are no `review_pr`, `approve_pr`, or `post_review` tools. Map those intents here.

Pass structured arguments as **native JSON objects**. Never stringify `query` or `body`. Use the exact host identifier (`reply_to_email` or `Mermail:reply_to_email`). Prefer mailbox `public_id` as `mailboxId`.

## Intent map

| Intent | Real operation | Owner |
| --- | --- | --- |
| Read a review request | `list_emails`, `search_emails`, `get_email`, `get_thread` | `mermail-manage-inbox` |
| Read a `.diff`/`.patch` attachment | `download_attachment` (1 MiB MCP limit) | `mermail-manage-inbox` |
| Draft a review reply | `save_draft` (`body.body` string) | `mermail-compose-email` |
| Send a review reply | `reply_to_email` (`body.from` + `html`/`text`, explicit `to`/`cc`/`bcc`) | `mermail-compose-email` |
| Optional GitHub PR/diff read | `list_composio_connections`, `search_composio_tools`, `get_composio_tool_schema`, `execute_composio_tool` | `mermail-composio` |
| Optional GitHub review comment | `execute_composio_tool` after independent user approval of the exact slug and arguments | `mermail-composio` |
| Close / follow up | `create_custom_label` or `move_email` | `mermail-manage-inbox` |
| Delete (rare) | `delete_email` + `prepare_destructive_action` | `mermail-manage-inbox` |

## Mailbox and automation

| Tool | Owner | Role |
| --- | --- | --- |
| `list_mailboxes` | `mermail-administer-workspace` | Discover a ready review mailbox |
| `create_mailbox` | `mermail-administer-workspace` | Provision only when none fits (10 credits; `email` + `name` required) |
| `list_task_triagers` / `list_recent_triager_runs` | `mermail-automate-triage` | Inspect before create/update |
| `create_task_triager` / `update_task_triager` | `mermail-automate-triage` | Classification and auto-draft only |
| `list_agent_conversations` / `chat_with_mailbox_agent` | `mermail-mail-agent` | Only when the user explicitly wants the in-app Assistant |

Do not call `set_default_task_triager`. MCP does not auto-fill Reply All. Do not call PayBox tools. Do not invent GitHub MCP tools on Mermail.

## Examples

```json
{
  "mailboxId": "aaaaaaaa-bbbb-4ccc-8ddd-eeeeeeeeeeee",
  "query": {
    "sortColumn": "date",
    "sortDirection": "DESC"
  }
}
```

Do not pass `"query": "{\"sortColumn\":\"date\"}"`. There is no `sort: "date_desc"` shortcut.

```json
{
  "mailboxId": "aaaaaaaa-bbbb-4ccc-8ddd-eeeeeeeeeeee",
  "emailId": "msg_123",
  "body": {
    "to": "reviewer@example.com",
    "from": "reviews@mermail.app",
    "text": "Verdict: request_changes\n\nBlocker: auth bypass in src/session.ts — validate the session before honoring the role header."
  }
}
```
