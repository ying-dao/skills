---
name: shadowbot-cli
description: Operate ShadowBot (影刀) from Windows PowerShell via shadowbot.shell-cli.exe. Use when user requests login, mode/page/theme switch, app/group/collaborator actions, task run/stop/logs, trigger management, message center, extension install, or Studio open/edit/save/sync. Execute real CLI commands only; never invent commands.
---

# ShadowBot CLI Skill

## Runtime Contract
- Call format in PowerShell:
  - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe <subcommands>"`

## Hard Rules
1. Execute real commands only. Never invent command/flag/ID.
2. Never fabricate command names, subcommands, flags, or parameter values.
3. For Fast Paths, execute directly without help-chain exploration.
4. If command/flag is not found at runtime, report the exact error and stop; do not guess alternatives.
5. Keep user-provided values unchanged (username/password/name/path).
6. Use quoted Windows paths.
7. Keep command count minimal. Verify only at dependency boundaries.
8. For non-fast-path scenarios, use `-h` discovery first, then execute.
9. If connection/session check fails (not logged in, cannot query current account, cannot reach local ShadowBot endpoint), run login recovery before business commands.
10. Before prompting for a password, list remembered accounts with `shadowbot.shell-cli.exe auth account list` and reuse their usernames (password optional); if user/pass absent, take the first remembered account and login without `--password`.


## Minimal Workflow
1. Parse intent and required entities (account, app, trigger, id, page, theme).
2. Preflight session check:
   - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth current"`
   - If current account is unavailable, run login recovery.
   - If login recovery is needed, query remembered usernames: `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth account list"`
3. If intent matches Fast Paths, execute directly with mapped commands.
4. If intent is not in Fast Paths, use `-h` discovery to locate the right command/flags.
5. Return concise evidence: what ran + key output/result.

## Login Recovery
When any of these signals appear, recover by login first:
- `auth current` fails or returns not logged in.
- Error indicates ShadowBot local endpoint/session is unavailable.
- Error indicates account context is missing.

Recovery actions:
0. Query remembered usernames:
   - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth account list"`
    - If user did not supply credentials, take the first entry from the list and treat it as the current username (no `--password`).
    - If a remembered username matches a requested identity, favor it (login without `--password`).
1. If username is known:
   - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth login --username <u> --password <p>"`
   - If password is not provided, run:
     - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth login --username <u>"`
2. After login, verify:
   - `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth current"`
3. Then continue original business command chain.
4. If username is unknown, ask user for username (and optional password), then retry.

Login retry prompt template (ask only what is missing):
- Missing username:
  - `请提供影刀登录用户名。若你希望我直接尝试登录，也可以同时提供密码。`
- Username known, password missing and remembered-password login failed:
  - `当前账号 <u> 的免密登录失败。请提供该账号密码，我将重试登录并继续执行命令。`
- Username/password provided but login still failed:
  - `登录失败。请确认账号或密码是否正确，或确认影刀客户端是否已可用后让我重试。`

## Fast Paths

### Login
1. `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth login --username <u> --password <p>"`

### Open ShadowBot + Execute CLI
Use for requests like: "帮我打开影刀并执行CLI程序，并告诉我输出结果".
1. `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe auth current"`
2. If not logged in/session missing, run Login Recovery.
3. Execute requested CLI command(s) directly.
4. Return command output summary and key evidence lines.

### Run App
1. `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe console app"`
2. From list output, locate target app and extract `<app_id>`.
3. `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe console task run --app-id <app_id>"`
4. Capture returned `<task_id>` from run result for follow-up status/logs/stop actions.

### Stop App
1. Use known `<task_id>` from previous `console task run` result when available.
2. If `<task_id>` is unknown, run `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe console task history --page 1 --page-size 20"` and locate target task.
3. `powershell -NoProfile -ExecutionPolicy Bypass -Command "shadowbot.shell-cli.exe console task stop --task-id <task_id>"`

## Other Common Scenarios (Discover via -h)
- App management discovery:
  - `shadowbot.shell-cli.exe console app -h`
  - Case: list apps, get app detail, delete/recycle app, publish app.
- Task observation discovery:
  - `shadowbot.shell-cli.exe console task -h`
  - `shadowbot.shell-cli.exe console task history -h`
  - `shadowbot.shell-cli.exe console task status -h`
  - `shadowbot.shell-cli.exe console task logs -h`
  - `shadowbot.shell-cli.exe console task stop -h`
  - Case: query task status by task-id, fetch task logs by task-id, stop task by task-id.
- Trigger operations discovery:
  - `shadowbot.shell-cli.exe console trigger -h`
  - Case: add trigger, update trigger, enable/disable trigger, delete trigger.
- Message center discovery:
  - `shadowbot.shell-cli.exe console message -h`
  - Case: list unread, read one message, mark read/unread, read-all.
- UI operations discovery:
  - `shadowbot.shell-cli.exe console page -h`
  - `shadowbot.shell-cli.exe console theme -h`
  - `shadowbot.shell-cli.exe console link -h`
  - Case: switch page, switch theme, open community/web-console links.
- Extension operations discovery:
  - `shadowbot.shell-cli.exe console extension -h`
  - Case: install extension, query extension status/list.
- Studio operations discovery:
  - `shadowbot.shell-cli.exe studio -h`
  - Case: open/create/edit/save/sync studio project.
- System operations discovery:
  - `shadowbot.shell-cli.exe mode -h`
  - `shadowbot.shell-cli.exe system -h`
  - Case: switch assistant/console mode, query system state.

## Failure Handling
- If executable is not found in PATH, report missing executable and stop.
- If command returns CLI JSON error, surface that error directly.
- If scenario is ambiguous, ask for the missing entity (example: app name/id) or run `-h` discovery.
- If business command fails due to missing login/session, perform one login recovery attempt, then retry once.

## Output Style
- Default: concise result with executed commands and key evidence.
- If caller requests JSON, return:

```json
{
  "ok": true,
  "summary": "one-line result",
  "commands": ["exact executed commands"],
  "assertions": ["key checks and evidence"]
}
```
