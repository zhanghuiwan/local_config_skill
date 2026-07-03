# 工作流程

## 新建或初始化 Git/GitHub 项目

1. 读取 `config.json`，如果存在 `config.local.json`，用它深度覆盖默认配置。对象递归合并，数组整体替换。
2. 选择 `git.defaultProfile` 指定的 profile。
3. 确认 `userName`、`userEmail`、`githubUsername`、`defaultBranch` 是否完整且不是示例占位值。
4. 检查当前目录：

```bash
git rev-parse --show-toplevel
git status --short --branch
```

5. 如果未初始化 Git，且用户目标是创建项目，可以执行：

```bash
git init
git branch -M <defaultBranch>
```

6. 如果 `userName` 和 `userEmail` 都是真实有效值，写入项目级身份：

```bash
git config --local user.name "<userName>"
git config --local user.email "<userEmail>"
```

7. 如果任一值为空或仍是 `Your Name`、`you@example.com` 等示例占位值，停止并询问用户；禁止把空值或占位值写入 Git 配置。
8. 根据 `files.gitignoreSecrets` 补充 `.gitignore`。
9. 如果需要创建 GitHub remote，但缺少仓库名、组织名、remote 协议或认证方式，先询问用户。

## 配置本地大模型

1. 读取 `localModels.defaultProvider`。
2. 获取 provider 的 `baseUrl`、`apiKeyRequired`、`apiKeyEnv`、`apiKey`、`defaultModel`、`envNames`。
3. 如果 `baseUrl` 或 `defaultModel` 为空或仍是 `your-model-name` 等示例占位值，询问用户。
4. 如果 `apiKeyEnv` 非空，先检查环境变量是否存在。
5. 如果 `apiKeyRequired=true`，且环境变量不存在、`apiKey` 为空，询问用户是否要写入本地 env 文件或改用环境变量。
6. 如果 `apiKeyRequired=false`，且 key 为空，不要阻塞；仅在目标项目或 SDK 强制需要 key 时写入占位值。
7. 确认项目已有配置约定：

```bash
find . -maxdepth 2 \( -name ".env*" -o -name "*config*" -o -name "settings.*" \) -print
```

8. 按 `envNames` 指定的变量名写入 `files.envFile`，并确保它被 `.gitignore` 忽略。
9. 写入 `files.envExampleFile` 时只使用占位值。

## 安装 Python 依赖

1. 判断是否需要 Python：
   - `python.enabled=false`：跳过。
   - `python.enabled=true`：继续。
   - `python.enabled=auto`：只有用户请求 Python、项目存在 `python.projectMarkers` 中的文件、或安装目标明确需要 Python 时继续。
   - 如果不需要 Python，即使电脑没装 Python，也只汇报“本次不需要 Python，已跳过”。
2. 检查项目依赖文件：

```bash
find . -maxdepth 3 \( -name "pyproject.toml" -o -name "requirements*.txt" -o -name "environment*.yml" -o -name "setup.py" \) -print
```

3. 检查环境管理器和 Python。所有探测命令都必须允许工具缺失。类 Unix shell 使用：

```bash
command -v conda || true
command -v mamba || true
command -v micromamba || true
command -v python || true
command -v python3 || true
command -v conda >/dev/null 2>&1 && conda info --json || true
command -v conda >/dev/null 2>&1 && conda env list || true
command -v python >/dev/null 2>&1 && python --version || true
command -v python3 >/dev/null 2>&1 && python3 --version || true
```

Windows shell 使用 `where python`、`where conda`、`where mamba`、`where nvidia-smi` 等等价探测命令。

4. 如果需要 Python 但系统没有 Python、conda、mamba、micromamba 或项目 `.venv`，询问用户是否安装 Python/conda、指定已有路径，或跳过 Python 部分。不要自动安装系统级 Python。
5. 如果 `python.preferredCondaEnv` 非空且存在，使用该环境。
6. 如果当前是 base 且 `python.allowInstallIntoBase` 为 `false`，停止并询问用户选择环境。
7. 如果允许创建环境，使用配置中的环境名和 Python 版本；缺少名称时询问用户：

```bash
conda create -n "<env-name>" python=<python-version>
```

8. 安装时使用目标环境的 Python，优先使用 `conda run` 或目标 Python 绝对路径：

```bash
conda run -n "<env-name>" python -m pip install -r requirements.txt
conda run -n "<env-name>" python -c "import sys; print(sys.executable); print(sys.prefix)"
```

## 安装 CUDA/GPU 相关依赖

1. 先读取 `python.cuda` 配置。
2. 如果当前任务不需要 Python/GPU 依赖，跳过。
3. 如果 `requireProbe` 为 `true`，运行条件探测命令：

```bash
command -v nvidia-smi >/dev/null 2>&1 && nvidia-smi || true
command -v nvcc >/dev/null 2>&1 && nvcc --version || true
```

4. 确认当前 Python 环境不是 base，除非用户在配置和对话中都明确允许。
5. 识别目标框架：`pytorch`、`tensorflow`、`jax`、`vllm`、`bitsandbytes`、`flash-attn` 等。
6. 如果 `installCommandOverride` 非空，展示命令并确认目标环境后执行。
7. 如果没有 override，查官方安装说明，选择与驱动、CUDA、Python 版本兼容的命令。
8. 安装后运行最小验证，例如：

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

根据实际框架替换验证命令。
