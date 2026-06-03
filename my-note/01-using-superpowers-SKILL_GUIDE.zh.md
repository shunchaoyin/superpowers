# `using-superpowers` 技能深度解读

> 原文：[SKILL.md](SKILL.md)  
> 解读时间：2026-06-03  
> 本文档基于对 `using-superpowers` 的逐行分析，梳理其定位、规则体系、反合理化防御机制及关键设计决策。

---

## 一、技能定位：整个框架的"宪法"

`using-superpowers` 是 **Superpowers 框架的自举技能（Bootstrap Skill）**。它的核心职责只有一个：**教会 Agent 如何发现和使用其他技能**。没有它，其他 14 个技能即使存在于磁盘上，也永远不会被触发。

| 属性 | 说明 |
|------|------|
| **触发条件** | `Use when starting any conversation` —— 每个会话开始时自动加载 |
| **深度（Depth）** | 极高：117 行内容约束了所有后续行为的元规则 |
| **刚性（Rigid）** | 是。铁律，不可协商 |
| **文件位置** | `skills/using-superpowers/SKILL.md` |

---

## 二、结构拆解

### 2.1 YAML Frontmatter

```yaml
---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---
```

| 字段 | 作用 |
|------|------|
| `name` | 技能唯一标识符，其他技能通过此名称引用 |
| `description` | **极为关键**。这是 Claude Search Optimization（CSO）的核心——它决定了这个技能何时被自动加载。关键词 `starting any conversation` 确保每个新会话都会注入这套规则 |

### 2.2 子代理豁免条款

```markdown
<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>
```

**设计意图**：子代理（Subagent）通常已经被主代理分配了具体任务和上下文，不需要重新加载自举规则，避免冗余和循环。

---

## 三、核心规则体系

### 3.1 铁律：1% 规则

> *"If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill."*
>
> *"IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT."*
>
> *"This is not negotiable. This is not optional. You cannot rationalize your way out of this."*

这是整个 Superpowers 框架最著名的规则，也是**反理性化（Anti-Rationalization）**的第一道防线：

- **没有选择权**：即使你觉得这个技能可能不适用，也必须调用
- **不可协商**：Agent 不能说服自己跳过技能检查
- **覆盖所有情况**：包括澄清问题、简单查询、快速检查等 Agent 倾向于"走捷径"的场景

### 3.2 指令优先级

明确了三级优先级，解决冲突：

```
1. User's explicit instructions (CLAUDE.md, GEMINI.md, AGENTS.md, direct requests) — highest priority
2. Superpowers skills — override default system behavior where they conflict
3. Default system prompt — lowest priority
```

**关键设计**：用户指令永远最高。如果 `CLAUDE.md` 说"不要用 TDD"，而技能说"总是用 TDD"，**听用户的**。这保证了框架的侵入性可控——用户始终拥有最终控制权。

**示例**：

> If CLAUDE.md, GEMINI.md, or AGENTS.md says "don't use TDD" and a skill says "always use TDD," follow the user's instructions. The user is in control.

---

## 四、多平台适配

### 4.1 各平台技能调用方式

| 平台 | 调用方式 |
|------|----------|
| **Claude Code** | `Skill` 工具 |
| **Copilot CLI** | `skill` 工具（自动发现） |
| **Gemini CLI** | `activate_skill` 工具 |
| **其他环境** | 查阅平台文档 |

### 4.2 工具名映射

> *"Skills use Claude Code tool names."*

由于所有技能文档都使用 Claude Code 的工具名称编写，非 Claude 平台需要通过映射文件适配：

- [`references/copilot-tools.md`](references/copilot-tools.md)
- [`references/codex-tools.md`](references/codex-tools.md)
- [`references/gemini-tools.md`](references/gemini-tools.md)

**Gemini 特殊处理**：工具映射通过 `GEMINI.md` 自动加载，无需手动查阅。

---

## 五、使用规则

### 5.1 核心流程

原文包含一个 Graphviz DOT 流程图，描述了完整的技能调用决策链：

```
User message received
        |
        v
Might any skill apply? ---- definitely not ---> Respond
        |
   yes, even 1%
        v
Invoke Skill tool
        v
Announce: "Using [skill] to [purpose]"
        v
Has checklist? ---- no ---> Follow skill exactly
        |
       yes
        v
Create TodoWrite todo per item
        v
Follow skill exactly


About to EnterPlanMode?
        v
Already brainstormed? ---- no ---> Invoke brainstorming skill ---+
        |                                                        |
       yes------------------------------------------------------>|
                                                                 v
                                                     Might any skill apply?
```

### 5.2 关键行为约束

1. **先调用，后响应**：在任何回应（包括澄清问题）之前必须先调用技能
2. **宣布使用**：调用后必须告知用户 `"Using [skill] to [purpose]"`
3. **检查清单 Todo 化**：技能中的每个检查项必须转为 `TodoWrite` 工具调用
4. **严格遵循**：调用后必须严格按照技能内容执行

---

## 六、红旗表：反理性化防御

这是 Superpowers 框架**最著名的设计**——一张专门阻止 Agent 说服自己绕过流程的对照表：

| Agent 可能的想法 | 现实 |
|-----------------|------|
| "这只是个简单问题" | 问题就是任务。检查技能。 |
| "我需要先了解更多上下文" | 技能检查优先于澄清问题。 |
| "让我先探索代码库" | 技能会告诉你如何探索。先检查。 |
| "我可以快速查看 git/文件" | 文件缺乏对话上下文。检查技能。 |
| "让我先收集信息" | 技能会告诉你如何收集信息。 |
| "这不需要正式技能" | 如果技能存在，就用它。 |
| "我记得这个技能" | 技能会进化。读取当前版本。 |
| "这不算任务" | 行动 = 任务。检查技能。 |
| "技能太重量级了" | 简单的事会变复杂。使用它。 |
| "我先做这一件事" | 在做任何事之前先检查。 |
| "这感觉很有生产力" | 无纪律的行动浪费时间。技能防止这一点。 |
| "我知道那是什么意思" | 知道概念 ≠ 使用技能。调用它。 |

**设计哲学**：Agent（尤其是 LLM）有强烈的自我合理化倾向。这张表是**针对 Agent 认知偏见的防御性设计**，每一条都对应一个真实的 Agent 失败模式。

---

## 七、技能优先级与类型

### 7.1 当多个技能适用时的排序

```
1. Process skills first (brainstorming, debugging) - these determine HOW to approach the task
2. Implementation skills second (frontend-design, mcp-builder) - these guide execution
```

**原则**：先确定"如何做"（Process），再确定"做什么"（Implementation）。

| 场景 | 正确顺序 |
|------|----------|
| "让我们构建 X" | `brainstorming` → 实施技能 |
| "修复这个 Bug" | `systematic-debugging` → 领域特定技能 |

### 7.2 技能类型

| 类型 | 代表 | 约束 |
|------|------|------|
| **Rigid（刚性）** | TDD、debugging | **必须严格执行**。不可为适应上下文而削弱纪律 |
| **Flexible（灵活）** | patterns | 根据上下文调整原则 |

**关键**：技能本身会声明自己是刚性还是灵活。Agent 不能自行判断。

---

## 八、用户指令的边界

> *"Instructions say WHAT, not HOW. 'Add X' or 'Fix Y' doesn't mean skip workflows."*

这是**防止用户无意间绕过流程**的安全条款：

- 用户说"添加 X" → 这是 WHAT（目标）
- 但不能因此跳过 `brainstorming`、`writing-plans`、`test-driven-development` 等流程（HOW）

**例外**：如果用户在 `CLAUDE.md` 或显式指令中说"跳过 TDD"，则遵循 3.2 节的优先级规则，用户指令胜出。

---

## 九、关键信息速查表

| 维度 | 关键信息 |
|------|----------|
| **唯一职责** | 定义 Agent 如何发现、加载、执行其他技能的元规则 |
| **触发机制** | 每个会话开始时通过 `SessionStart` 钩子自动注入 |
| **核心铁律** | 1% 规则——只要有 1% 可能适用，就必须调用技能 |
| **优先级** | 用户显式指令 > Superpowers 技能 > 默认系统提示 |
| **反合理化** | 12 条红旗表，阻止 Agent 说服自己跳过流程 |
| **流程约束** | 必须先调用技能 → 宣布使用 → 创建 Todo（如有）→ 严格执行 |
| **多平台** | Claude Code / Copilot CLI / Gemini CLI / 其他，通过工具映射适配 |
| **计划模式前置条件** | 进入 `EnterPlanMode` 前必须先完成 `brainstorming` |
| **技能分类** | Process 技能优先于 Implementation 技能；Rigid 不可妥协 |
| **用户指令边界** | 用户说 WHAT，不说 HOW；不能因目标明确而跳过流程 |

---

## 十、为什么这个技能如此重要

`using-superpowers` 是**整个框架的行为根基**。它解决了 Agent 工作流中最大的系统性风险：

1. **技能发现失败**：Agent 不知道有哪些技能可用 → 1% 规则强制检查
2. **流程绕过**：Agent 觉得"这次不用那么正式" → 红旗表直接阻止
3. **优先级冲突**：用户指令与技能冲突 → 三级优先级明确裁决
4. **平台碎片化**：不同 Harness 工具名不同 → 统一映射层隐藏差异
5. **上下文丢失**：子代理重复加载 → 豁免条款优雅处理

如果没有这 117 行的"宪法"，其他 14 个技能将只是一堆静态 Markdown 文件，永远不会被触发。

---

*本文档由对 `using-superpowers/SKILL.md` 的逐行精读生成。*
