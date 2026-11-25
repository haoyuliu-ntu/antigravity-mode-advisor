# Antigravity Mode Advisor (`mode-advisor`)

`mode-advisor` 是一个用于 **Google Antigravity** 的全局 Skill，通过只读轻量侦察与四维评估，自动为当前开发或研究任务推荐最优的执行模式、内置模型、思考强度以及开箱即用的启动指令，同时避免 `Orchestrator + /swarm` 叠用造成的 Prompt Cache 失效与配额浪费。

## 核心功能

- **只读诊断、不越权修改**：仅评估任务维度并输出决策方案与启动指令，不改动项目源码、不私自拉起 Swarm。
- **7 选 1 模式决策树**：综合需求明确度、任务可并行性、交互形态与单点推理深度，自动在以下模式中择优：
  - `/grill-me`（需求模糊、关键取舍未定）
  - 单模型 `Gemini 4 Hydrogen`（1–2 个文件的简单明确改动）
  - 单模型 `Gemini 4 Neon`（高频短轮次、边探边改，最大化吃满 Prompt Cache）
  - `Orchestrator`（1 次清晰规划 + 10+ 步长尾编码与构建验证）
  - 单模型 `Gemini 4 Carbon` / `/boost`（高度耦合、不可并行的极限难点）
  - 切回单模型 + `/swarm`（3+ 互不冲突子模块并行推进，自动附带按角色固定单模型模板）
  - `/teamwork`（跨小时/跨天的大型多里程碑项目）
- **缓存与配额防踩坑**：明确拦截 `Orchestrator` 套娃 `/swarm` 的缓存杀手，并在 Gemini 配额紧张/耗尽时提供基于独立配额池的 `Claude 5.5` 应急切换路径。

## 目录结构

采用 `skills/<skill-name>/...` 结构，便于后续直接向仓库内扩展更多 Antigravity Skills：

```text
.
├── .gitignore
├── README.md
└── skills/
    └── mode-advisor/
        ├── SKILL.md
        └── references/
            └── model-and-cache-rules.md
```

## 快速安装

将 `skills/mode-advisor` 复制到本机 Antigravity 全局 Skills 目录（`~/.gemini/config/skills/`）：

### Windows (PowerShell)

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.gemini\config\skills" | Out-Null
Copy-Item -Recurse -Force ".\skills\mode-advisor" "$env:USERPROFILE\.gemini\config\skills\mode-advisor"
```

### macOS / Linux

```bash
mkdir -p ~/.gemini/config/skills
cp -R ./skills/mode-advisor ~/.gemini/config/skills/mode-advisor
```

新开一个 Antigravity 对话即可自动发现并加载该 Skill。

## 使用方法

在任意新对话中直接触发：

- **斜杠命令**：`/mode-advisor <你的任务描述>`
- **自然语言**：`"帮我选模式：<你的任务描述>"`、`"这个任务用什么模型好，要不要开 swarm？"`

> **省额度技巧**：可以先切换到轻量的 **`Gemini 4 Hydrogen`**（或使用 `/btw`）运行 `/mode-advisor` 获取分诊结果与启动指令，再手动切换到推荐的模式正式开始任务。
