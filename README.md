<div align="center">
  <b>mcp-google-gmail</b>

  <p align="center">
    <i>Your AI Assistant's Gateway to Gmail!</i> 📧
  </p>

[![PyPI - Version](https://img.shields.io/pypi/v/mcp-google-gmail)](https://pypi.org/project/mcp-google-gmail/)
[![PyPI Downloads](https://static.pepy.tech/badge/mcp-google-gmail)](https://pepy.tech/projects/mcp-google-gmail)
![GitHub License](https://img.shields.io/github/license/MindMadeLab/mcp-gmail)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/MindMadeLab/mcp-gmail/release.yml)
</div>

---

## 🤔 What is this?

`mcp-google-gmail` is a Python-based MCP server that acts as a bridge between any MCP-compatible client (like Claude Desktop, Cursor, or Windsurf) and the Gmail API. It allows you to list, read, search, send, draft, label, and trash emails — all driven by AI through natural language.

---

## 🚀 Quick Start

Essentially the server runs in one line: `uvx mcp-google-gmail@latest`.

This command will automatically download the latest code and run it. **We recommend always using `@latest`** to ensure you have the newest version with the latest features and bug fixes.

1.  **☁️ Prerequisite: Google Cloud Setup**
    *   You **must** configure Google Cloud Platform credentials and enable the Gmail API first.
    *   ➡️ Jump to the [**Detailed Google Cloud Platform Setup**](#-google-cloud-platform-setup-detailed) guide below.

2.  **🐍 Install `uv`**
    *   `uvx` is part of `uv`, a fast Python package installer. Install it if you haven't:
        ```bash
        # macOS / Linux
        curl -LsSf https://astral.sh/uv/install.sh | sh
        # Windows
        powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
        ```

3.  **🔐 Authenticate**
    *   Run the built-in auth command to set up your credentials:
        ```bash
        # Point to your OAuth credentials file
        GMAIL_CREDENTIALS_PATH="/path/to/credentials.json" uvx mcp-google-gmail@latest auth
        ```
    *   This opens your browser for Google sign-in. After granting permission, a `token.json` is saved automatically.
    *   You only need to do this **once** — subsequent runs use the cached token.

4.  **🏃 Run the Server!**
    ```bash
    uvx mcp-google-gmail@latest
    ```

5.  **🔌 Connect your MCP Client**
    *   Configure your client (e.g., Claude Desktop) to connect to the running server.
    *   ➡️ See [**Usage with Claude Desktop**](#-usage-with-claude-desktop) for config examples.

You're ready! Start issuing commands via your MCP client.

---

## ✨ Key Features

*   **Full Gmail Access:** Read, search, send, draft, label, and trash emails.
*   **15 Tools** covering all common Gmail operations.
*   **Flexible Authentication:** Supports OAuth 2.0, Service Accounts, Base64 injection, and Application Default Credentials.
*   **Pagination:** All list operations support `page_token` and `max_results`.
*   **Attachments:** Send emails with file attachments.
*   **Reply Threading:** Reply to existing threads with proper In-Reply-To headers.
*   **HTML Email:** Send plain text and/or HTML bodies.
*   **Stdio & SSE Transports:** Works with Claude Desktop, Cursor, and remote/container deployments.

---

## 🔐 Authentication

### The `auth` Command

Before using the MCP server, authenticate with Gmail:

```bash
# Using OAuth credentials (interactive — opens browser)
GMAIL_CREDENTIALS_PATH="/path/to/credentials.json" uvx mcp-google-gmail@latest auth

# Specify where to save the token
GMAIL_CREDENTIALS_PATH="/path/to/credentials.json" \
GMAIL_TOKEN_PATH="/path/to/token.json" \
uvx mcp-google-gmail@latest auth
```

On success, you'll see:

```
Credentials path: /path/to/credentials.json
Token path:       token.json

Authenticated as: you@gmail.com
Total messages:   12345
Token saved to:   token.json
```

### Authentication Priority

The server checks for credentials in this order:

1.  `GMAIL_CREDENTIALS_CONFIG` — Base64-encoded service account JSON (env var)
2.  `GMAIL_SERVICE_ACCOUNT_PATH` — Path to service account key file
3.  `GMAIL_TOKEN_PATH` — Path to existing OAuth token
4.  `GMAIL_CREDENTIALS_PATH` — Path to OAuth credentials (interactive browser flow)
5.  **Application Default Credentials** — `GOOGLE_APPLICATION_CREDENTIALS` / `gcloud` / GCP metadata

### Method A: OAuth 2.0 (Personal Accounts) 🧑‍💻

Best for personal use or local development.

1.  Set up OAuth credentials in Google Cloud Console (see [GCP Setup](#-google-cloud-platform-setup-detailed))
2.  Run `mcp-google-gmail auth` to authenticate via browser
3.  Token is cached for future use with automatic refresh

*   `GMAIL_CREDENTIALS_PATH` — Path to OAuth `credentials.json` (default: `credentials.json`)
*   `GMAIL_TOKEN_PATH` — Where to store the token (default: `token.json`)

### Method B: Service Account (Servers/Automation) ✅

Best for headless environments. Requires [domain-wide delegation](https://developers.google.com/identity/protocols/oauth2/service-account#delegatingauthority) for accessing user mailboxes.

*   `GMAIL_SERVICE_ACCOUNT_PATH` — Path to service account JSON key (default: `service_account.json`)

### Method C: Base64-Encoded Credentials (Containers) 🔒

Best for Docker, Kubernetes, CI/CD where file mounting is impractical.

*   `GMAIL_CREDENTIALS_CONFIG` — Base64-encoded content of your service account JSON
*   Generate with: `base64 -w 0 service_account.json`

### Method D: Application Default Credentials (GCP) 🌐

Best for Google Cloud environments (Cloud Run, GKE, Compute Engine).

*   Uses `GOOGLE_APPLICATION_CREDENTIALS` or `gcloud auth application-default login`
*   No additional env vars needed — used as automatic fallback

---

## 🛠️ Available Tools (15 Total)

### Read Operations

*   **`gmail_list_messages`** — List messages with query, labels, pagination (1-500 per page)
    *   `query` (optional): Gmail search query (e.g. `"is:unread"`, `"from:alice@example.com"`)
    *   `label_ids` (optional): Filter by label IDs (e.g. `["INBOX"]`)
    *   `max_results` (optional, default 20): Messages per page (1-500)
    *   `page_token` (optional): Token for next page
    *   `include_spam_trash` (optional, default false): Include spam/trash
    *   _Returns:_ `{messages: [{id, thread_id, snippet, subject, from, date}], next_page_token, result_size_estimate}`

*   **`gmail_get_message`** — Get full message by ID (headers, body, attachments)
    *   `message_id`: The Gmail message ID
    *   _Returns:_ `{id, thread_id, subject, from, to, cc, date, body_text, body_html, labels, attachments}`

*   **`gmail_search_messages`** — Search with Gmail query syntax, returns full details (1-100 per page)
    *   `query`: Gmail search query (e.g. `"has:attachment after:2024/01/01"`)
    *   `max_results` (optional, default 10): Results per page (1-100)
    *   `page_token` (optional): Token for next page
    *   _Returns:_ `{messages: [{full message details}], next_page_token, result_size_estimate}`

*   **`gmail_list_drafts`** — List drafts with pagination and query filter
    *   `max_results` (optional, default 20): Drafts per page (1-500)
    *   `page_token` (optional): Token for next page
    *   `query` (optional): Gmail search query to filter drafts
    *   _Returns:_ `{drafts: [{draft_id, message_id, subject, to, snippet}], next_page_token, result_size_estimate}`

### Send Operations

*   **`gmail_send_message`** — Send email with to/cc/bcc, HTML body, attachments, reply threading
    *   `to`, `subject`, `body` (required)
    *   `cc`, `bcc`, `html_body`, `attachment_paths` (optional)
    *   `reply_to_message_id`, `thread_id` (optional, for threading)
    *   _Returns:_ `{id, thread_id, label_ids}`

*   **`gmail_create_draft`** — Create a draft without sending (same params as send)
    *   _Returns:_ `{draft_id, message_id}`

*   **`gmail_update_draft`** — Update an existing draft (merges provided fields with existing)
    *   `draft_id` (required), all other fields optional
    *   _Returns:_ `{draft_id, message_id}`

*   **`gmail_delete_draft`** — Permanently delete a draft
    *   `draft_id` (required)
    *   _Returns:_ `{success: true}`

*   **`gmail_send_draft`** — Send an existing draft
    *   `draft_id` (required)
    *   _Returns:_ `{message_id, thread_id}`

### Label Operations

*   **`gmail_list_labels`** — List all labels (system and user-created)
    *   _Returns:_ `{labels: [{id, name, type}]}`

*   **`gmail_create_label`** — Create a new label (supports nesting with `/`)
    *   `name`: Label name (e.g. `"Projects/Work"`)
    *   _Returns:_ `{id, name}`

*   **`gmail_delete_label`** — Delete a user label (system labels cannot be deleted)
    *   `label_id`: The label ID
    *   _Returns:_ `{success: true}`

*   **`gmail_modify_message_labels`** — Add/remove labels from a message
    *   `message_id` (required)
    *   `add_label_ids` (optional): Label IDs to add
    *   `remove_label_ids` (optional): Label IDs to remove
    *   _Returns:_ `{id, label_ids}`

### Trash Operations

*   **`gmail_trash_message`** — Move a message to trash (auto-deleted after 30 days)
    *   `message_id`: The message ID
    *   _Returns:_ `{id, label_ids}`

*   **`gmail_untrash_message`** — Restore a message from trash
    *   `message_id`: The message ID
    *   _Returns:_ `{id, label_ids}`

---

## 🔌 Usage with Claude Desktop

Add the server config to your `claude_desktop_config.json`:

<details>
<summary>🔵 Config: uvx + OAuth (Recommended for personal use)</summary>

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-google-gmail@latest"],
      "env": {
        "GMAIL_CREDENTIALS_PATH": "/path/to/credentials.json",
        "GMAIL_TOKEN_PATH": "/path/to/token.json"
      }
    }
  }
}
```

**🍎 macOS Note:** If you get a `spawn uvx ENOENT` error, use the full path:
```json
"command": "/Users/yourusername/.local/bin/uvx"
```
</details>

<details>
<summary>🔵 Config: uvx + Service Account</summary>

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-google-gmail@latest"],
      "env": {
        "GMAIL_SERVICE_ACCOUNT_PATH": "/path/to/service_account.json"
      }
    }
  }
}
```
</details>

<details>
<summary>🟡 Config: Development (from cloned repo)</summary>

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/mcp-gmail", "mcp-google-gmail"]
    }
  }
}
```
</details>

---

## ⚙️ Usage with Cursor / Windsurf

```json
{
  "mcpServers": {
    "gmail": {
      "command": "uvx",
      "args": ["mcp-google-gmail@latest"],
      "env": {
        "GMAIL_CREDENTIALS_PATH": "/path/to/credentials.json",
        "GMAIL_TOKEN_PATH": "/path/to/token.json"
      }
    }
  }
}
```

---

## 🐳 SSE Transport (Remote / Container)

```bash
uv run mcp-google-gmail --transport sse
```

| Variable | Default | Description |
|:---------|:--------|:------------|
| `HOST` / `FASTMCP_HOST` | `0.0.0.0` | Bind address |
| `PORT` / `FASTMCP_PORT` | `8000` | Listen port |

---

## ☁️ Google Cloud Platform Setup (Detailed)

This setup is **required** before running the server.

1.  **Create/Select a GCP Project** — Go to the [Google Cloud Console](https://console.cloud.google.com/)
2.  **Enable the Gmail API** — Navigate to "APIs & Services" → "Library", search for "Gmail API", click Enable
3.  **Configure OAuth Consent Screen** — Go to "APIs & Services" → "OAuth consent screen", select External, fill in app name and contact email, add the scope `https://www.googleapis.com/auth/gmail.modify`
4.  **Create OAuth Credentials** — Go to "APIs & Services" → "Credentials" → "Create Credentials" → "OAuth 2.0 Client ID", select **Desktop application**
5.  **Download Credentials** — Click the download button and save as `credentials.json`
6.  **Authenticate** — Run the auth command:
    ```bash
    GMAIL_CREDENTIALS_PATH="/path/to/credentials.json" uvx mcp-google-gmail@latest auth
    ```
    Complete the browser sign-in. A `token.json` will be saved for future use.

For **Service Accounts**: Go to Credentials → Create Credentials → Service Account, create a key (JSON), download it. Note: Service accounts require [domain-wide delegation](https://developers.google.com/identity/protocols/oauth2/service-account#delegatingauthority) for Gmail access.

---

## 🔧 Environment Variables Reference

| Variable | Default | Description |
|:---------|:--------|:------------|
| `GMAIL_CREDENTIALS_CONFIG` | — | Base64-encoded service account JSON |
| `GMAIL_SERVICE_ACCOUNT_PATH` | `service_account.json` | Path to service account key file |
| `GMAIL_TOKEN_PATH` | `token.json` | Path to OAuth token file |
| `GMAIL_CREDENTIALS_PATH` | `credentials.json` | Path to OAuth client credentials file |
| `HOST` / `FASTMCP_HOST` | `0.0.0.0` | SSE transport bind address |
| `PORT` / `FASTMCP_PORT` | `8000` | SSE transport port |

---

## 💬 Example Prompts for Claude

*   "List my 10 most recent unread emails"
*   "Search for emails from alice@example.com with attachments"
*   "Send an email to bob@example.com with subject 'Meeting Notes' and the body 'Here are the notes from today.'"
*   "Create a draft reply to the last email from my manager"
*   "Label all emails from newsletter@example.com as 'Newsletters'"
*   "Trash all promotional emails from the last week"
*   "Show me the full content of message ID abc123"
*   "What are my unread emails about project deadlines?"

---

## 🤝 Contributing

Contributions are welcome! Please open an issue to discuss bugs or feature requests. Pull requests are appreciated.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Credits

*   Built with [FastMCP](https://github.com/jlowin/fastmcp)
*   Uses [Google API Python Client](https://github.com/googleapis/google-api-python-client)
*   Inspired by [mcp-google-sheets](https://github.com/xing5/mcp-google-sheets)
