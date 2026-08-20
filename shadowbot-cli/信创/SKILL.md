---
name: shadowbot-cli
description: Use shadowbot-cli to operate the local ShadowBot desktop client through its JSON command-line interface. Use when an Agent needs to inspect the client, authenticate, discover applications, run an authorized task, or inspect task results. Discover changing command details from the installed CLI help; never invent commands, flags, IDs, or credentials.
---

# ShadowBot CLI

Use `shadowbot-cli` as the only interface to the local ShadowBot desktop client. Do not call the local REST API directly or access desktop files/databases to perform CLI operations.

## Runtime rules

- Confirm the executable is available with `command -v shadowbot-cli` before the first operation. If it is unavailable, report that it is not on `PATH`; do not guess an installation path.
- Execute only commands supported by the installed binary. Never invent command names, flags, IDs, credentials, or input values.
- Commands return JSON. Read the command result and preserve non-sensitive identifiers needed by a later command.
- Keep user-provided values unchanged. Do not print, persist, or repeat passwords, access tokens, refresh tokens, local API tokens, or discovery-file contents.
- When reporting a failure, preserve the CLI's message and any non-sensitive identifiers useful for diagnosis.

## Discover commands

Use the nearest help page whenever a command or option is unfamiliar:

```bash
shadowbot-cli --help
shadowbot-cli system --help
shadowbot-cli auth --help
shadowbot-cli console --help
shadowbot-cli console app --help
shadowbot-cli console task --help
shadowbot-cli studio --help
```

If the installed CLI does not expose a requested capability, report it as unsupported and stop. Do not borrow commands from another product or from the Windows implementation.

## Minimal workflow

1. Check local availability with `shadowbot-cli system health`.
2. Inspect `shadowbot-cli system state` when the current mode or client state matters.
3. Use `shadowbot-cli auth current` when the operation needs account context.
4. Resolve any missing command details through `--help` before execution.
5. Execute the smallest command chain that fulfills the user's request.
6. Report the operation and concise, non-sensitive evidence.

Do not repeat health or account checks before every command in one uninterrupted workflow. Recheck after a failure indicating that the client, renderer, or account context changed.

## Authentication

Use the CLI login command when the user asks to log in or an authorized operation cannot proceed without authentication. If login is requested without a username, enter `Login and recovery` immediately; do not ask the user to choose among remembered accounts when a usable account is available:

```bash
shadowbot-cli auth login --username USERNAME --password PASSWORD
shadowbot-cli auth login --username USERNAME
```

Discover the current login options with `shadowbot-cli auth login --help`.

### Login and recovery

When a connection or session check fails, including an unavailable health check, or when the user requests login without providing a username, use this flow before business commands:

0. Query remembered usernames:
   - Run `shadowbot-cli auth account list`.
   - If the user did not provide a username, use the first account name that can be used for passwordless login. Do not ask the user to choose an account.
   - If a remembered username matches the requested identity, prefer that username.
1. If a username is known:
   - With a user-provided password, run `shadowbot-cli auth login --username USERNAME --password PASSWORD`.
   - Without a password, run `shadowbot-cli auth login --username USERNAME`.
   - The CLI may start the ShadowBot desktop client while processing `auth login`; allow recovery to complete.
2. Verify the login with `shadowbot-cli auth current`.
3. After verification succeeds, retry the original command once.
4. If no usable username is available, ask the user for a username and optional password, then retry recovery. Do not call `auth login` without a username.

Never recover or display a saved password in the Agent. If the CLI reports that another session must be replaced, stop and ask the user for explicit confirmation. Only after confirmation may you use the session-replacement login option documented by `auth login --help`; never choose it speculatively or automatically.

## Application and task workflow

When the user asks to run an application:

1. Use `shadowbot-cli console app --help` to discover application listing/search options.
2. Resolve the intended application and preserve its returned ID and source/type.
3. Use `shadowbot-cli console app detail --help` when input declarations are needed.
4. Use `shadowbot-cli console task run --help` to construct the run command.
5. Run a task only when the user has clearly requested that execution. If the request is ambiguous, ask before starting it.
6. Preserve the returned task ID and use `shadowbot-cli console task --help` to find status, logs, history, or stop commands.

Do not invent required inputs or substitute an application when multiple results match. Do not automatically retry a task submission after a timeout or unknown result.

If the CLI reports that the command is unsupported in the current mode, surface that message and stop; do not bypass the restriction or retry a different equivalent command.

## Report format

By default, do not paste the raw CLI/REST JSON envelope. Summarize the operation and its key non-sensitive result in concise prose. Include task IDs or other follow-up identifiers when they are needed.

When the caller explicitly requests JSON, return this wrapper and keep `commands` free of passwords, tokens, and other secrets:

```json
{
  "ok": true,
  "summary": "one-line result",
  "commands": ["exact executed commands with secrets redacted"],
  "assertions": ["key checks and evidence"]
}
```

## Failure handling

- If a command is unavailable, use `--help` once more at the nearest level and then report the installed CLI's actual surface.
- If authentication or account context is missing, follow `Login and recovery`; do not guess credentials.
- If the local client is unavailable, follow `Login and recovery` once; report the CLI error and ask for the missing username or password only if recovery cannot start or complete.
- If a task operation fails or times out, preserve its task ID when present and do not assume that it was canceled.

Keep the final report concise: operation performed, result, follow-up ID if any, and remaining user action.
