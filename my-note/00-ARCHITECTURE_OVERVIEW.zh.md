# Superpowers 项目架构概览

> 本文档基于对 Superpowers v5.1.0 代码库的全面梳理，涵盖目录结构、核心模块、依赖关系、架构模式与设计原则。
> 生成时间：2026-06-03

---

## 一、项目定位

**Superpowers** 是一个零依赖、多平台（Multi-Harness）的 Agent 技能框架。它不输出可执行代码库，而是输出一套**行为指令（Skills）**——即在恰当的时机自动触发、引导编码 Agent 遵循特定工作流的结构化文档。

- **仓库地址**：`ssh://git@ssh.github.com:443/obra/superpowers.git`
- **版本**：v5.1.0
- **许可证**：MIT
- **作者**：Jesse Vincent / Prime Radiant
- **核心特征**：零运行时依赖、多平台插件分发、行为塑形（Behavioral Shaping）

---

## 二、目录结构全景

```
superpowers/
├── .claude-plugin/              # Claude Code 插件清单
│   ├── marketplace.json
│   └── plugin.json
├── .codex-plugin/               # OpenAI Codex 插件清单
│   └── plugin.json
├── .cursor-plugin/              # Cursor IDE 插件清单
│   └── plugin.json
├── .opencode/                   # OpenCode.ai 插件（ESM）
│   ├── INSTALL.md
│   └── plugins/
│       └── superpowers.js       # 引导插件（bootstrap）
├── .github/                     # GitHub 模板与自动化
│   ├── FUNDING.yml
│   ├── ISSUE_TEMPLATE/
│   ├── modernize/
│   └── PULL_REQUEST_TEMPLATE.md
├── assets/                      # 品牌资源（图标、Logo）
├── docs/                        # 架构文档、实现方案、设计规格
│   ├── plans/                   # 历史实现计划
│   ├── superpowers/
│   │   ├── plans/               # 功能实现计划（按日期归档）
│   │   └── specs/               # 设计规格书（按日期归档）
│   ├── testing.md               # 集成测试方法论
│   ├── windows/
│   │   └── polyglot-hooks.md    # 跨平台钩子文档
│   └── README.opencode.md
├── hooks/                       # 会话启动钩子（跨平台）
│   ├── hooks.json               # Claude Code 钩子配置
│   ├── hooks-cursor.json        # Cursor 钩子配置
│   ├── run-hook.cmd             # Windows/Unix 多语言包装器
│   └── session-start            # Bash 钩子脚本
├── scripts/                     # 构建与发布自动化
│   ├── bump-version.sh          # 跨清单版本号统一升级
│   └── sync-to-codex-plugin.sh  # 同步至 Codex 应用市场
├── skills/                      # ★ 核心：全部 Agent 技能
│   ├── brainstorming/           # 苏格拉底式设计提炼
│   ├── dispatching-parallel-agents/   # 并发子代理调度
│   ├── executing-plans/         # 计划批量执行
│   ├── finishing-a-development-branch/ # 分支收尾决策
│   ├── receiving-code-review/   # 接收代码评审
│   ├── requesting-code-review/  # 发起代码评审
│   ├── subagent-driven-development/   # 子代理驱动开发
│   ├── systematic-debugging/    # 系统化调试
│   ├── test-driven-development/ # 测试驱动开发
│   ├── using-git-worktrees/     # Git 工作树隔离
│   ├── using-superpowers/       # 引导/技能发现（自举）
│   ├── verification-before-completion/ # 完成前验证
│   ├── writing-plans/           # 编写实现计划
│   └── writing-skills/          # 技能撰写方法论
├── tests/                       # 集成测试与评估测试
│   ├── brainstorm-server/       # 头脑风暴服务器测试
│   ├── claude-code/             # Claude Code 集成测试
│   ├── codex-plugin-sync/       # Codex 插件同步测试
│   ├── explicit-skill-requests/ # 显式技能请求测试
│   ├── opencode/                # OpenCode 平台测试
│   ├── skill-triggering/        # 技能自动触发测试
│   └── subagent-driven-dev/     # 子代理开发评估夹具
├── AGENTS.md -> CLAUDE.md       # 符号链接
├── CLAUDE.md                    # 贡献者指南（AI 专用）
├── GEMINI.md                    # Gemini CLI 引导参考
├── README.md                    # 快速开始与安装指南
├── RELEASE-NOTES.md             # 详细变更日志
├── package.json                 # Node ESM 入口（指向 OpenCode 插件）
├── gemini-extension.json        # Gemini CLI 扩展清单
└── .version-bump.json           # 版本号声明清单
```

### 2.1 目录组织原则

- **平台隔离**：每个目标平台拥有独立的插件清单目录（`.claude-plugin/`, `.codex-plugin/` 等），互不影响。
- **技能扁平化**：所有技能统一置于 `skills/` 下，采用**扁平命名空间**，无嵌套子包。
- **文档即代码**：设计决策通过带日期的规格书（`specs/YYYY-MM-DD-*.md`）和实现计划（`plans/YYYY-MM-DD-*.md`）归档，替代传统 ADR。
- **测试按平台拆分**：`tests/` 下按目标 Harness 分目录，确保各平台集成测试独立可运行。

---

## 三、核心模块与职责

### 3.1 技能库（Skills Library）—— 行为塑形层

这是项目的**最深模块（Deep Module）**。14 个技能并非代码库，而是塑造 Agent 行为的结构化指令。每个技能包含：

| 技能名 | 路径 | 核心职责 | 行数 |
|--------|------|----------|------|
| `using-superpowers` | `skills/using-superpowers/` | 自举与技能发现规则 | ~117 |
| `brainstorming` | `skills/brainstorming/` | 编码前苏格拉底式设计提炼 | ~164 |
| `writing-plans` | `skills/writing-plans/` |  bite-sized 实现计划撰写 | ~152 |
| `executing-plans` | `skills/executing-plans/` | 计划批量执行与检查点 | ~70 |
| `subagent-driven-development` | `skills/subagent-driven-development/` | 每任务独立子代理 + 两阶段评审 | ~279 |
| `dispatching-parallel-agents` | `skills/dispatching-parallel-agents/` | 并发子代理调度 | ~182 |
| `test-driven-development` | `skills/test-driven-development/` | RED-GREEN-REFACTOR 周期强制 | ~371 |
| `systematic-debugging` | `skills/systematic-debugging/` | 四阶段根因分析流程 | ~296 |
| `verification-before-completion` | `skills/verification-before-completion/` | 完成前证据门槛 | ~139 |
| `requesting-code-review` | `skills/requesting-code-review/` | 合并前评审发起 | ~103 |
| `receiving-code-review` | `skills/receiving-code-review/` | 技术性（非表演性）反馈处理 | ~213 |
| `using-git-worktrees` | `skills/using-git-worktrees/` | 隔离工作空间管理 | ~215 |
| `finishing-a-development-branch` | `skills/finishing-a-development-branch/` | 合并/PR/清理决策工作流 | ~251 |
| `writing-skills` | `skills/writing-skills/` | 技能撰写的 TDD 方法论 | ~655 |

每个技能的入口为 `SKILL.md`，采用 YAML Frontmatter 声明元数据（`name`, `description`），其中 `description` 被精心打磨以驱动技能加载（Claude Search Optimization）。

### 3.2 插件清单层（Plugin Manifests）—— 平台适配层

| 平台 | 清单路径 | 格式 | 特殊机制 |
|------|----------|------|----------|
| Claude Code | `.claude-plugin/plugin.json` + `hooks/hooks.json` | JSON | SessionStart 钩子注入 |
| OpenAI Codex CLI/App | `.codex-plugin/plugin.json` | JSON | 含 `skills` 与 `interface` 字段 |
| Cursor | `.cursor-plugin/plugin.json` + `hooks/hooks-cursor.json` | JSON | 独立钩子配置 |
| OpenCode.ai | `.opencode/plugins/superpowers.js` | ESM JS | `experimental.chat.messages.transform` 注入 |
| Gemini CLI | `gemini-extension.json` + `GEMINI.md` | JSON + MD | 扩展清单 + 工具映射 |

**关键设计**：所有平台最终都加载 `skills/using-superpowers/SKILL.md` 作为**引导（Bootstrap）**，再由引导按需加载其他技能。

### 3.3 钩子系统（Hooks）—— 会话启动注入层

- `hooks/session-start`：Bash 引导脚本，检测当前 Harness 环境变量（`CLAUDE_PLUGIN_ROOT`, `CURSOR_PLUGIN_ROOT`, `COPILOT_CLI` 等），注入自举内容。
- `hooks/run-hook.cmd`：**多语言包装器**，兼容 Windows CMD 与 Unix Bash，解决跨平台钩子执行问题。
- `hooks/hooks.json` / `hooks-cursor.json`：各平台钩子声明，定义触发时机与注入内容。

### 3.4 自动化脚本层（Scripts）

| 脚本 | 职责 |
|------|------|
| `bump-version.sh` | 跨 6+ 个 JSON 清单文件统一升级版本号，含漂移检测与未声明文件审计 |
| `sync-to-codex-plugin.sh` | 克隆下游 Fork，rsync 上游内容（锚定排除模式），保留目标元数据，通过 `gh` 提 PR |

### 3.5 测试体系（Tests）

按平台与测试类型拆分：

- **集成测试**：`claude-code/`, `opencode/` —— 在真实/模拟 Harness 环境中验证技能触发与行为。
- **触发测试**：`skill-triggering/`, `explicit-skill-requests/` —— 验证技能在特定用户消息下是否自动加载。
- **评估夹具**：`subagent-driven-dev/` —— 包含 Go 分形、Svelte Todo 等完整项目，用于测试子代理开发流程。
- **服务器测试**：`brainstorm-server/` —— 针对零依赖 WebSocket 服务器的协议与生命周期测试。

---

## 四、依赖关系图

### 4.1 平台消费关系

```
┌─────────────────────────────────────────────────────────────┐
│                      Harnesses (消费者)                       │
├─────────────┬─────────────┬─────────────┬───────────────────┤
│ Claude Code │ Codex CLI   │ Cursor      │ OpenCode.ai       │
│   ┌───┐     │   ┌───┐     │   ┌───┐     │   ┌───────────┐   │
│   │JSON│     │   │JSON│     │   │JSON│     │   │ ESM Plugin│   │
│   └───┘     │   └───┘     │   └───┘     │   └───────────┘   │
├─────────────┴─────────────┴─────────────┴───────────────────┤
│                      统一引导层                               │
│            skills/using-superpowers/SKILL.md                 │
│                     (自举/技能发现)                           │
└──────────────────────────┬──────────────────────────────────┘
                           │ 按需加载
                           ▼
            ┌──────────────────────────────┐
            │      skills/*/SKILL.md        │
            │    (14 个行为塑形技能)         │
            └──────────────────────────────┘
```

### 4.2 技能工作流链

```
[brainstorming] ──► [writing-plans] ──► [executing-plans]
                         │
                         ▼
              [subagent-driven-development]
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
   [using-git-    [test-driven-    [requesting-code-]
    worktrees]     development]     review]
                                         │
                                         ▼
                           [finishing-a-development-branch]

[systematic-debugging] ──► Phase 4 ──► [test-driven-development]

[verification-before-completion] ──► 用于所有完成点 gate check
```

### 4.3 核心依赖原则

- **技能间通过名称引用**：如 `superpowers:test-driven-development`，而非文件路径。由 Harness 的 `Skill` 工具在运行时解析。
- **零第三方依赖**：核心无 npm 依赖（仅 `brainstorm-server` 测试目录引入 `ws`）。纯 Bash 脚本完成自动化，纯 Node.js（无框架）实现服务器。
- **平台适配通过工具映射**：`skills/using-superpowers/references/` 下包含 `codex-tools.md`、`copilot-tools.md`、`gemini-tools.md`，将 Claude Code 工具名映射至各平台等效工具。

---

## 五、架构模式

### 5.1 多平台插件架构（Multi-Harness Plugin Architecture）

- **单一源码，多端分发**：同一份 `skills/` 源码通过不同格式的清单文件发布到 7+ 个 Agent 平台。
- **清单格式异构共存**：JSON（Claude/Codex/Cursor）、ESM JS（OpenCode）、纯 Markdown（Gemini）在同一仓库和平共存。
- **平台特定逻辑最小化**：平台差异尽量收敛在清单与钩子层，不侵入技能内容本身。

### 5.2 技能驱动的行为塑形（Skill-Based Behavioral Shaping）

- **文档即接口**：技能不是代码库，而是**结构化行为指令**。Agent 阅读并遵循这些指令，从而改变默认行为。
- **描述字段即触发器（CSO）**：`SKILL.md` 中的 `description` 字段经过精心调优，确保在用户相关请求时自动触发技能加载（Claude Search Optimization）。
- **层级覆盖**：技能覆盖默认系统提示行为，但**显式用户指令**拥有最高优先级。

### 5.3 钩子驱动的引导注入（Hook-Driven Bootstrap Injection）

- **SessionStart 钩子**：在每个新会话开始时自动注入 `using-superpowers` 引导，使技能能够自动触发。
- **多语言包装器（Polyglot Wrapper）**：`.cmd` 文件同时兼容 Windows CMD 与 Unix Bash，解决跨平台钩子执行的难题。
- **环境变量检测**：钩子通过检测特定环境变量判断当前 Harness 类型，动态调整注入内容。

### 5.4 零依赖设计（Zero-Dependency Design）

- **理念**：Superpowers 核心刻意保持零运行时依赖，确保在任何环境中都能即插即用。
- **表现**：
  - 无 `node_modules`（生产环境）
  - 无构建工具链（无 `tsconfig.json`、无 ESLint、无 Prettier）
  - 纯 Bash 完成版本管理与同步
  - 手写 RFC 6455 WebSocket 实现（`brainstorming/scripts/server.cjs`）
- **红线**：PR 若引入第三方依赖（除非是支持全新 Harness），将被拒绝。

### 5.5 文档的测试驱动开发（TDD-Applied-to-Documentation）

- **`writing-skills` 技能**：将技能撰写视为 RED-GREEN-REFACTOR 循环。
- **压力测试**：通过子代理对技能进行对抗性压力测试，验证行为合规性。
- **铁律**："没有先失败的测试，就没有新技能"（No skill without a failing test first）。

### 5.6 扁平命名空间的单体仓库（Monorepo with Flat Skill Namespace）

- 所有技能置于 `skills/` 下的扁平命名空间，无子包或嵌套模块。
- 技能的辅助文件（提示模板、脚本、参考资料）存放在各自技能目录内，保持局部性（Locality）。

---

## 六、设计原则

### 6.1 测试驱动开发（Test-Driven Development）
 foundational 原则。无论是代码还是文档，都必须先写失败的测试，再写最小实现，最后重构。在 `writing-skills` 中，这一原则被直接应用于技能文档的迭代。

### 6.2 系统化优于随意（Systematic over Ad-hoc）
 强调流程胜过直觉。调试技能强制四阶段流程；计划技能强制编码前设计；评审技能强制两阶段评审顺序。

### 6.3 复杂度削减（Complexity Reduction / YAGNI）
 技能文档中反复出现"你不需要它"（You Aren't Gonna Need It）的警告。禁止过度工程化，接口应尽可能小（高 Leverage）。

### 6.4 证据优于宣称（Evidence over Claims）
 `verification-before-completion` 要求 Agent 在宣称成功前必须运行命令、读取输出并提供证据。这是防止幻觉（Hallucination）的关键 gate。

### 6.5 "人类伙伴"术语体系（Human Partner Terminology）
 这是**刻意选择的声音（Voice）**。Agent 在技能文档中始终称用户为 "your human partner" 而非 "the user"，以此塑造协作而非从属的 Agent 行为。

### 6.6 反合理化（Anti-Rationalization）
 技能文档中包含大量的**红旗表（Red Flags Tables）**和**合理化对策列表**。Agent 被明确禁止说服自己绕过流程——这是针对 Agent 自我合理化倾向的防御性设计。

### 6.7 每个任务独立上下文（Fresh Context per Task）
 `subagent-driven-development` 要求为每个任务派发独立的子代理，确保上下文干净，防止上下文污染（Context Pollution）。

### 6.8 两阶段评审（Two-Stage Review）
 顺序至关重要：先进行**规格合规评审**（Spec Compliance Review），再进行**代码质量评审**（Code Quality Review）。不可颠倒。

### 6.9 跨平台兼容（Cross-Platform Compatibility）
 多语言钩子、平台原生工具优先（如优先使用平台原生 worktree 工具而非 `git worktree`）、Windows 生命周期特殊处理。

### 6.10 透明与归属（Transparency & Attribution）
 贡献者必须在 PR 中披露使用的模型、Harness、插件版本及人工审核者。隐瞒 Agent 作者身份是关闭 PR 的理由。94% 的 PR 拒绝率部分源于此。

### 6.11 Token 效率（Token Efficiency）
 技能文档被激进地压缩：
 - 引导工作流目标 < 150 词
 - 频繁加载的技能目标 < 200 词
 - 其他技能目标 < 500 词
 这是为了在 Agent 上下文窗口中保持高效。

### 6.12 无例外铁律（No Exceptions Without Human Permission）
 在 TDD、调试、验证、技能撰写等多个技能中出现"Iron Law"模式：纪律是不可协商的，Agent 不得自行决定跳过步骤，必须获得人类伙伴的明确许可。

---

## 七、关键实现细节

### 7.1 OpenCode 插件（`.opencode/plugins/superpowers.js`）

- 使用 `experimental.chat.messages.transform` 将引导内容注入**第一条用户消息**，避免系统消息 Token 膨胀。
- 模块级缓存防止在 Agent 每一步重复读取磁盘。
- `package.json` 的 `main` 字段指向此文件，使其成为 Node ESM 入口。

### 7.2 头脑风暴服务器（`skills/brainstorming/scripts/server.cjs`）

- 零依赖 Node.js HTTP/WebSocket 服务器，**手写 RFC 6455 协议实现**。
- 提供视觉头脑风暴内容，支持自动换行的 HTML 片段服务。
- 配套完整的生命周期测试（`tests/brainstorm-server/`）。

### 7.3 版本同步机制

- `.version-bump.json` 显式声明所有包含版本号的文件路径。
- `bump-version.sh` 使用 `jq` 统一升级，包含漂移检测与未声明文件审计。
- `sync-to-codex-plugin.sh` 通过 rsync 的锚定排除模式精确控制同步内容。

---

## 八、架构词汇表（项目内统一术语）

本项目在架构讨论中使用以下精确定义：

| 术语 | 定义 |
|------|------|
| **Module（模块）** | 任何具有接口与实现的事物：函数、类、包、切片。 |
| **Interface（接口）** | 调用者使用模块所需了解的一切：类型、不变量、错误模式、顺序、配置。不仅是类型签名。 |
| **Implementation（实现）** | 模块内部的代码。 |
| **Depth（深度）** | 接口处的杠杆：小接口背后的大量行为。**Deep** = 高杠杆；**Shallow** = 接口几乎与实现一样复杂。 |
| **Seam（接缝）** | 接口所在之处；可以在不就地编辑的情况下改变行为的位置。 |
| **Adapter（适配器）** | 在接缝处满足接口的具体事物。 |
| **Leverage（杠杆）** | 调用者从深度中获得的好处。 |
| **Locality（局部性）** | 维护者从深度中获得的好处：变更、缺陷、知识集中于一处。 |

### 核心原则

- **删除测试**：想象删除一个模块。如果复杂度消失了，它是一个传透模块（Pass-through）；如果复杂度在 N 个调用方重新出现，它才物有所值。
- **接口即测试面**：测试应通过接口进行，而非拆解实现。
- **一个适配器 = 假设的接缝；两个适配器 = 真实的接缝**。

---

## 九、总结

Superpowers 的架构本质上是**围绕文档与行为而非代码与数据**构建的。它通过以下方式实现高 Leverage 与 Locality：

1. **深模块**：14 个技能每个都将大量行为塑形逻辑隐藏在极小的接口（YAML Frontmatter + 精炼描述）之后。
2. **清晰接缝**：平台适配层（插件清单、钩子、工具映射）与核心技能内容完全分离，允许新增 Harness 而不触及技能本体。
3. **零依赖**：消除了外部依赖带来的版本、安全与可用性风险，使项目在任何环境中可立即运行。
4. **流程即代码**：将 TDD、调试、评审等软件开发流程编码为 Agent 可阅读、可遵循的指令，实现了对 Agent 行为的系统性约束而非临时性提示。

---

*本文档由 `improve-codebase-architecture` skill 辅助生成。*
