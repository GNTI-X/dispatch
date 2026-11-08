# Dispatch CLI

Dispatch CLI is the human-facing command-line interface for the Dispatch tutorial project.

It runs on Node.js and exposes the `dispatch` binary for manual usage, scripting, and debugging. MCP remains the primary agent-facing interface in the tutorial.

## Installation

```bash
npm install -g @gnti/dispatch
```

## Configuration

Configure the Telegram bot token once:

```bash
dispatch init --telegram-bot-token "<bot-token>"
```

The CLI stores local configuration at `~/.config/dispatch/config.json`.

## Send A Telegram Message

```bash
dispatch telegram "<chat-id>" "Hello from Dispatch"
```

For scriptable output, pass `--json`:

```bash
dispatch telegram "<chat-id>" "Hello from Dispatch" --json
```

The CLI uses `@gnti/dispatch-core` internally and does not duplicate Telegram API logic.
