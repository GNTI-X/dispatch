# Dispatch Rename Plan

## Goal

Rename the project from the generic `starter` identity to `dispatch` so the tutorial is immediately understandable.

Dispatch should communicate the product idea clearly:

```text
One messaging capability exposed through CLI, MCP, and Skill interfaces.
```

The first provider remains Telegram, but the project name should leave room for future providers such as Discord, Slack, email, SMS, or other messaging platforms.

## Naming Decision

Use `Dispatch` as the human-facing product name and `dispatch` as the command/package name.

Recommended positioning:

```text
Dispatch lets agents and developers send messages through CLI, MCP, and Skill interfaces.
```

Avoid continuing to describe the repository as only a starter. It can still be tutorial-friendly, but the name should describe the tool being built.

## Public Name Map

Rename these public identifiers:

```text
Starter                    -> Dispatch
starter                    -> dispatch
starter-workspace          -> dispatch-workspace
@starter/core              -> @gnti/dispatch-core
@starter/cli               -> @gnti/dispatch
@starter/local-mcp         -> @gnti/dispatch-mcp
@starter/remote-mcp        -> dispatch-remote-mcp
@starter/skill             -> dispatch-skill
starter telegram ...       -> dispatch telegram ...
starter init ...           -> dispatch init ...
starter-mcp                -> dispatch-mcp
starter-local              -> dispatch-local
starter-remote             -> dispatch-remote
~/.config/starter          -> ~/.config/dispatch
```

Keep these names unchanged:

```text
packages/core
packages/cli
packages/local-mcp
apps/remote-mcp
packages/skills/dispatch
sendTelegramMessage
telegram MCP tool
telegram CLI command
TELEGRAM_BOT_TOKEN
```

## Package Changes

Update root `package.json`:

```json
{
  "name": "dispatch-workspace"
}
```

Update package names:

```json
// packages/core/package.json
{
  "name": "@gnti/dispatch-core"
}

// packages/cli/package.json
{
  "name": "@gnti/dispatch",
  "bin": {
    "dispatch": "./src/index.ts"
  }
}

// packages/local-mcp/package.json
{
  "name": "@gnti/dispatch-mcp",
  "bin": {
    "dispatch-mcp": "./src/index.ts"
  }
}

// apps/remote-mcp/package.json
{
  "name": "dispatch-remote-mcp",
  "private": true
}

// packages/skills/dispatch/package.json
{
  "name": "dispatch-skill",
  "private": true
}
```

Update all workspace dependencies from `@starter/core` to `@gnti/dispatch-core`.

After package changes, run:

```bash
bun install
```

This should regenerate `bun.lock` with the new package scope.

## Source Code Changes

Update imports:

```ts
import { sendTelegramMessage, telegramMessageInputSchema } from "@gnti/dispatch-core";
```

Update MCP server names:

```text
starter-local  -> dispatch-local
starter-remote -> dispatch-remote
```

Update CLI program name and config paths in `packages/cli`:

```text
program name: dispatch
config dir:   ~/.config/dispatch
config file:  ~/.config/dispatch/config.json
```

Do not rename the Telegram operation yet. The canonical operation should remain:

```text
core function: sendTelegramMessage
CLI command:   dispatch telegram <chatId> <message>
MCP tool:      telegram
Skill usage:   telegram
```

## Documentation Changes

Update README title to make the project obvious:

```md
# Dispatch: Send Messages Through MCP, CLI, and Skills
```

Replace generic starter language with product-specific language:

```text
Dispatch is a tutorial project for building one messaging capability and exposing it through three interfaces: a CLI, a local MCP server, a deployable remote MCP server, and Skill instructions.
```

Update examples:

```bash
bun run dev:cli telegram "<chat-id>" "Hello from Dispatch"
dispatch init --telegram-bot-token "<bot-token>"
dispatch telegram "<chat-id>" "Hello from Dispatch" --json
```

Update MCP client examples:

```json
{
  "mcpServers": {
    "dispatch": {
      "command": "dispatch-mcp",
      "args": [],
      "environment": {
        "TELEGRAM_BOT_TOKEN": "<bot-token>"
      }
    }
  }
}
```

Update remote MCP examples:

```json
{
  "mcp": {
    "dispatch": {
      "type": "remote",
      "url": "https://your-host.example.com/mcp",
      "enabled": true,
      "headers": {
        "Authorization": "Bearer {env:TELEGRAM_BOT_TOKEN}"
      }
    }
  }
}
```

Update Skill documentation:

```text
Starter Skill -> Dispatch Skill
Starter toolset -> Dispatch toolset
@starter/core -> @gnti/dispatch-core
starter CLI -> dispatch CLI
```

Update `AGENTS.md` so future agents preserve the new project identity and do not reintroduce `starter` naming.

## Repository Directory

The current local folder is:

```text
/Users/antonioerdeljac/Work/mcp-starter
```

Renaming the local repository directory is optional and should be handled separately from code changes because it affects local shell paths, editor state, MCP client config, and ignored local config files.

If renamed later, prefer:

```text
/Users/antonioerdeljac/Work/dispatch
```

## Local Config Migration

The CLI config path changes from:

```text
~/.config/starter/config.json
```

to:

```text
~/.config/dispatch/config.json
```

Do not add backward compatibility unless there is a concrete need. This repository is still a tutorial project, so a clean rename is easier to teach.

Update docs to tell users to rerun:

```bash
dispatch init --telegram-bot-token "<bot-token>"
```

## Verification

After implementation, run:

```bash
bun install
bun run typecheck
bun run dev:cli init --telegram-bot-token "<bot-token>"
bun run dev:cli telegram "<chat-id>" "Hello from Dispatch"
bun run dev:cli telegram "<chat-id>" "Hello from Dispatch" --json
TELEGRAM_BOT_TOKEN="<bot-token>" bun run dev:local-mcp
bun run dev:remote-mcp
```

Manual checks:

```text
No @starter imports remain.
No starter CLI examples remain.
No Starter product references remain except historical specs if intentionally preserved.
The telegram operation still works through core, CLI, local MCP, remote MCP, and Skill guidance.
```

Use targeted searches:

```bash
rg "@starter|starter|Starter|mcp-starter|starter-mcp|starter-local|starter-remote"
rg "@dispatch|dispatch|Dispatch"
```

## Commit Strategy

Make the rename as one focused commit:

```text
refactor: rename project to dispatch
```

Do not combine the rename with feature changes, provider additions, or behavior changes.
