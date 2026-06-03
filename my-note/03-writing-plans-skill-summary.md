# Writing-Plans Skill 文档总结

> 原始技能：Writing Plans  
> 作用：根据 spec 编写全面、可执行的实施计划，将设计转化为 bite-sized 任务

---

## 一、核心目标

将已批准的 **spec（设计文档）** 转化为工程师可以**零上下文执行**的详细实施计划。

**假设前提：**
- 工程师是熟练开发者，但**几乎不了解代码库和领域知识**
- 工程师不太擅长测试设计
- 工程师可能有 questionable taste（需要明确指导）

**输出：** 保存到 `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`

**核心原则贯穿始终：**
- **DRY** —— 不要重复
- **YAGNI** —— 不要过度设计
- **TDD** —— 测试驱动
- **Frequent commits** —— 频繁提交

---

## 二、关键要求（硬性规范）

### 2.1 启动声明

开始时必须声明：
> "I'm using the writing-plans skill to create the implementation plan."

### 2.2 任务粒度

**每个步骤是一个动作，耗时 2-5 分钟：**

| 好例子 | 坏例子 |
|--------|--------|
| "Write the failing test" | "Implement the feature" |
| "Run it to make sure it fails" | "Add tests" |
| "Implement the minimal code to make the test pass" | "Handle edge cases" |
| "Commit" | "Clean up" |

### 2.3 无占位符（No Placeholders）

以下内容是**计划失败**，绝对禁止：

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above"（不带实际测试代码）
- "Similar to Task N"（工程师可能乱序阅读任务）
- 只描述做什么但不展示怎么做（代码步骤必须含代码块）
- 引用未在任何任务中定义的类型、函数或方法

### 2.4 必须包含的精确信息

- **精确文件路径** —— `exact/path/to/file.py`
- **完整代码** —— 每个改变代码的步骤都要展示代码
- **精确命令及预期输出** —— `pytest tests/path/test.py::test_name -v` + Expected: PASS

---

## 三、计划文档结构

### 3.1 文档头部（必须）

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

### 3.2 任务结构

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

---

## 四、编写流程（七步）

| 步骤 | 内容 | 说明 |
|------|------|------|
| 1 | **Scope Check** | 检查 spec 是否覆盖多个独立子系统。如是，建议拆分为多个计划 |
| 2 | **File Structure** | 映射将被创建或修改的文件，明确每个文件的职责 |
| 3 | **Bite-Sized Task Granularity** | 将工作拆分为 2-5 分钟粒度的步骤 |
| 4 | **Plan Document Header** | 写入强制头部模板 |
| 5 | **Task Structure** | 写入具体任务，含 Files、Steps、代码、命令、预期输出 |
| 6 | **Self-Review** | 对照 spec 自检覆盖率、占位符、类型一致性 |
| 7 | **Execution Handoff** | 提供两种执行方式供用户选择 |

---

## 五、各阶段详细要求

### 5.1 Scope Check（范围检查）

- 如果 spec 覆盖多个独立子系统，应在 brainstorm 阶段就拆分
- 如果未拆分，建议拆分为**独立的计划** —— 每个子系统一个计划
- 每个计划应产出**可独立运行、可测试**的软件

### 5.2 File Structure（文件结构）

在定义任务前，先映射文件：

- **设计边界清晰的单元** —— 每个文件单一职责
- **偏好小而聚焦的文件** —— 文件过大通常意味着职责过多
- **一起变更的文件应放在一起** —— 按职责拆分，而非按技术层拆分
- **遵循现有模式** —— 现有代码库使用大文件时不要单方面重构，但如果要修改的文件已过于臃肿，在计划中包含拆分是合理的

### 5.3 Task Granularity（任务粒度）

- **每个步骤是一个动作**，不是一类动作
- **TDD 循环**是最小单元：写失败测试 → 运行确认失败 → 最小实现 → 运行确认通过 → 提交
- **频繁提交** —— 每个逻辑单元完成后都提交

### 5.4 Self-Review（自审）

写完计划后，以 fresh eyes 对照 spec 检查：

1. **Spec Coverage（spec 覆盖率）**
   - 浏览 spec 的每个章节/需求
   - 能否指出实现它的任务？
   - 列出任何缺口

2. **Placeholder Scan（占位符扫描）**
   - 搜索计划中的 red flags —— "No Placeholders" 中列出的任何模式
   - 修复它们

3. **Type Consistency（类型一致性）**
   - 后续任务中使用的类型、方法签名、属性名是否与前面任务定义的一致？
   - Task 3 中叫 `clearLayers()`，Task 7 中叫 `clearFullLayers()` 是 bug

**发现问题 → inline 修复 → 无需重新审查 → 继续**

### 5.5 Execution Handoff（执行移交）

保存计划后，提供两种执行选项：

> "Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:
>
> **1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration
>
> **2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints
>
> **Which approach?"

**用户选择后：**

| 选项 | 调用的 Skill |
|------|-------------|
| Subagent-Driven | `superpowers:subagent-driven-development` |
| Inline Execution | `superpowers:executing-plans` |

---

## 六、文件结构与关联关系

`skills/writing-plans/` 目录下包含主文档和审查模板。

### 6.1 文件清单

| 文件 | 类型 | 作用 |
|------|------|------|
| `SKILL.md` | 主文档 | **入口与总纲**，定义计划结构、任务粒度、无占位符规则、自审清单、执行移交 |
| `plan-document-reviewer-prompt.md` | 提示模板 | **计划审查模板**，设计意图是 dispatch subagent 审查计划完整性、spec 对齐度、任务可执行性 |

### 6.2 文件调用关系

```
┌─────────────────┐
│   SKILL.md      │  ← 入口文档，定义计划编写流程
│  (主 skill 文件)  │
└────────┬────────┘
         │
         ├────────────────────────────────────────┐
         │                                        │
         ▼                                        ▼
┌─────────────────────────┐            ┌─────────────────────────┐
│  superpowers:subagent-  │            │ plan-document-reviewer  │  ← 待集成
│   driven-development    │            │    -prompt.md           │
│  (执行方式一)            │            │ (审查提示模板)           │
└─────────────────────────┘            └─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│  superpowers:executing  │
│        -plans           │
│  (执行方式二)            │
└─────────────────────────┘
```

### 6.3 关联说明

1. **SKILL.md → 执行 Skills**
   - 计划编写完成后，必须通过 **Execution Handoff** 调用下游 skill 执行
   - **不得直接开始写代码** —— 必须通过 `subagent-driven-development` 或 `executing-plans` 执行
   - 这是与 `brainstorming` skill 类似的硬性门槛

2. **SKILL.md ↔ plan-document-reviewer-prompt.md** ⚠️ **待集成**
   - SKILL.md 的 Self-Review 部分要求当前 agent **自行审查**计划
   - `plan-document-reviewer-prompt.md` 的设计意图是 **dispatch subagent 进行独立审查**
   - **但当前 SKILL.md 并未引用该文件**，属于 Document Review System 扩展计划中未完成的集成
   - 与 `brainstorming` 中的 `spec-document-reviewer-prompt.md` 处于相同状态

---

## 七、与 Brainstorming Skill 的衔接

```
Brainstorming ──► Spec 文档 ──► Writing-Plans ──► Plan 文档 ──► 执行 (subagent/inline)
     │                │               │                │
     │                │               │                └──► subagent-driven-development
     │                │               │                     executing-plans
     │                │               │
     │                │               └──► 基于 spec 编写实施计划
     │                │
     │                └──► docs/superpowers/specs/YYYY-MM-DD-*.md
     │
     └──► 调用 writing-plans skill（Brainstorming 的唯一下一步）
```

**衔接规则：**
- `brainstorming` 完成后**必须**调用 `writing-plans`
- `writing-plans` 的输入是 `brainstorming` 产出的 **spec 文档**
- `writing-plans` 完成后**必须**调用 `subagent-driven-development` 或 `executing-plans`
- **不得跳过计划直接编码**

---

## 八、常见陷阱与 Red Flags

| 危险想法 | 现实 |
|----------|------|
| "这个计划太简单，不需要详细步骤" | 每个步骤都必须精确到代码和命令 |
| "先写个大致框架，细节后面补" | "TODO"、"TBD"、"implement later" 是绝对禁止的占位符 |
| "Similar to Task 3" | 工程师可能乱序阅读，每个任务必须自包含 |
| "跳过计划直接开始编码" | 必须通过 subagent-driven-development 或 executing-plans 执行 |
| "只描述做什么，不写怎么做" | 代码步骤必须包含完整代码块 |

---

## 九、总结

`writing-plans` skill 是一个**强制性的计划编制流程**，其核心哲学是：

1. **计划先于编码** —— 没有计划就不写代码
2. **零上下文可执行** —— 计划必须精确到让不了解代码库的工程师也能执行
3. **TDD 驱动** —— 每个任务遵循红-绿-重构循环
4. **无占位符** —— 计划中的每个字都必须是可执行的指令
5. **结构化移交** —— 计划完成后必须通过专门的执行 skill 实施

该 skill 是 Superpowers 体系中**从设计到实现的关键桥梁**，确保所有编码工作建立在经详细规划、可逐项执行的任务清单之上。
