# Brainstorming Skill 文档总结

> 原始技能：Brainstorming Ideas Into Designs  
> 作用：通过自然协作对话，将想法转化为完整的设计与规格说明

---

## 一、核心目标

将模糊的想法或需求，经过结构化对话与探索，最终输出为**可落地的设计文档（spec）**，为后续实现提供清晰蓝图。

---

## 二、硬性门槛（HARD-GATE）

> ⚠️ **在呈现设计并获得用户明确批准之前，禁止：**
> - 调用任何实现类 skill（如 frontend-design、mcp-builder 等）
> - 编写任何代码
> - 搭建项目脚手架
> - 采取任何实现行动

**这一规则适用于所有项目，无论 perceived simplicity（ perceived 简单）。**

### 反模式警告："这个太简单了，不需要设计"

即使是一个 todo list、单功能工具、配置修改，都必须走完整流程。"简单"项目恰恰是未经验证的假设造成最大浪费的地方。设计可以简短（几句话），但**必须呈现并获得批准**。

---

## 三、九步检查清单（必须按序完成）

| 步骤 | 内容 | 说明 |
|------|------|------|
| 1 | **探索项目上下文** | 检查文件、文档、最近提交 |
| 2 | **提供视觉辅助**（如需要） | 独立消息，不与其他内容合并 |
| 3 | **提出澄清问题** | 一次一个问题，理解目的/约束/成功标准 |
| 4 | **提出 2-3 种方案** | 附带权衡分析与推荐 |
| 5 | **呈现设计** | 按复杂度分节，每节后获得用户确认 |
| 6 | **编写设计文档** | 保存到 `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` |
| 7 | **规格自审** | 检查占位符、矛盾、歧义、范围 |
| 8 | **用户审查书面规格** | 请用户审核 spec 文件后再继续 |
| 9 | **过渡到实现** | 调用 `writing-plans` skill 创建实施计划 |

### 流程图关键节点

```
探索上下文 → 是否需要视觉辅助？ → 澄清问题 → 提出方案 → 呈现设计 → 用户批准？
                                                                    ↓ 否，修订
                                                              是 → 编写设计文档 → 自审 → 用户审查？
                                                                                          ↓ 否，修改
                                                                                    是 → 调用 writing-plans
```

**终态：调用 `writing-plans`。** 除此之外，不得调用任何其他实现 skill。

---

## 四、各阶段详细要求

### 4.1 理解想法阶段

1. **先检查项目状态**：文件、文档、最近提交
2. **评估范围**：如果请求描述多个独立子系统（如"构建一个包含聊天、文件存储、计费和分析的平台"），立即标记。不要花费时间细化一个需要分解的项目。
3. **过大项目处理**：帮助用户分解为子项目 → 明确独立模块、关系、构建顺序 → 对第一个子项目走正常 brainstorm 流程。每个子项目有自己的 spec → plan → implementation 周期。
4. **一次只问一个问题**：如果需要深入探索，拆分为多个问题
5. **优先使用选择题**：比开放式问题更容易回答
6. **聚焦理解**：目的（purpose）、约束（constraints）、成功标准（success criteria）

### 4.2 探索方案阶段

- **必须提出 2-3 种不同方案**，附带 trade-offs（权衡分析）
- 以对话方式呈现，包含推荐方案及理由
- **首推推荐选项**，并解释原因

### 4.3 呈现设计阶段

- 确信已理解需求后，正式呈现设计
- **每节长度按复杂度调整**：简单问题几句话，复杂问题 200-300 词
- **每节结束后询问**："这部分看起来对吗？"
- **必须覆盖的维度**：
  - 架构（Architecture）
  - 组件（Components）
  - 数据流（Data Flow）
  - 错误处理（Error Handling）
  - 测试（Testing）
- **准备好回溯**：如果某些内容不合理，返回澄清阶段

### 4.4 设计原则：隔离与清晰

- 将系统拆分为**单一职责**的小单元
- 每个单元通过**明确定义的接口**通信
- 每个单元应能独立理解和测试
- 问自己：
  - 不读内部实现，能否理解这个单元做什么？
  - 修改内部实现，是否会破坏消费者？
- **文件过大通常是信号**：说明该单元承担了过多职责

### 4.5 在现有代码库中工作

- 探索现有结构后再提出变更
- 遵循现有模式
- 如果现有代码存在问题（文件过大、边界不清、职责纠缠），将**针对性改进纳入设计**
- **不提出无关重构**：始终聚焦当前目标

---

## 五、设计完成后

### 5.1 文档编写

- **保存路径**：`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
- 用户偏好位置可覆盖此默认路径
- 如有 `elements-of-style:writing-clearly-and-concisely` skill，可使用
- **提交到 git**

### 5.2 规格自审（Spec Self-Review）

写完后以 fresh eyes 检查：

1. **占位符扫描**：是否有 "TBD"、"TODO"、不完整章节、模糊需求？修复它们。
2. **内部一致性**：各章节是否矛盾？架构是否与功能描述匹配？
3. **范围检查**：是否足够聚焦以支持单个实现计划，还是需要进一步分解？
4. **歧义检查**：任何需求是否有两种解释？如果是，选择一种并明确说明。

修复后无需重新审查，直接继续。

### 5.3 用户审查门槛（User Review Gate）

自审通过后，必须请用户审核：

> "Spec written and committed to `<path>`. Please review it and let me know if you want to make any changes before we start writing out the implementation plan."

- 等待用户响应
- 如要求修改，修改后重新运行自审循环
- **仅在用户批准后**才继续

### 5.4 过渡到实现

- **调用 `writing-plans` skill** 创建详细实施计划
- **不得调用任何其他 skill**
- `writing-plans` 是 brainstorm 后的唯一下一步

---

## 六、关键原则（Key Principles）

| 原则 | 说明 |
|------|------|
| **一次一个问题** | 不要一次抛出多个问题，避免 overwhelm |
| **优先选择题** | 比开放式问题更容易回答 |
| **YAGNI 原则** | 无情地移除设计中不必要的功能 |
| **探索替代方案** | 在确定方案前始终提出 2-3 种替代方法 |
| **增量验证** | 先呈现设计，获得批准后再推进 |
| **保持灵活** | 当某些内容不合理时，愿意返回澄清 |

---

## 七、视觉辅助（Visual Companion）

### 7.1 定位

- 浏览器式视觉辅助工具，用于展示 mockups、diagrams、视觉选项
- **作为工具存在，非模式**：接受辅助仅意味着在需要时可用，不意味着每个问题都通过浏览器

### 7.2 提供方式

当预期后续问题涉及视觉内容（mockups、布局、图表）时，**单独发送一条消息**提供：

> "Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)"

**此消息必须独立发送**，不得与澄清问题、上下文摘要或其他内容合并。

### 7.3 使用决策

即使用户同意，也需**针对每个问题**决定是否使用浏览器：

- **使用浏览器**：内容本身是视觉的（mockups、wireframes、布局对比、架构图、并排视觉设计）
- **使用终端**：文本内容（需求问题、概念选择、权衡列表、A/B/C/D 文本选项、范围决策）

> 关于 UI 主题的问题不自动等于视觉问题。"此上下文中 personality 是什么意思？"是概念问题 → 用终端；"哪个 wizard 布局更好？"是视觉问题 → 用浏览器。

### 7.4 后续

如用户同意，在继续前阅读详细指南：`skills/brainstorming/visual-companion.md`

---

## 八、常见陷阱与 Red Flags

| 危险想法 | 现实 |
|----------|------|
| "这个太简单了，不需要设计" | 简单项目最容易因假设错误而浪费工作 |
| "我先写点代码看看" | 硬性门槛禁止任何实现行动 |
| "我已经知道用户要什么" | 仍需呈现设计并获得批准 |
| "直接跳到 writing-plans" | 必须先完成 spec 并获用户批准 |

---

## 九、文件结构与关联关系

`skills/brainstorming/` 目录下包含主文档、子指南、审查模板及视觉辅助工具脚本。它们共同构成完整的 brainstorm 工作流。

### 9.1 文件清单

| 文件 | 类型 | 作用 |
|------|------|------|
| `SKILL.md` | 主文档 | **入口与总纲**，定义完整流程、硬性门槛、九步清单、关键原则、视觉辅助概述 |
| `visual-companion.md` | 子指南 | **视觉辅助详细操作手册**，是 SKILL.md 中视觉辅助章节的展开，说明启动、循环、内容编写、CSS 类、事件格式等 |
| `spec-document-reviewer-prompt.md` | 提示模板 | **规格审查模板**，用于步骤 7（规格自审）或分派子代理审查 spec 文档 |
| `scripts/start-server.sh` | 启动脚本 | 启动视觉辅助本地服务器，按平台适配（macOS/Linux/Windows/Codex/Gemini CLI） |
| `scripts/stop-server.sh` | 停止脚本 | 停止视觉辅助服务器会话 |
| `scripts/server.cjs` | 服务端 | 核心服务器：监视 `screen_dir` 中的 HTML 文件，将最新文件服务到浏览器；接收点击事件写入 `state_dir/events` |
| `scripts/frame-template.html` | 框架模板 | 自动包裹内容片段，提供统一头部、CSS 主题、选择指示器及交互基础设施 |
| `scripts/helper.js` | 客户端脚本 | 浏览器端辅助脚本，处理选项点击、选择状态切换、事件上报 |

### 9.2 文件调用关系

```
┌─────────────────┐
│   SKILL.md      │  ← 入口文档，定义整个 brainstorm 流程
│  (主 skill 文件)  │
└────────┬────────┘
         │
         ├────────────────────────────────────────┐
         │                                        │
         ▼                                        ▼
┌─────────────────┐                    ┌─────────────────────────┐
│ visual-companion│  ← 当用户同意使用    │ spec-document-reviewer  │  ← 步骤 7
│    .md          │    视觉辅助后阅读    │    -prompt.md           │    规格自审/子代理审查
│ (视觉辅助指南)   │                    │ (审查提示模板)           │
└────────┬────────┘                    └─────────────────────────┘
         │
         ├────────────────────────────────────────┐
         │                                        │
         ▼                                        ▼
┌─────────────────┐                    ┌─────────────────────────┐
│ start-server.sh │  ← 启动命令         │   frame-template.html   │  ← 渲染包裹
│  stop-server.sh │  ← 停止命令         │      helper.js          │  ← 交互逻辑
│   server.cjs    │  ← 核心服务         │                         │
│  (scripts/)     │                    │      (scripts/)           │
└─────────────────┘                    └─────────────────────────┘
```

### 9.3 关联说明

1. **SKILL.md ↔ visual-companion.md**
   - SKILL.md 在"视觉辅助"章节中给出概述和使用原则，并指向 `visual-companion.md` 作为详细指南
   - 当用户同意使用视觉辅助后，必须阅读 `visual-companion.md` 才能正确操作服务器循环、编写内容片段、处理事件

2. **SKILL.md ↔ spec-document-reviewer-prompt.md** ⚠️ **待集成**
   - SKILL.md 步骤 7（规格自审）要求检查占位符、一致性、歧义、范围
   - `spec-document-reviewer-prompt.md` 提供了标准化的审查提示模板，设计意图是用于 **dispatch subagent 进行独立审查**
   - **但当前 SKILL.md 并未引用该文件**，主流程仍由同一个 agent 自行审查
   - 该文件属于 **Document Review System** 扩展计划的一部分（详见下方 9.4）

3. **visual-companion.md ↔ scripts/**
   - `visual-companion.md` 是操作手册，`scripts/` 是其实现层
   - `start-server.sh` / `stop-server.sh` 管理服务生命周期
   - `server.cjs` 实现文件监视与浏览器推送机制
   - `frame-template.html` 和 `helper.js` 实现内容包裹与客户端交互
   - `visual-companion.md` 中所有操作指令（启动、写文件、读取事件）均依赖这些脚本

4. **九步清单与文件的映射**

   | 步骤 | 对应文件 |
   |------|----------|
   | 步骤 1~5（探索、提问、方案、设计） | SKILL.md |
   | 步骤 2（视觉辅助提供） | SKILL.md → visual-companion.md → scripts/* |
   | 步骤 6（编写设计文档） | SKILL.md |
   | 步骤 7（规格自审） | SKILL.md（自行审查，未接入 reviewer subagent） |
   | 步骤 8（用户审查） | SKILL.md |
   | 步骤 9（过渡到实现） | SKILL.md → writing-plans skill |

### 9.4 重要发现：spec-document-reviewer-prompt.md 尚未集成

**状态：** 已设计，未完全集成  
**发现时间：** 2026/06/03

#### 背景

`spec-document-reviewer-prompt.md` 是 **Document Review System** 扩展计划的产物，相关文档：

- **设计文档：** `docs/superpowers/specs/2026-01-22-document-review-system-design.md`
- **实施计划：** `docs/superpowers/plans/2026-01-22-document-review-system.md`
- **测试脚本：** `tests/claude-code/test-document-review-system.sh`

#### 设计意图

为工作流增加两个审查循环：

```
brainstorming -> spec -> SPEC REVIEW LOOP -> writing-plans -> plan -> PLAN REVIEW LOOP -> implementation
```

1. **Spec Document Review** —— brainstorm 之后，writing-plans 之前，使用 `spec-document-reviewer-prompt.md`
2. **Plan Document Review** —— writing-plans 之后，implementation 之前，使用 `skills/writing-plans/plan-document-reviewer-prompt.md`

#### 预期接入方式

按实施计划，应在 `SKILL.md` 的 "After the Design" 章节后新增 **Spec Review Loop**：

```markdown
**Spec Review Loop:**
After writing the spec document:
1. Dispatch spec-document-reviewer subagent (see spec-document-reviewer-prompt.md)
2. If ❌ Issues Found:
   - Fix the issues in the spec document
   - Re-dispatch reviewer
   - Repeat until ✅ Approved
3. If ✅ Approved: proceed to implementation setup
```

#### 当前实际状态

| 已完成 | 未完成 |
|--------|--------|
| `spec-document-reviewer-prompt.md` 已创建 | `SKILL.md` **未修改**以接入审查循环 |
| `plan-document-reviewer-prompt.md` 已创建 | `writing-plans/SKILL.md` **未修改** |
| 测试脚本已存在 | 主 skill 流程仍使用"自己审查"方式 |

#### 影响

- **缺少独立审查环节** —— 当前步骤 7 由同一个 agent 自行审查，没有 subagent 的独立视角
- **审查循环的自动化迭代未实现** —— 发现问题后没有自动重审机制
- **资源闲置** —— prompt 模板已就绪但未被调用

#### 后续关注

如需启用该功能，需要：
1. 修改 `skills/brainstorming/SKILL.md`，接入 **Spec Review Loop**
2. 修改 `skills/writing-plans/SKILL.md`，接入 **Plan Review Loop**
3. 验证测试脚本 `test-document-review-system.sh`

---

## 十、总结

`brainstorming` skill 是一个**强制性的设计前置流程**，其核心哲学是：

1. **设计先于实现** —— 无论项目多小
2. **增量验证** —— 每一步都获得用户确认
3. **清晰与隔离** —— 产出边界清晰、职责单一的系统设计
4. **结构化输出** —— 最终生成可落地的 spec 文档，作为 implementation plan 的输入

该 skill 是 Superpowers 体系中**从需求到实现的关键桥梁**，确保所有后续实现工作建立在经用户批准的明确设计基础之上。
