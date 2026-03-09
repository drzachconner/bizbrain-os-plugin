---
name: Google Workspace
description: >
  Hybrid Google Workspace integration — gwcli for lightweight daily email, calendar,
  and drive operations (zero MCP context cost), plus full MCP subprocess for Docs, Sheets,
  Slides, Forms, and complex multi-service workflows.
  Triggers on: "check email", "send email", "search email", "calendar", "schedule meeting",
  "google drive", "search drive", "google docs", "google sheets".
version: 1.0.0
---

# Google Workspace

You manage Google Workspace operations using a hybrid two-tool architecture: gwcli for fast daily tasks and google_workspace_mcp for heavy document work.

## Brain Location

Check: BIZBRAIN_PATH env → ~/bizbrain-os/

## TWO TOOLS — Hybrid Architecture

### 1. gwcli (CLI Tool — Lightweight Default)

- **Purpose:** Daily email, calendar, drive operations with zero MCP context cost
- **Install:** `npm install -g google-workspace-cli` (or `npm link` from `~/Repos/google-workspace-cli`)
- **Requires:** Google Cloud OAuth credentials (Desktop Application type)
- **Profile management:** `gwcli profiles add <name> --client <path-to-client-secret.json>`

**Gmail Commands:**

```
gwcli gmail list [--unread] [--limit N] [--format json]
gwcli gmail search "query" [--format json]
gwcli gmail read <message-id> [--format json]
gwcli gmail send --to user@example.com --subject "..." --body "..."
gwcli gmail reply <message-id> --body "..."
gwcli gmail archive <message-id>
```

**Calendar Commands:**

```
gwcli calendar list [--format json]
gwcli calendar events [--days N] [--limit N] [--format json]
gwcli calendar create "Title" --start "2025-01-15 10:00" --end "2025-01-15 11:00"
gwcli calendar update <event-id> --title "..." --start "..."
gwcli calendar delete <event-id>
```

**Drive Commands:**

```
gwcli drive list [--folder <id>] [--format json]
gwcli drive search "query" [--format json]
gwcli drive download <file-id> [--output path]
gwcli drive export <doc-id> --format pdf|xlsx|pptx
```

### 2. google_workspace_mcp (Full MCP — On-Demand for Heavy Work)

- **Purpose:** Docs, Sheets, Slides, Forms, Chat, Tasks, Contacts, Apps Script
- **When to use:** Creating/editing Google Docs, manipulating spreadsheets, building presentations, managing forms
- **Invoke via subprocess:** `/mcp run google-workspace "create a doc titled..."` (no permanent context cost)
- **Install:** `uvx workspace-mcp --tool-tier core`
- **12 services:** Gmail, Drive, Calendar, Docs, Sheets, Slides, Forms, Chat, Tasks, Contacts, Apps Script, Search

## Brain Integration Points

### Gmail → Intake System

When processing emails:
1. Use `gwcli gmail search "is:unread"` to find new messages
2. For actionable emails, extract content and route to `_intake-dump/`
3. Link sender to brain entities if matched
4. Extract action items → todo system

### Calendar → Entity History

When viewing meetings:
1. Use `gwcli calendar events --days 7` for upcoming schedule
2. Match attendee names/emails against entity contacts
3. Log meetings to entity `_context/history.md`

### Drive → Document Search

When searching for client/project documents:
1. Use `gwcli drive search "client name"` to find files
2. Use `gwcli drive export <id> --format pdf` for downloads
3. Route to `_intake-dump/` for processing if needed

## OAuth Setup (Shared by Both Tools)

Both tools use the same Google Cloud OAuth credentials:

1. Go to https://console.cloud.google.com
2. Create a new project (or select existing)
3. Enable APIs: Gmail API, Google Calendar API, Google Drive API
4. For Docs/Sheets/Slides (MCP only): also enable Docs API, Sheets API, Slides API
5. Create OAuth 2.0 credentials → Desktop Application type
6. Download client secret JSON
7. For gwcli: `gwcli profiles add personal --client ~/Downloads/client_secret_*.json`
8. For MCP: Set env vars GOOGLE_OAUTH_CLIENT_ID and GOOGLE_OAUTH_CLIENT_SECRET

## Decision Matrix

| Task | Tool | Why |
|------|------|-----|
| Check/search email | gwcli | Zero context cost, fast |
| Send email | gwcli | Simple CLI command |
| View calendar | gwcli | Zero context cost |
| Schedule meeting | gwcli | Simple CLI command |
| Search Drive | gwcli | Zero context cost |
| Download files | gwcli | Simple CLI command |
| Create/edit Google Doc | MCP subprocess | Needs Docs API tools |
| Manipulate spreadsheet | MCP subprocess | Needs Sheets API tools |
| Build presentation | MCP subprocess | Needs Slides API tools |
| Manage forms | MCP subprocess | Needs Forms API tools |
| Complex multi-service workflow | MCP subprocess | Needs multiple API tools |

## Quick Commands

| Request | Action |
|---------|--------|
| "Check my email" | `gwcli gmail list --unread --format json` |
| "Search for emails from Tim" | `gwcli gmail search "from:tim" --format json` |
| "What's on my calendar today?" | `gwcli calendar events --days 1 --format json` |
| "Schedule a meeting with..." | `gwcli calendar create "..." --start "..." --end "..."` |
| "Find the proposal doc" | `gwcli drive search "proposal" --format json` |
| "Create a Google Doc" | `/mcp run google-workspace "create doc titled ..."` |
| "Update the spreadsheet" | `/mcp run google-workspace "update sheet ..."` |

## Important Notes

- **Always use `--format json`** with gwcli so Claude can parse structured output.
- **Pre-approve gwcli in settings:** Add `"allow": ["Bash(gwcli:*)"]` to avoid confirmation prompts on every command.
