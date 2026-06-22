# Tests 目录工作总结

> 来源目录：`tests/`
> 作用：对 Superpowers 项目的各核心功能进行端到端测试，覆盖服务器、插件加载、技能触发、技能行为验证和子代理驱动开发等多个维度。

---

## 一、目录结构总览

```
tests/
├── brainstorm-server/      # Brainstorm 服务器单元/集成测试
├── claude-code/            # Claude Code CLI 技能行为测试套件
├── codex-plugin-sync/      # Codex 插件同步脚本测试
├── explicit-skill-requests/# 显式技能请求测试（用户直接点名技能）
├── opencode/               # OpenCode 插件测试套件
├── skill-triggering/       # 隐式技能触发测试（自然语言 prompt）
└── subagent-driven-dev/    # 子代理驱动开发完整集成测试
```

---

## 二、各子目录详解

### 2.1 `brainstorm-server/`

**目的**：测试 `skills/brainstorming/scripts/server.cjs` 这个零依赖 HTTP+WebSocket 服务器。

**包含文件**：
- `server.test.js`：集成测试。通过 `child_process.spawn` 启动真实服务器进程，测试 HTTP 服务、WebSocket 通信、文件监听和头脑风暴工作流完整行为。
- `ws-protocol.test.js`：单元测试。独立测试 WebSocket 协议层，包括：
  - `computeAcceptKey()`：RFC 6455 握手密钥计算，与规范示例值精确比对
  - `encodeFrame()`：帧编码（小帧/中帧/大帧），验证 FIN bit、opcode、payload length 字段
  - `decodeFrame()`：帧解码，测试客户端掩码帧、分片帧、PING/PONG/CLOSE 帧
  - 零依赖限制验证：确保被测模块不 require `ws` 等外部库
- `windows-lifecycle.test.sh`：Shell 测试，验证服务器在 Windows/MSYS2 环境下的 60 秒生命周期检查（因 Node.js 无法看到 MSYS2 PID，须禁用 OWNER_PID 监控）。
- `package.json`：声明 `ws` 为测试专用依赖（不随插件分发）。

**核心设计亮点**：
- 采用 TDD 模式：`ws-protocol.test.js` 在实现前先写完，`process.exit(1)` 保证失败可见
- 服务器通过 `server-started` 标志异步等待启动，确保测试时序稳定

---

### 2.2 `claude-code/`

**目的**：通过 Claude Code CLI（`claude -p`）无头模式调用，验证技能被正确加载并按预期指导 Agent 行为。

**包含文件**：

| 文件 | 类型 | 说明 |
|------|------|------|
| `test-helpers.sh` | 工具库 | 提供 `run_claude`、`assert_contains`、`assert_not_contains`、`assert_order` 等断言函数；`create_test_project`、`cleanup_test_project` 管理临时工作目录 |
| `run-skill-tests.sh` | 主运行器 | 解析 `--verbose`、`--test`、`--timeout`、`--integration` 参数，分离快速测试和集成测试两档 |
| `test-subagent-driven-development.sh` | 快速测试 | 验证技能加载后 Claude 能描述正确工作流顺序（spec review 先于 code quality review）、自审要求、计划只读一次等核心行为 |
| `test-subagent-driven-development-integration.sh` | 集成测试 | 实际跑完整的 subagent-driven-development 工作流 |
| `test-requesting-code-review.sh` | 集成测试 | 植入 SQL 注入漏洞和密码哈希泄露 Bug，验证代码审查子代理能将其标记为 Critical/Important |
| `test-document-review-system.sh` | 集成测试 | 创建含蓄意错误（TODO 未完成、架构描述模糊）的 spec 文档，验证文档审查子代理能发现这些问题 |
| `test-worktree-native-preference.sh` | 行为测试 | RED-GREEN-REFACTOR 三阶段验证：无技能时 Agent 用 `git worktree add`，有 Step 1a（显式命名 EnterWorktree 工具 + consent bridge）后 Agent 改用原生工具；已验证 50/50 次零失败率 |
| `analyze-token-usage.py` | 工具脚本 | 解析 stream-json 格式的会话记录，按主会话和子代理 ID 分类统计 input_tokens、output_tokens、cache_creation、cache_read 消耗 |

---

### 2.3 `codex-plugin-sync/`

**目的**：测试 `scripts/sync-to-codex-plugin.sh` 这个同步脚本的正确性。

**包含文件**：
- `test-sync-to-codex-plugin.sh`：Shell 测试，使用 `assert_equals`、`assert_contains`、`assert_not_contains`、`assert_matches` 四类断言，验证版本号替换、manifest 字段更新、文件内容生成是否符合预期，测试中固定了 `PACKAGE_VERSION=1.2.3` 和 `MANIFEST_VERSION=9.8.7` 以保证结果确定性。

---

### 2.4 `explicit-skill-requests/`

**目的**：验证用户在不使用插件命名空间前缀的情况下，直接点名技能时 Claude 是否会正确调用该技能。

**结构**：
- `run-test.sh`：核心运行器，为每次测试创建隔离的 HOME 目录（避免用户上下文干扰），自动搭建包含 `docs/superpowers/plans/` 结构的虚拟项目目录。
- `run-all.sh`：批量运行所有显式请求测试（subagent-driven-development、systematic-debugging、brainstorming 等）。
- `prompts/`：存放各技能的触发 prompt 文件（如 `subagent-driven-development-please.txt`、`use-systematic-debugging.txt`）。

**关键机制**：
- 通过 `--output-format stream-json` 捕获 Claude 的 tool 调用记录
- 在 JSON 输出中检索 `"skill":"skillname"` 或 `"skill":"namespace:skillname"` 字段判断技能是否被调用

---

### 2.5 `opencode/`

**目的**：针对 OpenCode 平台的插件集成测试。

**包含文件**：

| 文件 | 说明 |
|------|------|
| `setup.sh` | 初始化隔离测试环境（独立 HOME、config 目录、plugin symlink、个人测试技能 fixture） |
| `run-tests.sh` | 主运行器，支持 `--integration`、`--verbose`、`--test` 参数 |
| `test-plugin-loading.sh` | 验证 plugin symlink 存在且目标有效、skills 目录非空、`using-superpowers` 技能必存、JS 语法合法、bootstrap 不引用错误路径 |
| `test-bootstrap-caching.sh` | 测试 bootstrap 内容缓存机制（issue #1202）：验证首次 transform 后结果被缓存；`SKILL.md` 缺失时也缓存未找到状态，不每次重探 |
| `test-bootstrap-caching.mjs` | 上述测试的 Node.js 实现逻辑 |
| `test-tools.sh` | 集成测试：验证 `use_skill` 和 `find_skills` 工具（需 OpenCode 运行时） |
| `test-priority.sh` | 集成测试：在 superpowers、personal（`~/.opencode/skills/`）、project（`.opencode/skills/`）三个位置分别创建同名技能，并注入唯一 marker，验证优先级解析是否正确（project > personal > superpowers） |

---

### 2.6 `skill-triggering/`

**目的**：用自然语言 prompt（不点名技能）测试 Agent 能否自动识别并触发正确的技能。

**覆盖技能**：
- `systematic-debugging`：给出一段测试失败的错误栈，期望触发调试技能
- `test-driven-development`：TDD 相关场景
- `writing-plans`：描述多步骤需求，期望触发计划书写技能
- `dispatching-parallel-agents`：多个独立失败场景，期望触发并行代理分发技能
- `executing-plans`：有现成计划需要执行的场景
- `requesting-code-review`：完成任务后触发代码审查场景

**运行机制**：
- `run-test.sh`：启动 `claude -p` 并传入 `--plugin-dir`、`--dangerously-skip-permissions`、`--max-turns 3`、`--output-format stream-json`，最终在 stream-json 输出中查找 Skill 工具的调用记录判定通过或失败。
- `run-all.sh`：遍历上述 6 个技能，汇总 PASSED/FAILED 结果。

---

### 2.7 `subagent-driven-dev/`

**目的**：端对端验证 `subagent-driven-development` 技能在真实代码项目上的完整执行效果。

**包含子目录**：
- `go-fractals/`：一个 Go 语言分形绘制项目的脚手架，用于测试 Agent 实现计划并通过 Go 测试
- `svelte-todo/`：一个 Svelte 待办事项应用脚手架，用于测试前端技术栈的计划执行

**运行机制**：
- `run-test.sh`：接受 `<test-name>` 参数，调用对应目录的 `scaffold.sh` 搭建项目，再用 `--plugin-dir` 启动 Claude 执行计划，最后校验测试结果（Go `go test ./...` 或前端构建通过）。

---

## 三、测试基础设施模式

### 断言库
所有 Shell 测试文件共享一套断言模式：
- `pass`/`fail`：打印 `[PASS]`/`[FAIL]` + 测试名
- `assert_contains`/`assert_not_contains`：grep 检查输出是否包含特定模式
- `assert_equals`：精确字符串比较
- `assert_matches`/`assert_not_matches`：正则表达式匹配
- `assert_order`：验证 A 在 B 之前出现（用于工作流顺序验证）
- `assert_count`：验证模式出现次数

### 隔离策略
- OpenCode 测试：自定义 HOME 目录，避免污染用户配置
- Claude Code 测试：`create_test_project` 在 `/tmp` 创建 git 仓库，`trap ... EXIT` 确保测试后清理
- Skill triggering 测试：输出写入 `/tmp/superpowers-tests/$TIMESTAMP/` 带时间戳的目录

### 层次划分
| 类型 | 速度 | 依赖 |
|------|------|------|
| 协议单元测试（`ws-protocol.test.js`）| 极快（毫秒级） | Node.js |
| 插件结构检查（`test-plugin-loading.sh`）| 快（秒级） | Node.js |
| 技能行为快速测试（`test-subagent-driven-development.sh`）| 中（30s/测试） | Claude Code CLI |
| 技能触发测试（skill-triggering）| 中（5 分钟/技能）| Claude Code + plugin |
| 完整工作流集成测试（SDD integration）| 慢（10–30 分钟）| Claude Code + plugin |

---

## 四、测试覆盖的核心问题

1. **WebSocket 协议正确性**：服务器能否正确握手、编解码帧，且无外部依赖
2. **插件加载完整性**：symlink 有效、JS 无语法错误、必需技能文件存在
3. **Bootstrap 缓存**：技能文件读取只发生一次，不重复 I/O
4. **技能触发准确性**：Agent 在自然语言 prompt 下是否主动读取正确技能
5. **显式请求解析**：用户点名但不加前缀时技能是否被正确调用
6. **工作流顺序**：spec review 必须在 code quality review 之前；plan 只读一次
7. **原生工具偏好**：使用 `using-git-worktrees` 时 Agent 选用平台原生工具而非裸 `git worktree add`
8. **安全审查能力**：代码审查子代理能否检出 SQL 注入、密码泄露等 Critical 级别漏洞
9. **优先级解析**：project > personal > superpowers 三级技能覆盖规则
10. **同步脚本正确性**：版本号和 manifest 字段同步准确

---

## 五、一句话记忆

`tests/` 目录是 Superpowers 的行为证据库——从协议字节到完整 Agent 工作流，每一层都有对应验证；测试本身也遵循"先写失败测试"的原则，是项目设计决策的可执行说明书。
