# mcp-gmail: AI Assistant Gateway to Gmail

An MCP (Model Context Protocol) server that gives AI assistants like Claude full access to Gmail — list, read, search, send, draft, label, and trash emails.

## Quick Start

### Prerequisites

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) package manager
- A Google Cloud project with the Gmail API enabled
- OAuth credentials (`credentials.json`) or a service account key

### Installation

```bash
# From PyPI
uvx mcp-gmail@latest

# From source
git clone https://github.com/MindMade/mcp-gmail.git
cd mcp-gmail
uv run mcp-gmail
```

### Environment Setup

```bash
# OAuth (interactive — first run opens browser)
export GMAIL_CREDENTIALS_PATH="/path/to/credentials.json"
export GMAIL_TOKEN_PATH="/path/to/token.json"

# Or service account (headless)
export GMAIL_SERVICE_ACCOUNT_PATH="/path/to/service_account.json"
```

## Key Features

- **Full Gmail access** — read, search, send, draft, label, trash
- **15 tools** covering all common Gmail operations
- **Flexible auth** — OAuth, service account, base64 env var, or Application Default Credentials
- **Pagination** — all list operations support `page_token` and `max_results`
- **Attachments** — send emails with file attachments
- **Reply threading** — reply to existing threads with proper headers
- **HTML email** — send plain text and/or HTML bodies
- **Stdio and SSE transports** — works with Claude Desktop, Cursor, and remote deployments

## Authentication Methods

Priority order: `GMAIL_CREDENTIALS_CONFIG` → `GMAIL_SERVICE_ACCOUNT_PATH` → `GMAIL_TOKEN_PATH` → `GMAIL_CREDENTIALS_PATH` → Application Default Credentials

### Method A: Base64-Encoded Service Account (containers)

- Set `GMAIL_CREDENTIALS_CONFIG` to the base64-encoded content of your service account JSON
- No files needed — ideal for Docker / CI / serverless
- Requires domain-wide delegation for accessing user mailboxes

### Method B: Service Account File

- Set `GMAIL_SERVICE_ACCOUNT_PATH` to the path of your service account JSON key file
- Default: `service_account.json` in the working directory
- Requires domain-wide delegation for accessing user mailboxes

### Method C: OAuth2 (interactive, personal accounts)

- Set `GMAIL_CREDENTIALS_PATH` to the path of your OAuth `credentials.json`
- Set `GMAIL_TOKEN_PATH` to where the token should be stored (default: `token.json`)
- First run opens a browser for Google sign-in
- Subsequent runs use the cached token with automatic refresh

### Method D: Application Default Credentials

- Uses `GOOGLE_APPLICATION_CREDENTIALS` environment variable
- Falls back to active `gcloud auth application-default login`
- Falls back to GCP metadata service (for Cloud Run, GKE, etc.)

## Available Tools (15 Total)

### Read Operations

- `gmail_list_messages` — List messages with query, labels, pagination (1-500 per page)
- `gmail_get_message` — Get full message by ID (headers, body, attachments)
- `gmail_search_messages` — Search with Gmail query syntax, returns full details (1-100 per page)
- `gmail_list_drafts` — List drafts with pagination and query filter

### Send Operations

- `gmail_send_message` — Send email with to/cc/bcc, HTML body, attachments, reply threading
- `gmail_create_draft` — Create a draft without sending
- `gmail_update_draft` — Update an existing draft (merges provided fields with existing)
- `gmail_delete_draft` — Permanently delete a draft
- `gmail_send_draft` — Send an existing draft

### Label Operations

- `gmail_list_labels` — List all labels (system and user-created)
- `gmail_create_label` — Create a new label (supports nesting with "/")
- `gmail_delete_label` — Delete a user label
- `gmail_modify_message_labels` — Add/remove labels from a message

### Trash Operations

- `gmail_trash_message` — Move a message to trash (auto-deleted after 30 days)
- `gmail_untrash_message` — Restore a message from trash

## Claude Desktop Configuration

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-gmail@latest"],
      "env": {
        "GMAIL_CREDENTIALS_PATH": "/path/to/credentials.json",
        "GMAIL_TOKEN_PATH": "/path/to/token.json"
      }
    }
  }
}
```

### With Service Account

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-gmail@latest"],
      "env": {
        "GMAIL_SERVICE_ACCOUNT_PATH": "/path/to/service_account.json"
      }
    }
  }
}
```

### From Source (development)

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uv",
      "args": ["--directory", "/path/to/mcp-gmail", "run", "mcp-gmail"]
    }
  }
}
```

## Cursor / Windsurf Configuration

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-gmail@latest"],
      "env": {
        "GMAIL_CREDENTIALS_PATH": "/path/to/credentials.json",
        "GMAIL_TOKEN_PATH": "/path/to/token.json"
      }
    }
  }
}
```

## SSE Transport (remote / container)

```bash
uv run mcp-gmail --transport sse
```

Environment variables for SSE mode:

| Variable | Default | Description |
|----------|---------|-------------|
| `HOST` | `0.0.0.0` | Bind address |
| `PORT` | `8000` | Listen port |

## Google Cloud Platform Setup (Detailed)

1. **Create a Google Cloud project** at https://console.cloud.google.com
2. **Enable the Gmail API** — go to APIs & Services > Library, search for "Gmail API", click Enable
3. **Configure the OAuth consent screen** — go to APIs & Services > OAuth consent screen, select External, fill in app name and contact email
4. **Create OAuth credentials** — go to APIs & Services > Credentials > Create Credentials > OAuth 2.0 Client ID, select "Desktop application"
5. **Download credentials** — click the download button on your new credential and save as `credentials.json`
6. **First authentication** — run `uv run mcp-gmail` and complete the browser sign-in. A `token.json` will be saved for future use.

For service accounts: go to Credentials > Create Credentials > Service Account, create a key (JSON), and download it.

## Environment Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `GMAIL_CREDENTIALS_CONFIG` | — | Base64-encoded service account JSON |
| `GMAIL_SERVICE_ACCOUNT_PATH` | `service_account.json` | Path to service account key file |
| `GMAIL_TOKEN_PATH` | `token.json` | Path to OAuth token file |
| `GMAIL_CREDENTIALS_PATH` | `credentials.json` | Path to OAuth client credentials file |
| `HOST` / `FASTMCP_HOST` | `0.0.0.0` | SSE transport bind address |
| `PORT` / `FASTMCP_PORT` | `8000` | SSE transport port |

## Example Prompts for Claude

- "List my 10 most recent unread emails"
- "Search for emails from alice@example.com with attachments"
- "Send an email to bob@example.com with subject 'Meeting Notes' and body 'Here are the notes from today's meeting.'"
- "Create a draft reply to the last email from my manager"
- "Label all emails from newsletter@example.com as 'Newsletters'"
- "Trash all promotional emails from the last week"
- "Show me the full content of message ID abc123"

## License

MIT
