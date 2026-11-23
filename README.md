# Claude Code / Codex 国内配置指南

在国内使用 Claude Code 和 Codex（终端、桌面版、VS Code 插件）时，用第三方 API 服务接入的配置方法和常见报错排查。

本文的配置方式适用于任何兼容 Anthropic Messages API 或 OpenAI Responses API 的服务。示例中的地址以 [PingCode](https://pingcode.ai/docs) 为例，换成其他服务时，替换地址和密钥即可。

## 目录

- [Claude Code 终端版](#claude-code-终端版)
- [Codex CLI 终端版](#codex-cli-终端版)
- [用 CC Switch 管理多个供应商](#用-cc-switch-管理多个供应商)
- [VS Code 插件](#vs-code-插件)
- [常见报错排查](#常见报错排查)
- [恢复原配置](#恢复原配置)

## Claude Code 终端版

### 1. 安装

先安装 [Node.js LTS](https://nodejs.org/)，确认 `node --version` 能输出版本号，然后执行：

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

能看到版本号就说明安装成功。Windows 请使用 PowerShell，不要用 CMD。

### 2. 配置服务地址和密钥

Claude Code 通过两个环境变量切换 API 服务：

| 变量 | 作用 |
|---|---|
| `ANTHROPIC_BASE_URL` | API 服务地址，不填时默认是 `api.anthropic.com` |
| `ANTHROPIC_AUTH_TOKEN` | 第三方服务的密钥 |

推荐写进 `~/.claude/settings.json`（Windows 为 `%USERPROFILE%\.claude\settings.json`），对终端、桌面版和 VS Code 插件都生效：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.pingcode.ai",
    "ANTHROPIC_AUTH_TOKEN": "你的密钥"
  }
}
```

文件里已有其他字段时，只合并 `env` 部分，不要整个覆盖。

也可以写进 Shell 启动文件（macOS 默认 `~/.zshrc`，Linux 一般是 `~/.bashrc`）：

```bash
export ANTHROPIC_BASE_URL="https://api.pingcode.ai"
export ANTHROPIC_AUTH_TOKEN="你的密钥"
```

两种方式只选一种。混用时，排查问题要两处都检查。

### 3. 确认接入

重新打开终端，进入项目目录运行 `claude`，在会话里输入 `/status`：

- 看不到 Base URL 那一行：环境变量没有生效。
- 登录方式仍显示 claude.ai 账号：地址生效了，但密钥没有生效。

发一条简短消息，收到回复后，到服务商的使用日志里核对这次调用。

完整步骤（含 Windows / macOS / Linux 一键脚本）：[Claude（终端）接入教程](https://pingcode.ai/docs/claude-cli)

## Codex CLI 终端版

### 1. 安装

```bash
npm install -g @openai/codex
codex --version
```

npm 下载慢时，可以加 `--registry=https://npmreg.proxy.ustclug.org`。

### 2. 配置 `config.toml`

Codex 用 `~/.codex/config.toml` 定义自定义供应商。`model_provider` 指向的段必须存在，否则启动时会报 `Model provider not found`。

```toml
model = "填写服务商提供的模型名"
model_provider = "pingcode"

[model_providers.pingcode]
name = "PingCode"
base_url = "https://api.pingcode.ai/v1"
wire_api = "responses"
requires_openai_auth = true
```

注意 `base_url` 要保留 `/v1`。

### 3. 配置密钥

`requires_openai_auth = true` 时，Codex 从 `~/.codex/auth.json` 读取密钥：

```json
{
  "OPENAI_API_KEY": "你的密钥"
}
```

如果系统里还设置过 `OPENAI_API_KEY`、`OPENAI_BASE_URL` 或 `CODEX_API_KEY` 环境变量，它们可能覆盖文件配置，建议先清掉。

完整步骤：[Codex（终端）接入教程](https://pingcode.ai/docs/codex-cli)。桌面版见 [Codex（桌面版）](https://pingcode.ai/docs/codex-app)。

## 用 CC Switch 管理多个供应商

同时使用多个 API 服务，或者要在官方账号和第三方服务之间切换时，可以用 [CC Switch](https://github.com/farion1231/cc-switch) 图形化管理，不用手动改配置文件。

在 CC Switch 中选择工具，进入「添加供应商 → 自定义配置」：

| 配置项 | Claude | Codex |
|---|---|---|
| 请求地址 | `https://api.pingcode.ai` | `https://api.pingcode.ai/v1`（保留 `/v1`） |
| 上游格式 | Anthropic Messages（原生） | Responses（原生） |
| 模型 | 点「获取模型列表」后做模型映射 | 点「获取模型列表」后选择 |

切换供应商后，要完全退出并重新打开对应工具，再新建会话。

带截图的步骤：[CC Switch 配置教程](https://pingcode.ai/docs/cc-switch)

## VS Code 插件

VS Code 里的 Claude 插件（发布者 Anthropic）和 Codex 插件（发布者 OpenAI）会复用终端版的用户配置：

| 插件 | 读取的配置 |
|---|---|
| Claude | `~/.claude/settings.json` |
| Codex | `~/.codex/config.toml`、`~/.codex/auth.json` |

所以先按上面配好终端版，再装插件即可。远程开发（SSH、WSL、容器）时，要在插件实际运行的那台机器上配置。

改完配置插件没反应，在命令面板执行 `Developer: Reload Window`。

详见：[VS Code 插件接入](https://pingcode.ai/docs/vscode-extensions)

## 常见报错排查

每次只改一项配置，方便判断是哪一步出了问题。

| 现象 | 先检查什么 |
|---|---|
| `command not found` / 找不到 `claude` | 重新打开终端；检查 npm 全局目录是否在 PATH 里 |
| `401` / `403` / `invalid token` | 地址和密钥是否配套；密钥是否完整、启用、未过期 |
| `model not found` | 模型名是否是服务商支持的名字，不要直接套用其他供应商的模型标识；检查 `ANTHROPIC_MODEL` 等旧设置 |
| 连接超时 | 网络和服务地址；不要关闭证书校验 |
| 配置改了不生效 | 是否同时存在 `settings.json` 和 Shell 环境变量两份配置；项目目录下的 `.claude/settings.json` 是否覆盖了用户配置 |
| 终端可用、桌面版不可用 | 两者要分别检查，桌面版不一定读取 Shell 环境变量 |

更多问题：[工具使用问题](https://pingcode.ai/docs/faq-tools) · [有余额为什么仍无法调用](https://pingcode.ai/docs/faq-balance) · [密钥泄露怎么办](https://pingcode.ai/docs/faq-key-exposed)

## 恢复原配置

- 先备份 `~/.claude/settings.json`，再删除 `env` 里的 `ANTHROPIC_BASE_URL`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_API_KEY`，其他设置保留。
- 不要删除整个 `.claude` 目录或 `.claude.json`。
- 切回官方账号前，先清掉同名环境变量，再重新打开 Claude 按官方流程登录。

## 安全提示

- 第三方 API 服务能看到你发送的全部请求内容，包括代码。涉及敏感代码时，请评估服务商是否可信。
- 密钥不要提交到 Git 仓库，也不要发在截图或群聊里。泄露后立刻在服务商后台禁用并重新生成。

## 参考

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [Codex 官方仓库](https://github.com/openai/codex)
- [PingCode 使用手册](https://pingcode.ai/docs)

欢迎提 Issue 或 PR 补充其他工具和报错案例。
