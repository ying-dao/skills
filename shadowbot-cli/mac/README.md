# ShadowBot CLI 简介（中文）

## CLI 是什么

`shadowbot-cli` 是 ShadowBot 的命令行工具。
它把常见操作封装为标准命令，并输出结构化 JSON，适合手动执行和自动化调用。

影刀 App 从 `1.22.2` 版本开始支持 `shadowbot-cli`。

## CLI 支持哪些功能

- 账号与登录：登录、查询当前账号
- 应用与任务：查询应用、运行应用、查询任务、停止任务
- 系统与模式：健康检查、状态查询、模式切换
- 其他控制台能力：触发器、扩展、Studio 等

## 如何使用 CLI

### 方式 1：直接通过命令行调用

在终端中执行：

```bash
shadowbot-cli --help
```

常用示例：

健康检查：

```bash
shadowbot-cli system health
```

登录：

```bash
shadowbot-cli auth login --username alice --password 123456
```

查看当前登录账号：

```bash
shadowbot-cli auth current
```

查询应用：

```bash
shadowbot-cli console app
shadowbot-cli console app --app-type subscribed --search "报表"
```

运行应用（先查 `app-id`，再运行）：

```bash
shadowbot-cli console app
shadowbot-cli console task run --app-id <app-id> --app-type developed
```

### 方式 2：安装 SKILL 让 Agent 调用

当你希望 Agent 自动执行 ShadowBot CLI 操作时，可以在 Agent 工具里安装并启用 `shadowbot-cli` SKILL。
安装时，直接把 SKILL 文件的实际路径告诉 LLM 即可。

自然语言安装示例：

- `帮我安装 shadowbot-cli SKILL，文件路径是 .../SKILL.md`
- `请从 .../SKILL.md 安装并启用 shadowbot-cli skill`
- `Install and enable shadowbot-cli skill from .../SKILL.md`
