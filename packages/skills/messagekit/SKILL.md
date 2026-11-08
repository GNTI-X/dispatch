---
name: dispatch
description: Use Dispatch to send Telegram messages from agents through the Dispatch MCP tool or CLI fallback. Use when a user asks to send a Telegram message, use Dispatch, interact with the Dispatch toolset, verify Dispatch manually, or choose between Dispatch MCP and CLI workflows.
---

# Dispatch

Use Dispatch to send Telegram messages from an agent. Prefer MCP when available, and use the CLI as the fallback.

## Interfaces

- Prefer the MCP `telegram` tool when the current client has Dispatch configured.
- Use the CLI when MCP is unavailable, when debugging locally, or when a shell-friendly workflow is required.
- Treat `@gnti/dispatch-core` as an implementation detail, not as a direct user interface.

## MCP Usage

Use the `telegram` MCP tool to send a Telegram message.

Required tool input:

- `chatId`: Telegram chat ID.
- `message`: Message text to send.

Do not include Telegram bot tokens in MCP tool arguments. Local MCP clients should provide `TELEGRAM_BOT_TOKEN` through the Dispatch MCP server environment.

For remote MCP clients, configure the MCP server URL with the Telegram bot token in the path:

```text
https://your-dispatch-host.example.com/<telegram-bot-token>/mcp
```

Remote Dispatch servers may require Clerk OAuth. If the MCP endpoint returns `401 Unauthorized` with `WWW-Authenticate`, let the MCP client complete its OAuth login flow instead of sending users to a browser redirect manually.

Treat the full remote MCP URL like a secret because it contains the embedded Telegram bot token.

## CLI Fallback

Use `--json` whenever the result needs to be parsed by an agent or script.

First-time CLI setup:

```bash
dispatch init --telegram-bot-token "<bot-token>"
```

Send a message:

```bash
dispatch telegram "<chat-id>" "Hello from Dispatch" --json
```

## Safety

- Prefer MCP tools for agent workflows when they are available.
- Never ask users to put Telegram bot tokens directly in MCP tool arguments.
- Treat remote MCP URLs containing Telegram bot tokens as secrets.
- Do not reimplement Telegram API calls in the agent; use MCP or CLI.
- Report command failures directly instead of guessing at missing state.
