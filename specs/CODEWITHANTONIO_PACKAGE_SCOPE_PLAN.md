# CodeWithAntonio Package Scope Plan

## Goal

Move Dispatch away from the assumed `@dispatch/*` npm scope and into a package model that reflects the real distribution plan.

The npm organization is `@gnti`, but not every workspace package should use that scope. Use the scope only for packages intended to be installable from npm.

## Final Naming Decision

Published npm packages:

```text
packages/cli       -> @gnti/dispatch
packages/core      -> @gnti/dispatch-core
packages/local-mcp -> @gnti/dispatch-mcp
```

Private workspace-only packages/apps:

```text
apps/remote-mcp    -> dispatch-remote-mcp
packages/skills/dispatch -> dispatch-skill
```

This gives the workspace a clear convention:

```text
@gnti/* = installable npm package
dispatch-*       = private workspace-only package or app
```

Do not use generic package names such as `@gnti/core` or `@gnti/cli`. The package basename should keep the Dispatch product identity.

## Package Roles

`@gnti/dispatch` is the primary user-facing CLI package.

Users install it from npm and run the `dispatch` binary:

```bash
npm install -g @gnti/dispatch
dispatch --help
```

`@gnti/dispatch-core` is the shared implementation package.

It is required by the CLI and MCP packages, and it can also be used directly as an SDK for programmatic usage:

```bash
npm install @gnti/dispatch-core
```

```ts
import { sendTelegramMessage } from "@gnti/dispatch-core";
```

`@gnti/dispatch-mcp` is the local MCP stdio server package.

Users install it when they want an MCP client to launch Dispatch locally:

```bash
npm install -g @gnti/dispatch-mcp
```

MCP clients still run the `dispatch-mcp` binary:

```json
{
  "mcpServers": {
    "dispatch": {
      "command": "dispatch-mcp"
    }
  }
}
```

`dispatch-remote-mcp` is a private Hono server app for remote MCP deployment.

It is deployed as a service, not installed from npm.

`dispatch-skill` is a private workspace package for Skill documentation/instructions.

It is not currently published. The Skill may later be bundled into `@gnti/dispatch`, but leave it as a separate private workspace package for now.

## Keep Unchanged

These product, runtime, and tutorial-facing names should not change:

```text
Product name:       Dispatch
CLI binary:         dispatch
MCP binary:         dispatch-mcp
Config path:        ~/.config/dispatch/config.json
CLI command:        dispatch telegram <chatId> <message>
MCP tool name:      telegram
Core operation:     sendTelegramMessage
Repository folder:  dispatch
Root package name:  dispatch-workspace
Package folders:    packages/core, packages/cli, packages/local-mcp, packages/skills/dispatch
Remote app folder:  apps/remote-mcp
```

The rename should affect package identities, dependency specifiers, imports, and documentation package references. It should not change the Dispatch brand or command names.

## Package Manifest Changes

Update `packages/cli/package.json`:

```json
{
  "name": "@gnti/dispatch",
  "bin": {
    "dispatch": "./src/index.ts"
  },
  "dependencies": {
    "@gnti/dispatch-core": "workspace:*"
  }
}
```

Update `packages/core/package.json`:

```json
{
  "name": "@gnti/dispatch-core"
}
```

Update `packages/local-mcp/package.json`:

```json
{
  "name": "@gnti/dispatch-mcp",
  "bin": {
    "dispatch-mcp": "./src/index.ts"
  },
  "dependencies": {
    "@gnti/dispatch-core": "workspace:*"
  }
}
```

Update `apps/remote-mcp/package.json`:

```json
{
  "name": "dispatch-remote-mcp",
  "private": true,
  "dependencies": {
    "@gnti/dispatch-core": "workspace:*"
  }
}
```

Update `packages/skills/dispatch/package.json`:

```json
{
  "name": "dispatch-skill",
  "private": true
}
```

## Source Code Changes

Update all imports from the old core package name:

```ts
import { sendTelegramMessage, telegramMessageInputSchema } from "@dispatch/core";
```

to the new package name:

```ts
import { sendTelegramMessage, telegramMessageInputSchema } from "@gnti/dispatch-core";
```

Expected source files:

```text
packages/cli/src/index.ts
packages/local-mcp/src/index.ts
apps/remote-mcp/src/index.ts
```

Update source-facing descriptions that reference the core package name, such as the CLI description in `packages/cli/src/index.ts`.

Do not rename the CLI program from `dispatch`.

## Lockfile Changes

After manifest changes, run:

```bash
bun install
```

This should update `bun.lock` so workspace package entries use:

```text
@gnti/dispatch
@gnti/dispatch-core
@gnti/dispatch-mcp
dispatch-remote-mcp
dispatch-skill
```

Do not edit the lockfile by hand unless `bun install` cannot regenerate it correctly.

## Documentation Changes

Update package identity references in:

```text
README.md
AGENTS.md
packages/skills/dispatch/SKILL.md
specs/CLI_LOCAL_LINK.md
specs/REMOTE_MCP_PLAN.md
specs/REMOTE_MCP_HONO_REFACTOR.md
specs/MESSAGEKIT_RENAME_PLAN.md
```

Replace package references:

```text
@dispatch/cli
@dispatch/core
@dispatch/local-mcp
@dispatch/remote-mcp
@dispatch/skill
```

with:

```text
@gnti/dispatch
@gnti/dispatch-core
@gnti/dispatch-mcp
dispatch-remote-mcp
dispatch-skill
```

Keep product and runtime examples unchanged when they refer to the CLI, MCP server name, config path, or Dispatch brand.

Examples that should stay unchanged:

```bash
dispatch --help
dispatch init --telegram-bot-token "<bot-token>"
dispatch telegram "<chat-id>" "Hello from Dispatch"
dispatch telegram "<chat-id>" "Hello from Dispatch" --json
```

```json
{
  "mcpServers": {
    "dispatch": {
      "command": "dispatch-mcp"
    }
  }
}
```

## Verification Command Updates

Update package filter examples from:

```bash
bun --filter @dispatch/cli dev init --telegram-bot-token "<bot-token>"
bun --filter @dispatch/cli dev telegram "<chat-id>" "Hello from Dispatch"
bun --filter @dispatch/cli dev telegram "<chat-id>" "Hello from Dispatch" --json
TELEGRAM_BOT_TOKEN="<bot-token>" bun --filter @dispatch/local-mcp dev
bun --filter @dispatch/remote-mcp dev
```

to:

```bash
bun --filter @gnti/dispatch dev init --telegram-bot-token "<bot-token>"
bun --filter @gnti/dispatch dev telegram "<chat-id>" "Hello from Dispatch"
bun --filter @gnti/dispatch dev telegram "<chat-id>" "Hello from Dispatch" --json
TELEGRAM_BOT_TOKEN="<bot-token>" bun --filter @gnti/dispatch-mcp dev
bun --filter dispatch-remote-mcp dev
```

## Implementation Order

1. Update package names and workspace dependency specifiers in package manifests.
2. Run `bun install` to regenerate `bun.lock`.
3. Update TypeScript import specifiers and package-name descriptions.
4. Update README, Skill instructions, AGENTS.md, and existing specs.
5. Search for remaining `@dispatch/` references and classify each occurrence as either intentional historical context or a missed rename.
6. Run formatting, linting, and typechecking.

## Verification

Run:

```bash
bun install
bun run format
bun run lint
bun run typecheck
```

Then smoke-check package filters:

```bash
bun --filter @gnti/dispatch dev --help
TELEGRAM_BOT_TOKEN="<bot-token>" bun --filter @gnti/dispatch-mcp dev
bun --filter dispatch-remote-mcp dev
```

The Telegram send commands still require a real bot token and chat ID for full manual verification.

## Success Criteria

- Published npm packages use the `@gnti/*` scope.
- Private workspace-only packages use unscoped `dispatch-*` names.
- No package is named under `@dispatch/*`.
- CLI and MCP workspace dependencies use `@gnti/dispatch-core`.
- All TypeScript imports resolve through `@gnti/dispatch-core`.
- The CLI package is named `@gnti/dispatch`.
- The CLI command remains `dispatch`.
- The local MCP package is named `@gnti/dispatch-mcp`.
- The local MCP binary remains `dispatch-mcp`.
- The remote MCP app is named `dispatch-remote-mcp` and remains private.
- The Skill package is named `dispatch-skill` and remains private.
- The human-facing product name remains `Dispatch`.
- `bun run typecheck` succeeds after the rename.
