# HarmonyOS Code Agent 评测工具

面向鸿蒙工程的 Code Agent 评测工具，覆盖三类复杂任务：**缺陷修复 / 新特性开发 / 鸿蒙化迁移**。

以**编译后的可执行文件（exe）**交付，**无需安装 Python / Node 或任何依赖环境**，Windows 上解压即用。

---

## 一、交付物

解压后目录结构：

```
harmony-bench/
├── bin/                  # 全部可执行文件（核心）
│   └── harmony_bench.exe # 统一入口，其余 exe 由它内部自动调用
├── configs/              # 内置配置
│   ├── harmonization.yaml   # 迁移流水线配置
│   ├── pipeline.yaml        # 缺陷流水线配置
│   └── dataset.yaml
└── README.md
```

> **注意**：`bin/` 里的所有 exe 是一个整体，`harmony_bench.exe` 会调用同目录下的其他 exe（`harmonization_main.exe`、`stage*.exe`、`run_eval.exe` 等）。**不要单独移动、改名或删除任何一个 exe**，保持整个 `bin/` 目录完整。

---

## 二、前置要求

- Windows 10 / 11（64 位）
- 跑 Agent（`run` / `batch`）需要 LLM API key，见[第六节](#六api-key-配置)
- 生成任务（`generate`）需要你自己准备的工程数据，见[第五节](#五数据说明)

---

## 三、快速验证

进入解压目录，先确认能跑：

```powershell
bin\harmony_bench.exe --help

# dry-run：只打印将要执行的命令，不真正运行（验证环境/参数是否正常）
bin\harmony_bench.exe generate migration `
  --android D:\data\android --harmonyos D:\data\harmony `
  --config configs\harmonization.yaml --dry-run
```

所有子命令都支持 `--dry-run`。

---

## 四、使用流程

统一入口是 `bin\harmony_bench.exe`，下面所有命令都从解压目录根执行（这样 `configs\...` 相对路径才找得到）。

### 1. 生成任务

三类任务分别用 `generate` 的三个子命令。

**迁移任务**（Android → 鸿蒙）：

```powershell
bin\harmony_bench.exe generate migration `
  --android D:\data\pair1_android `
  --harmonyos D:\data\pair1_harmony `
  --config configs\harmonization.yaml `
  --api-mapping D:\data\api_mapping.json `
  --output-dir outputs\migration
```

| 参数 | 说明 |
|------|------|
| `--android` / `--harmonyos` | 构成 pair 的 Android / 鸿蒙项目路径 |
| `--config` | 迁移流水线配置（内置在 `configs\`，固定用这个路径） |
| `--api-mapping` | HomeTran API 映射库 JSON（你自己准备） |
| `--output-dir` | 任务包输出目录 |

**新特性任务**：

```powershell
bin\harmony_bench.exe generate feature `
  --project D:\data\harmony_project `
  --feature-types route_navigation,network_capability `
  --output D:\data\feature_tasks
```

| 参数 | 说明 |
|------|------|
| `--feature-types` | 逗号分隔的特性类型（缺省全类型） |
| `--max-tasks` | 最大任务数 |
| `--difficulty` | 难度过滤 |

**缺陷修复任务**：

```powershell
bin\harmony_bench.exe generate defect `
  --config configs\pipeline.yaml `
  --project D:\data\harmony_project `
  --project-name sample_clock `
  --output D:\data\outputs
```

| 参数 | 说明 |
|------|------|
| `--config` | 缺陷流水线配置（内置在 `configs\`） |
| `--project-name` | 报告用的项目名 |
| `--output` | 输出根目录 |

### 2. 评估（prepare → run → eval）

生成任务后，评估分三步走。以迁移任务为例：

**第 1 步 prepare** —— 准备工作区：

```powershell
bin\harmony_bench.exe prepare `
  --task-dir outputs\migration\task_packages\<task_id> `
  --output-dir task_runs
```

**第 2 步 run** —— 跑 Agent（需要 API key）：

```powershell
bin\harmony_bench.exe run `
  --task-dir task_runs\<task_id> `
  --model deepseek-v4 --seed 42 --prompt migration
```

| 参数 | 说明 |
|------|------|
| `--model` | `gpt5-codex` / `qwen3-coder` / `deepseek-v4` / `claude-sonnet5` |
| `--prompt` | 任务类型：`defect` / `feature` / `migration` |
| `--seed` | 随机种子（多跑几次用 42/43/44） |
| `--step-limit` / `--cost-limit` | 步数 / 成本上限 |

**第 3 步 eval** —— 评估：

```powershell
bin\harmony_bench.exe eval `
  --task-run-dir task_runs\<task_id> --model deepseek-v4 --seed 42
```

产出 `eval_result.json`（resolved、resolution_rate、各检查项、轨迹信息）。

### 3. 批量全流程

一条命令完成 prepare + run + eval：

```powershell
# 迁移任务（seeds 固定 42/43/44）
bin\harmony_bench.exe batch --task-type migration --models deepseek-v4 gpt5-codex

# 缺陷 / 新特性
bin\harmony_bench.exe batch `
  --task-type feature --models deepseek-v4 --seeds 42 43 44 `
  --pilot-tasks D:\data\pilot_tasks.jsonl --workers 4
```

> 批量模式需要一个「任务清单」文件：迁移任务读取 `bin\curated_tasks.jsonl`，缺陷/新特性通过 `--pilot-tasks` 指定。这个清单由你 `generate` 产出后自行整理（每行一个 JSON，含 `task_path` 字段指向任务包目录）。

---

## 五、数据说明

- **工程数据不进工具**：Android/鸿蒙 pair 项目、输出目录、`api_mapping.json` 都是你自己的数据，用参数传入即可。
- **任务清单**：批量模式需要，格式见上一节说明。
- 工具本身只内置了 `configs\`（流水线配置）和编译好的逻辑，不含任何工程数据。

---

## 六、API key 配置

跑 Agent（`run` / `batch`）需要 LLM key，通过环境变量传入（PowerShell）：

```powershell
$env:CHATANYWHERE_API_KEY = "sk-xxx"
```

cmd 下则用：

```cmd
set CHATANYWHERE_API_KEY=sk-xxx
```

设好后再执行 `run` / `batch` 即可。

---

## 七、常见问题

- **`bin\` 里的 exe 能不能只留 `harmony_bench.exe`？** 不能。它是统一入口，内部会按需调用同目录的其他 exe，删掉会导致对应功能失效。
- **报 `Project path not found`？** 说明 `--android` / `--harmonyos` / `--project` 传的路径不存在，检查是否用了正斜杠/相对路径拼错。
- **`run` 一直报错？** 先确认 `CHATANYWHERE_API_KEY` 已设置且有效。
- **迁移任务跑不出结果？** 确认 `--api-mapping` 传了正确的映射 JSON。

---

## 八、交付说明

本工具以编译后的二进制交付，**不包含 Python / Node 源码**，接收方无需配置运行环境即可直接使用。
