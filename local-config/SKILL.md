---
name: local-config
description: "基于用户可编辑配置文件的本地项目配置 skill。用于新建或初始化 Git/GitHub 项目时设置项目级 Git 用户名和邮箱，配置本地大模型或 OpenAI-compatible API 的 base URL、API key、默认模型，准备 .env/.gitignore 等本地配置文件，以及按需安装 Python 依赖、创建或选择 conda/mamba/venv 环境；尤其适用于 CUDA、GPU、PyTorch、TensorFlow、JAX、vLLM、bitsandbytes、flash-attn、llama.cpp GPU 构建等场景。触发词包括：新建项目、初始化项目、配置 git、配置 GitHub、设置用户名邮箱、本地大模型、LLM API、base_url、apikey、Python 环境、conda 环境、CUDA 安装、GPU 依赖、安装依赖、requirements、pyproject、venv。"
---

# Local Config

## 核心原则

本 skill 的配置来源是同目录下的 `config.json` 和可选的 `config.local.json`。执行任何项目初始化、Git 身份设置、本地大模型连接、Python 环境或 CUDA 依赖安装前，必须先读取并理解配置。

如果配置项为空、缺失、明显不适合当前项目，必须询问用户补全或确认。不要猜测用户名、邮箱、API key、模型地址、conda 环境名、CUDA 版本、Python 安装方式或安装命令。

## 必须先做

1. 定位本 skill 目录，读取 `config.json`。
2. 如果存在 `config.local.json`，读取它并深度覆盖 `config.json`。对象递归合并，数组整体替换。不要把 `config.local.example.json` 当作有效配置读取。
3. 如果需要字段说明，读取 `references/config-schema.md`。
4. 如果要执行完整项目初始化，读取 `references/workflows.md`。
5. 检查目标项目的现状：Git 状态、已有配置文件、项目技术栈、是否需要 Python、Python 环境、conda/mamba/venv、CUDA 工具链。
6. 只对目标项目做项目级修改，除非用户明确要求全局配置。

## 配置读取规则

- `config.json` 中的空字符串、空数组、`null`、`"ask"` 都表示需要询问用户或从项目现状推断后再确认。
- 示例占位值也必须视为未配置，包括但不限于 `your-*`、`replace-me`、`TODO`、`Your Name`、`you@example.com`、`your-github-username`、`your-model-name`。
- `config.json` 只放可提交的默认配置，不要放真实密钥。真实 API key、私有 base URL、私有邮箱等本地敏感覆盖值应放在 `config.local.json` 或环境变量中。
- 如果用户要求保存本机私有配置，创建或修改 `config.local.json`，不要把私有值写入 `config.json`。
- 当同时存在 `apiKey` 与 `apiKeyEnv` 时，优先使用环境变量；只有环境变量不存在且 `apiKey` 非空时才使用配置文件里的 key。
- 写入任何配置前必须校验值真实有效；禁止把空字符串、`null`、`"ask"` 或示例占位值写入 Git 配置、env 文件、remote URL 或安装命令。
- 任何输出给用户的密钥、token、cookie、私有 endpoint 都必须脱敏。
- 如果配置和项目已有约定冲突，先说明冲突，再询问用户选择。
- 如果 `behavior.dryRunByDefault` 为 `true`，只能输出计划、待修改文件和待执行命令，不得实际写文件、改 Git 配置或安装依赖，除非用户在当前对话中明确确认执行。

## Git/GitHub 项目配置

用于新建项目、初始化仓库、准备 GitHub 项目或修复 Git 身份配置。

执行流程：

1. 读取 `git.defaultProfile`，再到 `git.profiles` 中找到对应 profile。
2. 检查目标项目是否已经是 Git 仓库。
3. 检查项目级 Git 身份：

```bash
git config --local user.name
git config --local user.email
```

4. 如果项目级身份缺失，且配置中的 `userName` 和 `userEmail` 都是真实有效值，写入项目级配置：

```bash
git config --local user.name "<userName>"
git config --local user.email "<userEmail>"
```

5. 如果 `userName` 或 `userEmail` 为空或仍是示例占位值，停止并询问用户；禁止写入空值或占位 Git 身份。
6. 不要默认执行 `git config --global`。
7. 如果 `githubUsername`、remote URL、仓库名等配置为空或仍是示例占位值，创建 GitHub 远程仓库或设置 remote 前必须询问用户。
8. 修改 `.gitignore` 时保留已有内容，只补充配置中 `files.gitignoreSecrets` 指定的本地密钥文件规则。

## 本地大模型配置

用于配置 Ollama、LM Studio、OpenAI-compatible 服务、自建网关或其他本地 LLM 服务。

执行流程：

1. 读取 `localModels.defaultProvider`，再读取 `localModels.providers[defaultProvider]`。
2. 确认 `baseUrl`、`apiKeyRequired`、`apiKeyEnv`、`apiKey`、`defaultModel`、`envNames` 是否完整。
3. 如果项目已有 `.env.example`、`.env.local`、`.env`、配置模块或框架约定，优先沿用项目约定。
4. 如果项目没有约定，使用 `files.envFile` 作为本地环境文件，使用 `files.envExampleFile` 作为可提交示例文件。
5. 写入真实 key 时只能写入本地文件；示例文件只能写占位值。
6. 不要在日志、最终回复、commit message 或文档中暴露真实 key。
7. 写入前必须确认 `baseUrl` 和 `defaultModel` 非空；如果 `apiKeyRequired` 为 `true`，且 key 既不在环境变量中，也不在本地配置中，先询问用户。
8. 如果 `apiKeyRequired` 为 `false` 且 key 为空，不要为了 key 阻塞；只有目标项目或 SDK 强制要求 key 时才写入占位值。

写入本地环境文件的变量名来自 provider 的 `envNames` 配置，例如：

```dotenv
<baseUrlEnv>=<baseUrl>
<apiKeyEnv>=<apiKey or env value>
<modelEnv>=<defaultModel>
```

## Python、Conda 与 CUDA

用于安装 Python 依赖、创建环境、选择现有环境或安装 GPU/CUDA 相关包。只有当用户请求、项目文件或配置表明需要 Python 时才进入此流程。

执行流程：

1. 先判断是否需要 Python：
   - 如果 `python.enabled` 为 `false`，跳过 Python 流程。
   - 如果 `python.enabled` 为 `auto`，只有用户请求 Python、项目存在 Python 标记文件，或安装目标明确依赖 Python 时才继续。
   - 如果项目不需要 Python，即使电脑没装 Python，也不要安装或报错；只汇报“本次不需要 Python，已跳过”。
2. 如果需要 Python，再探测系统现状。探测命令必须允许工具缺失，不要因为未安装 Python、conda 或 CUDA 直接失败。类 Unix shell 使用：

```bash
command -v conda || true
command -v mamba || true
command -v micromamba || true
command -v python || true
command -v python3 || true
command -v nvidia-smi || true
command -v nvcc || true
command -v conda >/dev/null 2>&1 && conda info --json || true
command -v conda >/dev/null 2>&1 && conda env list || true
command -v python >/dev/null 2>&1 && python --version || true
command -v python3 >/dev/null 2>&1 && python3 --version || true
command -v nvidia-smi >/dev/null 2>&1 && nvidia-smi || true
command -v nvcc >/dev/null 2>&1 && nvcc --version || true
```

Windows shell 使用 `where python`、`where conda`、`where nvidia-smi` 等等价探测命令。

3. 如果需要 Python 但系统没有任何 Python、conda、mamba、micromamba 或项目 `.venv`，停止并询问用户希望安装 Python、安装 conda/mamba，还是跳过 Python 部分；不要自动安装系统级 Python。
4. 如果存在 conda/mamba/micromamba，必须先列出现有环境。
5. 如果当前环境是 `base`，且 `python.allowInstallIntoBase` 不是 `true`，不得在 base 中安装项目依赖。
6. 优先使用 `python.preferredCondaEnv` 指定的环境；如果为空或不存在，询问用户选择已有环境或创建新环境。
7. 创建新环境时使用 `python.createEnv.name` 与 `python.createEnv.pythonVersion`。如果名称为空，先询问用户。
8. 安装包时必须绑定目标解释器。优先使用 `conda run` 或目标 Python 绝对路径，不要依赖当前 shell 里的裸 `python`：

```bash
conda run -n "<env-name>" python -m pip install ...
python -c "import sys; print(sys.executable); print(sys.prefix)"
```

9. 涉及 CUDA/GPU 包时，先确认 GPU、驱动、CUDA runtime、Python 版本与框架版本兼容。不要凭空选择 CUDA 版本。
10. 如果配置中 `python.cuda.installCommandOverride` 非空，优先使用该命令，但执行前仍要确认目标环境不是 base，且 CUDA 探测结果合理。
11. 如果兼容性不确定，查阅官方安装文档后再执行。

## 输出要求

完成后只汇报：

- 读取了哪个配置 profile/provider/env
- 修改了哪些项目文件或项目级 Git 配置
- 哪些配置仍为空，需要用户补全
- Python 是否需要；如果需要，Python/conda/CUDA 是否已确认，实际使用了哪个环境；如果不需要或未安装但可跳过，说明已跳过
- 是否有真实密钥被写入本地文件，必须只说“已写入/未写入”，不要展示值
