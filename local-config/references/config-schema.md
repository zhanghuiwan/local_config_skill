# config.json 字段说明

`config.json` 是本 skill 的可提交默认配置入口。用户通过 `npx skills add` 安装后，可以直接修改 `config.json` 中的非敏感默认值；真实 API key、私有 endpoint、私有邮箱等本地敏感值推荐放入同目录的 `config.local.json`。

读取规则：

1. 先读取 `config.json`。
2. 如果存在 `config.local.json`，用它深度覆盖 `config.json`。对象递归合并，数组整体替换。
3. `config.local.example.json` 只是示例，不能当作有效配置读取。
4. 写入任何项目配置前必须校验值真实有效；空字符串、`null`、空数组、`"ask"` 和示例占位值都不能写入目标项目。

示例占位值包括但不限于 `your-*`、`replace-me`、`TODO`、`Your Name`、`you@example.com`、`your-github-username`、`your-model-name`。如果用户把 `config.local.example.json` 复制成 `config.local.json` 但没有修改这些值，必须把它们当作未配置并询问用户。

如果用户要求保存本机私有配置，创建或修改 `config.local.json`；不要把真实 API key、私有 endpoint、私有邮箱等值写入可提交的 `config.json`。

## 顶层字段

| 字段 | 含义 |
| --- | --- |
| `version` | 配置文件版本，当前为 `1` |
| `git` | Git/GitHub 身份与仓库初始化配置 |
| `localModels` | 本地大模型或 OpenAI-compatible 服务配置 |
| `files` | 目标项目中本地配置文件和 `.gitignore` 规则 |
| `python` | 按需启用的 Python、conda、venv、CUDA 安装策略 |
| `behavior` | 执行时的安全策略 |

## git

`git.defaultProfile` 指向 `git.profiles` 中的一个 profile。常见用法是用户配置 `default`、`work`、`personal` 等多个身份。

| 字段 | 含义 |
| --- | --- |
| `userName` | 写入 `git config --local user.name` 的值 |
| `userEmail` | 写入 `git config --local user.email` 的值 |
| `githubUsername` | GitHub 用户名，用于创建 remote、仓库命名或提示 |
| `defaultBranch` | 新仓库默认分支名 |
| `commitSigning` | `true`、`false` 或 `"ask"` |
| `signingKey` | GPG/SSH 签名 key，空值表示询问或不设置 |

不要默认写入全局 Git 配置。除非用户明确要求，否则只设置项目级 local 配置。

## localModels

`localModels.defaultProvider` 指向 `localModels.providers` 中的一个 provider。

| 字段 | 含义 |
| --- | --- |
| `baseUrl` | OpenAI-compatible API 地址，例如 `http://127.0.0.1:11434/v1` |
| `apiKeyRequired` | `true`、`false` 或 `"ask"`；Ollama/LM Studio 通常为 `false` |
| `apiKeyEnv` | 优先读取的环境变量名 |
| `apiKey` | 本地覆盖配置中的 API key。`config.json` 中应保持为空；有真实值时必须脱敏处理 |
| `defaultModel` | 默认模型名 |
| `envNames` | 写入目标项目 env 文件时使用的变量名映射 |
| `headers` | 额外 HTTP headers |

如果 `apiKeyEnv` 和 `apiKey` 同时存在，优先读取环境变量。真实 key 不应写入可提交文件。`apiKeyRequired=false` 时，不要因为 key 为空阻塞流程；只有目标项目或 SDK 强制需要 key 时才写占位值。

`envNames` 字段：

| 字段 | 含义 |
| --- | --- |
| `baseUrl` | base URL 写入目标 env 文件时使用的变量名 |
| `apiKey` | API key 写入目标 env 文件时使用的变量名 |
| `model` | 默认模型写入目标 env 文件时使用的变量名 |

## files

| 字段 | 含义 |
| --- | --- |
| `envFile` | 目标项目中写入真实本地配置的文件 |
| `envExampleFile` | 目标项目中可提交的示例环境文件 |
| `gitignoreSecrets` | 需要补充进 `.gitignore` 的本地密钥文件规则 |

更新 `.gitignore` 时只能追加缺失规则，不能覆盖用户已有内容。

## python

| 字段 | 含义 |
| --- | --- |
| `enabled` | `true`、`false` 或 `"auto"`；`auto` 表示仅当项目或用户请求需要 Python 时启用 |
| `projectMarkers` | 用于判断项目是否需要 Python 的文件名列表 |
| `preferredCondaEnv` | 优先使用的 conda/mamba 环境名 |
| `allowInstallIntoBase` | 是否允许装进 conda base，默认必须为 `false` |
| `allowVenvFallback` | 没有 conda 时是否允许创建 `.venv` |
| `packageManagerPreference` | `conda-first`、`mamba-first`、`venv-first` 或 `ask` |
| `createEnv.enabled` | 是否允许创建新环境，`true`、`false` 或 `"ask"` |
| `createEnv.name` | 新环境名称 |
| `createEnv.pythonVersion` | 新环境 Python 版本 |

如果电脑没有 Python，但当前项目不需要 Python，必须跳过 Python 流程，不能要求用户安装。只有项目需要 Python 或用户明确要求 Python 依赖安装时，才询问用户如何安装或选择环境。

涉及 CUDA/GPU 包时，必须先探测系统，不允许在未确认环境的情况下安装。

## python.cuda

| 字段 | 含义 |
| --- | --- |
| `requireProbe` | 是否必须先运行 `nvidia-smi` / `nvcc --version` |
| `preferredCudaVersion` | 用户偏好的 CUDA 版本，空值表示不要猜测 |
| `framework` | 目标框架，例如 `pytorch`、`tensorflow`、`jax`、`vllm` |
| `installCommandOverride` | 用户手写的安装命令，非空时优先但仍需检查环境 |

## behavior

| 字段 | 含义 |
| --- | --- |
| `askBeforeGlobalChanges` | 全局配置变更前是否询问用户 |
| `redactSecretsInOutput` | 输出时是否脱敏密钥 |
| `preserveExistingFiles` | 是否保留已有文件内容，只做最小修改 |
| `dryRunByDefault` | 是否默认只展示计划不执行 |
