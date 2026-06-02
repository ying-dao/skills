# ShadowBot Shell CLI Overview (English)

## What the CLI Is

`shadowbot.shell-cli.exe` is the command-line tool for ShadowBot.  
It wraps common operations as standard commands and returns structured JSON output for both manual and automated usage.

## What the CLI Supports

- Account and login: login, current account check
- Apps and tasks: app listing, app run, task query, task stop
- System and mode: health check, state check, mode switch
- Other console capabilities: triggers, messages, extensions, Studio, and more

## How to Use the CLI

### Option 1: Run from command line

Use PowerShell:

```powershell
shadowbot.shell-cli.exe -h
```

Common examples:

Login:

```powershell
shadowbot.shell-cli.exe auth login --username alice --password 123456
```

Check current account:

```powershell
shadowbot.shell-cli.exe auth current
```

Run an app (list apps first, then run by app id):

```powershell
shadowbot.shell-cli.exe console app
shadowbot.shell-cli.exe console task run --app-id <app-id>
```

### Option 2: Install SKILL for agent-driven usage

When you want an agent to automate ShadowBot CLI operations, install and enable the `shadowbot-cli` SKILL in your agent tool.  
During installation, provide the actual SKILL file path directly to the LLM.

Natural-language install prompts:

- `帮我安装 shadowbot-cli SKILL，文件路径是 D:\...\SKILL.md`
- `Please install and enable shadowbot-cli skill from D:\...\SKILL.md`
- `Install shadowbot-cli skill using D:\...\SKILL.md`

