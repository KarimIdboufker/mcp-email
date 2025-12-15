# Copilot / AI Agent Instructions for mcp-email-server

Purpose
- Short: this repository implements an MCP (multi-capability) stdio server that exposes email-related tools (send, fetch, draft) backed by Gmail IMAP/SMTP.

Quick start (developer)
- Activate the project's virtualenv: PowerShell: `myenv\Scripts\Activate.ps1`, Bash: `source myenv/Scripts/activate`.
- Ensure these environment variables are set before running: `EMAIL_USER`, `EMAIL_APP_PASSWORD` (Gmail app password). A .env may be used but is not present by default.
- Run the server locally: `python email_main.py` (the server binds to stdio; it's intended to be driven by an MCP client over stdin/stdout).

Key files
- `email_main.py` — main MCP server implementation and the authoritative place for tool definitions and business logic. See [email/email_main.py](email/email_main.py#L1-L400).
- `README.md` — tiny project readme. See [email/README.md](email/README.md#L1).
- `myenv/` — local virtualenv containing installed packages; there is no `requirements.txt` in the repo, so the venv reflects runtime deps.

Architecture & data flow (what to know)
- The repo implements an MCP stdio server (using `mcp.server.stdio.stdio_server`) that exposes tools via `@server.list_tools()` and `@server.call_tool()`.
- Tools are defined with `types.Tool` objects and JSON `inputSchema` — follow these schemas when constructing tool calls (see `send_email`, `fetch_n_emails`, `draft_reply` in `email_main.py`).
- Runtime flow: an MCP client sends a tool call over stdio → `handle_call_tool` routes by `name` → functions use Gmail IMAP/SMTP → responses are returned as `types.TextContent` (plain JSON/text payloads).

Project-specific conventions
- Use `mcp` decorators: `@server.list_tools()` returns tool metadata; `@server.call_tool()` must accept `(name: str, arguments: dict)` and return a list of `types.TextContent` instances.
- Input validation is expressed via JSON Schema in `inputSchema` (enforce required fields as declared).
- All IO functions are `async` (async/await) — preserve async signatures and avoid blocking calls.
- When returning email lists, `fetch_n_emails` returns JSON-serializable Python structures; the current implementation returns `json.dumps(emails, ensure_ascii=False, indent=2)` in a `TextContent`.

Integration points & external dependencies
- Gmail SMTP: `smtp.gmail.com:587` (starttls) for `send_email`.
- Gmail IMAP: `imap.gmail.com:993` for fetching messages and appending drafts (`[Gmail]/Drafts`).
- Environment variables: `EMAIL_USER`, `EMAIL_APP_PASSWORD` — required at runtime. Avoid committing secrets.
- The code imports `mcp` library and `mcp.server.stdio` — an MCP runtime is required (already installed in `myenv`).

Examples (payloads)
- Call `send_email` (JSON body):
  {
    "to": "recipient@example.com",
    "subject": "Hello",
    "body": "Text body"
  }
- Call `fetch_n_emails` (JSON body):
  { "n": 5 }
- Call `draft_reply` (JSON body):
  {
    "id": "123",
    "from": "sender@example.com",
    "subject": "Original",
    "body": "Original body",
    "reply_body": "Draft reply text"
  }

Notes for AI agents (dos and don'ts)
- Do read `email_main.py` top-to-bottom before making changes — tool definitions, routing, and network calls are co-located.
- Do not hardcode credentials or modify environment variable handling; prefer env vars or a new `.env` (explicitly ask the user before adding one).
- Do preserve JSON Schema shapes when modifying or adding tools; update both `list_tools()` and `handle_call_tool()` routing together.
- Do not expose network credentials or create new remote listeners; this server communicates over stdio and via outbound Gmail servers only.
- If adding new behavior interacting with Gmail, prefer using high-level libraries and handle IMAP multipart parsing similarly to existing code.

If you need more context
- Ask for the preferred runtime workflow (how the MCP client connects), whether a `requirements.txt` or container image should be created, and whether a `.env.example` is desired.

---
This file was generated to help AI agents start making safe, productive changes in this repo. If anything is missing or unclear, tell me what to expand.
