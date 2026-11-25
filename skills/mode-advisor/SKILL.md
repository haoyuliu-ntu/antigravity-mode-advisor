---
name: mode-advisor
description: >-
  Diagnose a user's task and recommend the optimal Antigravity execution mode,
  built-in model, thinking level, and ready-to-paste launch prompt while
  protecting prompt cache hit rate and quota. Use when the user asks which mode
  or model to use, whether to pick Orchestrator, /swarm, /boost, /teamwork,
  /grill-me, or standalone Gemini 4 Neon/Carbon/Hydrogen/Claude, says "帮我选模式",
  "用什么模型好", "要不要开 swarm", or invokes /mode-advisor.
---

# Mode Advisor (任务模式与模型诊断器)

Read-only triage skill that recommends the best Antigravity execution mode, model, and thinking level for a task, and generates a copy-pasteable launch prompt.

## Guardrails

1. **只读分诊、不越权执行**：
   - 仅负责分析任务并输出推荐方案与启动指令；**严禁修改任何项目代码**，**严禁私自调用 `swarm` 工具**。
   - 诚实遵守平台边界：Agent 无法替用户热切换当前对话的 UI 模型下拉框，也无法替用户输入斜杠命令，必须在输出中明确告知用户手动切换步骤。
2. **轻量侦察（节省额度）**：
   - 若任务涉及当前工作区代码，最多执行 1–2 次只读快速查看（例如列出顶层目录或统计相关模块/包数量），切忌为了分诊而通读大量源码。
   - 若任务不依赖本地代码或描述已足够清晰，直接根据用户描述评估即可。
3. **语言一致**：始终使用与用户相同的语言回复（用户用中文时使用简体中文）。

---

## Step 1: 四维快速评估

从以下 4 个维度快速判断任务特征：

1. **需求明确度**：目标、输入输出与关键取舍是否已经敲定？
2. **任务宽度与可并行性**：能否干净地拆分成 **3 个及以上**、文件边界互不重叠的独立子模块/子包？
3. **交互形态与长尾比例**：是“高频短轮次探索、边聊边改”，还是“1 次清晰规划 + 10 步以上长尾编码与构建跑测”？
4. **单点推理深度**：是常规日常修改，还是高度耦合的底层架构、并发竞态、复杂算法或疑难根因排查？

---

## Step 2: 7 选 1 决策树（按优先级顺序匹配）

按从上到下的顺序匹配第一条符合的规则：

1. **需求模糊、关键设计决策未定**
   - **推荐**：先用 **`/grill-me`**（搭配单模型 `Gemini 4 Neon (中)` 或 `Gemini 4 Hydrogen (中)`）。
   - **理由**：先低成本通过多轮设计树访谈对齐边界，避免直接开大模型或重型模式写歪返工。
2. **范围小（1–2 个文件）且规则明确**（改文案、加单测、升级依赖、写小脚本、简单修复）
   - **推荐**：单模型 **`Gemini 4 Hydrogen (中)`**（复杂一点可调至 `高`）。
   - **理由**：响应最快、配额最省，无需规划阶段或编排开销。
3. **面广、可干净拆成 3+ 个互不冲突的独立子任务**（多包修单测、前后端+文档+独立验收、多路并行审计）
   - **推荐**：**先退出 Orchestrator，切回单一模型（如 `Gemini 4 Neon (中)`），再运行 `/swarm`**。
   - **关键要求**：在生成的 `/swarm` 指令中，必须明确写出每个 `@worker` 的文件所有权、`@verifier` 验收职责，并**按角色指定固定单一模型**（例如：所有 worker 使用 `Gemini 4 Hydrogen`，`@verifier` 使用 `Gemini 4 Neon`）。
4. **跨小时/跨天、多里程碑仓库级迁移或从零搭建大型系统**
   - **推荐**：**`/teamwork`**。
   - **理由**：需要前置范围访谈、独立项目工作区与里程碑审计门禁。
5. **高度耦合、不可并行但极难的单点攻关**（隐蔽并发竞态、复杂算法调优、核心架构设计）
   - **推荐**：单模型 **`Gemini 4 Carbon (高)`** 或 **`/boost`**（仅付费计划）。
   - **理由**：每一步都需要极限推理与严密推演，不宜把执行步骤下放给轻量执行器。
6. **单线明确目标 + 规划少、执行长**（1 次完整规划 + 10 步以上多文件编辑、编译与测试验证）
   - **推荐**：**`Orchestrator`**
     - 默认 Planner：`Gemini 4 Neon (中/高)`（仅当架构风险极高或 Neon 曾失败时，才建议在模型菜单悬停 `Orchestrator` 临时覆盖为 `Gemini 4 Carbon (高)`）。
     - Executor：沿用全局设置中的快模型（`Gemini 4 Hydrogen` 或 `Claude Sonnet 5.5`）。
   - **理由**：规划与执行交接次数极少（1–2 次），把长尾落地交给更快、更省的 Executor 性价比最高。
7. **高频来回沟通、探索式 Debug、边聊边改**
   - **推荐**：单模型 **`Gemini 4 Neon (中/高)`**（或 `Extra` 用于极限疑难推导）。
   - **理由**：全程固定同一模型可吃满 **Prompt Cache** 前缀缓存；若开启 Orchestrator，Planner 与 Executor 频繁来回交接会反复打穿缓存，反而更慢更费额度。

---

## Step 3: 缓存与配额防踩坑检查

输出建议前，快速核对以下三条规则（详细模型参数与缓存机制见 [references/model-and-cache-rules.md](./references/model-and-cache-rules.md)，仅在用户追问底层原理或需核对细节时再读取）：

* **严禁 `Orchestrator` × `/swarm` 套娃**：只要推荐了 `/swarm`，必须在“切换步骤”中用醒目提示提醒用户**先在模型选择器退出 Orchestrator，切回单一模型**，避免 `N 个 Agent × 内部频繁换模型` 导致缓存命中率暴跌。
* **配额耗尽应急路由**：如果用户提到 Gemini 配额紧张或已用尽，提醒 **Gemini 家族共享同一个配额池**，而 **Claude 5.5 (Sonnet / Opus)** 拥有独立配额池（限非试用付费 Google AI Pro、Google AI Ultra 与 Enterprise）；此时应建议直接切到纯 Claude 单模型，或确保 Orchestrator 的 Planner 与 Executor 全设为 Claude。
* **历史退场模型提示**：不要主动推荐即将于 2026-11-30 下线的旧模型（`Gemini 4 Argon`、`Gemini 3.8 Flash`、`Claude 4.6`、`GPT-OSS-120b`）。

---

## Step 4: 输出格式（精炼“处方单”）

严格使用以下结构回复，保持简洁、可直接操作：

```markdown
### 🎯 推荐模式：<模式名称 + 模型与思考强度>
*备选方案：<一句话说明什么情况下改用备选>*

### 🧭 判定依据
- **任务特征**：<对应四维评估的核心结论>
- **效率与缓存**：<为什么该方案在质量、速度与 Prompt Cache / 配额上最优>

### ⚙️ 切换步骤
1. <在输入框下方模型选择器中的具体点击操作（如有避坑提醒一并写明）>
2. <复制下方启动指令并发送>

### 📋 可直接复制的启动指令
```text
<量身定制的 Prompt 或 /swarm /grill-me /boost 命令，包含清晰边界与验证方式>
```
```
