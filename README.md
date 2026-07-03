# Local Config Skill

English | [中文](#中文)

`local-config` is a lightweight agent skill for applying local project setup defaults safely. It helps coding agents configure per-project Git identity, local model connection settings, local env files, and optional Python/Conda/CUDA environment workflows without guessing private values or writing secrets into committed files.

## Features

- Configure project-local `git user.name` and `git user.email`.
- Prepare local LLM or OpenAI-compatible API settings.
- Keep private values in `config.local.json` or environment variables.
- Add safe `.env` / `.gitignore` patterns for local secrets.
- Handle Python, Conda, and CUDA setup only when the project actually needs it.
- Avoid installing dependencies into Conda `base` by default.

## Installation

```bash
npx skills add zhanghuiwan/local_config_skill
```

## Configuration

After installation, edit the installed skill configuration:

- `config.json`: shared defaults that are safe to commit.
- `config.local.json`: private local overrides such as real API keys, private endpoints, Git identity, and preferred Conda environments.
- `config.local.example.json`: a template for creating `config.local.json`.

Do not put real API keys or other secrets in `config.json`.

## Repository Layout

The installable skill lives in `local-config/`:

```text
local-config/
├── SKILL.md
├── config.json
├── config.local.example.json
└── references/
```

## Usage

Ask your agent to use the `local-config` skill when creating or configuring a project, for example:

```text
Use local-config to initialize this project with my Git identity and local model settings.
```

The skill will read the config files first, ask when required values are missing, and avoid global or destructive changes unless explicitly confirmed.

---

## 中文

`local-config` 是一个轻量级 agent skill，用于安全地复用本机项目配置。它可以帮助代码代理为项目设置 Git 用户名和邮箱、本地大模型连接、本地环境变量文件，以及按需处理 Python、Conda、CUDA 环境流程，同时避免猜测私密配置或把密钥写入可提交文件。

## 功能

- 配置项目级 `git user.name` 和 `git user.email`。
- 准备本地大模型或 OpenAI-compatible API 配置。
- 将私密值保存在 `config.local.json` 或环境变量中。
- 为本地密钥文件补充安全的 `.env` / `.gitignore` 规则。
- 仅在项目确实需要时处理 Python、Conda、CUDA 环境。
- 默认避免把依赖安装到 Conda `base` 环境。

## 安装

```bash
npx skills add zhanghuiwan/local_config_skill
```

## 配置

安装后，修改已安装 skill 中的配置文件：

- `config.json`：可提交的共享默认配置。
- `config.local.json`：本机私有覆盖配置，例如真实 API key、私有 endpoint、Git 身份、偏好的 Conda 环境。
- `config.local.example.json`：创建 `config.local.json` 时可参考的模板。

不要把真实 API key 或其他密钥写入 `config.json`。

## 仓库结构

可安装的 skill 位于 `local-config/`：

```text
local-config/
├── SKILL.md
├── config.json
├── config.local.example.json
└── references/
```

## 使用

在创建或配置项目时，让你的 agent 使用 `local-config` skill，例如：

```text
Use local-config to initialize this project with my Git identity and local model settings.
```

该 skill 会先读取配置文件，在必要字段缺失时询问用户，并避免未经确认的全局配置或破坏性修改。
