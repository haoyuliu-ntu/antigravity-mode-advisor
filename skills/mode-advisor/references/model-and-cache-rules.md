# Model, Thinking Level, and Cache Reference

Quick reference for `mode-advisor` when explaining underlying model trade-offs, quota pools, or prompt caching behavior.

## 1. Core Built-in Models (Active Lineup)

| Model | Positioning & Strengths | Context & Thinking Levels | Availability |
| :--- | :--- | :--- | :--- |
| **Gemini 4 Neon** | Balanced frontier coding & agent loop model. Fast turnaround with strong judgment; default Orchestrator planner & top single-model pick for interactive work. | Up to **512K** tokens; `Low`, `Medium` (default), `High`, `Extra` (`xhigh`) | Paid Pro (or eligible device trial), Ultra, Enterprise |
| **Gemini 4 Carbon** | Heavyweight long-horizon reasoning flagship. Exhaustive hypothesis checking and deep root-cause analysis, at higher latency and token cost. | `Low`, `Medium`, `High` | Paid Pro (or eligible device trial), Ultra, Enterprise |
| **Gemini 4 Hydrogen** | High-speed, quota-friendly everyday coding model & default Orchestrator executor. Ideal for boilerplate, unit tests, small edits, and docs. | `Low`, `Medium` (default), `High`, `Extra` (`xhigh`) | **All plans** (Free, Plus, Pro, Ultra, Enterprise) |
| **Claude Opus 5.5 / Sonnet 5.5 (thinking)** | Anthropic reasoning & coding models integrated into Antigravity. Great for cross-model review or when Gemini quota is exhausted. | `Low`, `Medium`, `High` | Paid **non-trial** Pro, Ultra, Enterprise |

> **Retiring models (avoid recommending)**: `Gemini 4 Argon`, `Gemini 3.8 Flash`, `Claude 4.6 (Sonnet/Opus)`, and `GPT-OSS-120b` are scheduled for removal on **November 30, 2026**.

---

## 2. Thinking Level Guidelines

* **`Low`**：极其明确的机械修改、简单文案或格式调整。
* **`Medium`（默认）**：绝大多数开发任务的性价比甜点。
* **`High`**：复杂重构、跨模块 Debug、架构设计或需要严谨边界检查的任务。
* **`Extra`（仅 Neon / Hydrogen）**：需要极限推理深度的复杂问题。

---

## 3. Prompt Cache & Quota Mechanics

1. **同模型连续调用才吃得满前缀缓存**：
   * 在单模型对话中，不变的系统指令、规则和历史代码上下文可以高比例命中 Prompt Cache。
   * **Orchestrator** 在 `Planner ↔ Executor` 每次交接时都会切换后端模型（尤其是跨厂商的 `Gemini + Claude` 组合），导致接收方无法复用对方的缓存而冷启动重读上下文。因此 Orchestrator 只适合**“1 次规划 + 10 步以上长执行”**，不适合高频来回短对话。
2. **`/swarm` 的 5 分钟缓存 TTL 与套娃陷阱**：
   * `/swarm` 的休眠成员唤醒间隔若超过约 5 分钟，Prompt Cache TTL 过期会导致冷缓存重读。
   * 如果在 **Orchestrator** 模式下开启 `/swarm`，Worker 默认继承 Orchestrator，变成 `N 个 Agent × 内部频繁切换模型`，会剧烈消耗配额。
   * **正确做法**：开 `/swarm` 前先切回单一模型；若需要强弱分工，直接在 `/swarm` 指令里为不同角色指定各自**固定的单一模型**（如 worker 用 `Gemini 4 Hydrogen`，`@verifier` 用 `Gemini 4 Neon`）。
3. **配额池隔离机制**：
   * **Gemini 共享配额池**：`Gemini 4 Neon / Carbon / Argon / Hydrogen` 共用同一个 Gemini 额度池；一个耗尽时，切到另一个 Gemini 模型同样不可用。
   * **Claude 独立配额池**：`Claude Sonnet 5.5 / Opus 5.5` 计入独立的第三方配额，不受 Gemini 配额耗尽影响（前提是账号订阅包含 Claude 5.5）。
   * **刷新周期**：Pro 与 Ultra 基础配额每 **5 小时**刷新一次，并受每周总上限约束；可在 **Settings** 或 CLI `/usage` 查看剩余额度。
