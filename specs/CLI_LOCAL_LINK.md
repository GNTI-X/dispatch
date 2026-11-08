# CLI Local Link Spec

## Goal

Allow the `@gnti/dispatch` package to be invoked from anywhere on the machine as `dispatch` before the package is published.

This is the first distribution step for the CLI: local global invocation through Bun's package linking.

## Current Package Requirement

`packages/cli/package.json` must expose the CLI binary:

```json
{
  "bin": {
    "dispatch": "./src/index.ts"
  }
}
```

`packages/cli/src/index.ts` must keep the Bun shebang:

```ts
#!/usr/bin/env bun
```

This lets Bun create a global `dispatch` command that runs the TypeScript entrypoint directly.

## Link Workflow

From the CLI package directory, register the package globally:

```bash
bun link
```

From any other directory on the machine, verify the linked binary:

```bash
dispatch --help
```

Configure the CLI:

```bash
dispatch init --telegram-bot-token "<bot-token>"
```

Send a Telegram message:

```bash
dispatch telegram "<chat-id>" "Hello from Dispatch"
```

Send a Telegram message with JSON output:

```bash
dispatch telegram "<chat-id>" "Hello from Dispatch" --json
```

## Acceptance Criteria

- Running `bun link` inside `packages/cli` creates a global `dispatch` command.
- Running `dispatch --help` from outside the repository prints the Dispatch CLI help output.
- Running `dispatch init --telegram-bot-token "<bot-token>"` writes local CLI config to `~/.config/dispatch/config.json`.
- Running `dispatch telegram "<chat-id>" "Hello from Dispatch"` uses the shared `@gnti/dispatch-core` Telegram operation.
- The CLI package does not duplicate Telegram business logic for linking or distribution.

## Unlink Workflow

To remove the local global command, run this from `packages/cli`:

```bash
bun unlink
```
