# Hooks 工作总结：会话启动自举与跨平台注入

> 生成时间：2026-06-23
> 关注范围：`hooks/` 目录中的 `SessionStart` 钩子，以及 OpenCode 中承担同类职责的插件 hook。

---

## 一、核心结论

Superpowers 的 hooks 系统做的事情很集中：**在编码 Agent 的新会话开始时，把 `using-superpowers` 技能完整注入上下文，让 Agent 从第一轮对话开始就知道自己拥有 Superpowers，并按技能系统工作。**

这套机制不是业务功能，而是 Superpowers 的自举层。它解决了三个问题：

1. **启动时机**：在会话创建、清空或压缩后自动运行，而不是等用户手动加载。
2. **内容注入**：读取 `skills/using-superpowers/SKILL.md`，包装成平台能识别的 JSON 输出。
3. **跨平台执行**：同一套仓库要兼容 Claude Code、Cursor、GitHub Copilot CLI、Windows CMD、macOS/Linux shell 等不同运行环境。

一句话概括：

```text
SessionStart hook
  -> run-hook.cmd 跨平台启动器
  -> session-start 读取 using-superpowers
  -> 输出平台特定 JSON
  -> Agent 第一轮上下文中已经拥有 Superpowers 引导
```

---

## 二、涉及文件

| 文件 | 职责 |
|------|------|
| `hooks/hooks.json` | Claude Code 的 `SessionStart` hook 声明。 |
| `hooks/hooks-cursor.json` | Cursor 插件系统使用的 hook 声明。 |
| `hooks/run-hook.cmd` | Windows CMD 与 Unix shell 都能运行的多语言包装器。 |
| `hooks/session-start` | 真正生成自举上下文 JSON 的 Bash 脚本。 |
| `.cursor-plugin/plugin.json` | Cursor 清单，显式指向 `./hooks/hooks-cursor.json`。 |
| `.opencode/plugins/superpowers.js` | OpenCode 的同类 hook 实现，不走 `hooks/` 目录。 |
| `docs/windows/polyglot-hooks.md` | 解释跨平台 polyglot hook 技术的设计文档。 |
| `RELEASE-NOTES.md` | 记录 hooks 在 Windows、Cursor、Copilot、Bash 等平台上的演进。 |

---

## 三、Claude Code hook 配置

`hooks/hooks.json` 定义的是 Claude Code 的 `SessionStart` hook：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

关键点：

1. **触发事件是 `SessionStart`**：这是会话级 hook，不是工具调用前后 hook。
2. **触发场景是 `startup|clear|compact`**：
   - `startup`：新会话启动时注入。
   - `clear`：清空上下文后重新注入。
   - `compact`：上下文压缩后重新注入。
   - 不包含 `resume`，避免恢复已有会话时重复注入已有上下文。
3. **命令通过 `run-hook.cmd` 间接执行**：不是直接运行 `session-start`，因为 Windows 下直接运行扩展名为空或 `.sh` 文件都可能触发错误行为。
4. **路径使用转义双引号**：`"${CLAUDE_PLUGIN_ROOT}/..."` 外层以 JSON 形式转义，兼容路径中含空格的情况，也避免单引号在 Windows CMD 中失效。
5. **`async: false`**：同步执行，确保第一轮模型响应前已经拿到 `using-superpowers` 内容。

这个文件的目标不是复杂，而是可靠：让 Claude Code 在最早的可用时机获得 Superpowers 引导。

---

## 四、Cursor hook 配置

`hooks/hooks-cursor.json` 是 Cursor 侧的配置：

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "command": "./hooks/run-hook.cmd session-start"
      }
    ]
  }
}
```

和 Claude Code 相比，差异主要在接口格式：

| 项目 | Claude Code | Cursor |
|------|-------------|--------|
| 事件名 | `SessionStart` | `sessionStart` |
| 配置文件 | `hooks/hooks.json` | `hooks/hooks-cursor.json` |
| 配置版本 | 无显式版本 | `version: 1` |
| 命令路径 | `"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd" session-start` | `./hooks/run-hook.cmd session-start` |
| 输出字段 | `hookSpecificOutput.additionalContext` | `additional_context` |

`.cursor-plugin/plugin.json` 通过 `"hooks": "./hooks/hooks-cursor.json"` 显式把 Cursor 插件清单接到这个配置文件。

重要维护点：`session-start` 脚本里优先检测 `CURSOR_PLUGIN_ROOT`，因为 Cursor 可能同时设置 `CLAUDE_PLUGIN_ROOT`。如果先判断 Claude 变量，Cursor 会拿到错误的输出格式。

---

## 五、`run-hook.cmd`：跨平台启动器

`run-hook.cmd` 是 hooks 系统中最有工程味的文件。它是一个 polyglot wrapper：同一个文件既能被 Windows CMD 当作批处理脚本运行，也能被 Unix shell 当作 shell 脚本运行。

### 5.1 Windows 路线

Windows CMD 看到开头：

```cmd
: << 'CMDBLOCK'
@echo off
...
CMDBLOCK
```

会把 `:` 当成 label 语法，继续执行批处理部分。它做的事是：

1. 检查是否传入脚本名，没有就输出 `run-hook.cmd: missing script name` 并退出。
2. 用 `%~dp0` 计算 hook 目录。
3. 优先查找 Git for Windows 的标准 Bash 路径：
   - `C:\Program Files\Git\bin\bash.exe`
   - `C:\Program Files (x86)\Git\bin\bash.exe`
4. 如果标准路径不存在，再用 `where bash` 查找 PATH 中的 Bash。
5. 找到 Bash 后执行：

```cmd
bash "%HOOK_DIR%%~1" %2 %3 ...
```

也就是把第一个参数当作真正 hook 脚本名，例如：

```text
run-hook.cmd session-start
  -> bash "<hooks-dir>\session-start"
```

如果 Windows 上没有 Bash，它会静默退出 `0`。这是一种可用性取舍：插件仍然可安装，只是不会获得 SessionStart 上下文注入。

### 5.2 Unix 路线

Unix shell 会把开头的 CMD 区块当作 heredoc 输入吃掉，然后运行文件底部的 shell 代码：

```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

它做的事是：

1. 计算 `run-hook.cmd` 所在目录。
2. 把第一个参数作为脚本名。
3. 用 `exec bash` 运行同目录下的真正 hook 脚本。

### 5.3 为什么脚本叫 `session-start`，而不是 `session-start.sh`

这是为了绕开 Claude Code 在 Windows 上对 `.sh` 的自动检测。历史上 Claude Code 2.1.x 会看到命令里有 `.sh` 就自动前置 `bash`。这会把 polyglot wrapper 搞乱，导致类似下面的错误路线：

```text
bash "run-hook.cmd" session-start.sh
```

于是当前设计选择：

```text
run-hook.cmd session-start
```

也就是：

- 包装器保留 `.cmd`，方便 Windows CMD 运行。
- 真正脚本去掉 `.sh` 扩展名，避免自动检测干扰。
- 所有平台都通过同一入口传入脚本名。

---

## 六、`session-start`：真正的自举内容生成器

`hooks/session-start` 是核心逻辑。它负责读取技能、拼装上下文、输出 JSON。

### 6.1 初始化

脚本使用：

```bash
#!/usr/bin/env bash
set -euo pipefail
```

含义：

- 使用环境中的 Bash，而不是写死 `/bin/bash`。
- 开启严格模式，尽早暴露未定义变量、命令失败和管道失败。

随后通过 `$0` 计算脚本目录与插件根目录：

```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
```

这里不用 `${BASH_SOURCE[0]}`，是为了避免某些 `/bin/sh` 或 dash 环境下的 `Bad substitution` 问题。

### 6.2 旧技能目录提醒

脚本检查：

```bash
legacy_skills_dir="${HOME}/.config/superpowers/skills"
```

如果这个旧目录存在，就把迁移提醒加入注入上下文。提醒的核心意思是：

```text
Superpowers 现在使用 Claude Code 的 skills 系统。
~/.config/superpowers/skills 里的自定义技能不会被读取。
应迁移到 ~/.claude/skills。
```

这个提醒放在 hook 注入里很实用：它不要求用户主动看升级文档，而是在下一次会话启动时直接告诉 Agent，再由 Agent 在第一轮回复中提醒用户。

### 6.3 读取 `using-superpowers`

脚本读取：

```bash
${PLUGIN_ROOT}/skills/using-superpowers/SKILL.md
```

如果读取失败，内容会退化为：

```text
Error reading using-superpowers skill
```

这保证 hook 本身仍然能输出 JSON，不会因为文件缺失直接爆掉整个启动流程。

### 6.4 JSON 转义

`using-superpowers` 是 Markdown，里面可能包含：

- 反斜杠
- 双引号
- 换行
- 回车
- tab

这些字符必须转义后才能放进 JSON 字符串。脚本用 Bash 参数替换完成：

```bash
s="${s//\\/\\\\}"
s="${s//\"/\\\"}"
s="${s//$'\n'/\\n}"
s="${s//$'\r'/\\r}"
s="${s//$'\t'/\\t}"
```

这个实现比逐字符循环更快。变更记录里也提到过以前存在 `escape_for_json` 的 O(n^2) 性能问题，当前做法是更适合启动路径的实现。

### 6.5 构造注入内容

最终拼成的上下文外层是：

```text
<EXTREMELY_IMPORTANT>
You have superpowers.

Below is the full content of your 'superpowers:using-superpowers' skill...

...using-superpowers 完整内容...

...可选 legacy warning...
</EXTREMELY_IMPORTANT>
```

这段内容有几个设计意图：

1. **强优先级标记**：用 `<EXTREMELY_IMPORTANT>` 强调这是会话启动时的核心行为约束。
2. **明确 Agent 已拥有 Superpowers**：避免 Agent 以为还需要用户额外安装或说明。
3. **内嵌完整 `using-superpowers`**：不用再让 Agent 第一轮主动读取这个技能。
4. **提示其他技能仍通过 Skill 工具加载**：`using-superpowers` 是 bootstrap，其他技能保持按需加载。

---

## 七、输出格式适配

`session-start` 最后根据环境变量输出三种 JSON 之一。

### 7.1 Cursor

检测条件：

```bash
[ -n "${CURSOR_PLUGIN_ROOT:-}" ]
```

输出：

```json
{
  "additional_context": "..."
}
```

原因：Cursor hook 使用 snake_case 的 `additional_context`。

### 7.2 Claude Code

检测条件：

```bash
[ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]
```

输出：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "..."
  }
}
```

原因：Claude Code 的 hook 输出走嵌套的 `hookSpecificOutput.additionalContext`。

额外注意：脚本注释指出 Claude Code 会同时读取 `additional_context` 和 `hookSpecificOutput`，而且不会去重。因此不能为了“兼容”同时输出两个字段，否则可能重复注入。

### 7.3 Copilot CLI 或未知平台

兜底输出：

```json
{
  "additionalContext": "..."
}
```

原因：GitHub Copilot CLI v1.0.11+ 支持 SDK 标准的顶层 `additionalContext`。未知平台也优先走这个更通用的格式。

### 7.4 平台判断顺序

当前顺序是：

```text
CURSOR_PLUGIN_ROOT
  -> CLAUDE_PLUGIN_ROOT 且非 COPILOT_CLI
  -> Copilot CLI 或未知平台
```

这个顺序很重要。Cursor 可能也设置 Claude 相关变量，所以 Cursor 必须先判断。Copilot CLI 可能借用 Claude plugin root 语义，所以 Claude 分支必须排除 `COPILOT_CLI`。

---

## 八、端到端流程图

### 8.1 Claude Code

```text
Claude Code session starts
  -> reads hooks/hooks.json
  -> matches startup / clear / compact
  -> runs "${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd" session-start
  -> run-hook.cmd chooses Windows CMD path or Unix shell path
  -> hooks/session-start reads skills/using-superpowers/SKILL.md
  -> outputs hookSpecificOutput.additionalContext
  -> model receives Superpowers bootstrap before first response
```

### 8.2 Cursor

```text
Cursor plugin loads
  -> .cursor-plugin/plugin.json points to hooks/hooks-cursor.json
  -> sessionStart runs ./hooks/run-hook.cmd session-start
  -> hooks/session-start sees CURSOR_PLUGIN_ROOT
  -> outputs additional_context
  -> Cursor injects Superpowers bootstrap into session context
```

### 8.3 Copilot CLI

```text
Copilot CLI sessionStart
  -> hook reaches hooks/session-start
  -> COPILOT_CLI is set
  -> outputs top-level additionalContext
  -> Copilot receives the same using-superpowers bootstrap
```

### 8.4 OpenCode

OpenCode 不使用 `hooks/` 目录，而是在 `.opencode/plugins/superpowers.js` 中使用 JavaScript 插件 hook：

```text
OpenCode plugin loads
  -> config hook registers skills directory
  -> experimental.chat.messages.transform injects bootstrap into first user message
  -> module-level cache avoids every agent step repeated disk reads
```

它和 `hooks/session-start` 的目标相同：让 `using-superpowers` 在会话早期进入上下文。但实现方式不同：

- `hooks/` 路线依赖命令行 hook 输出 JSON。
- OpenCode 路线依赖 JS 插件 API 修改消息数组。

一个小的同步风险：`docs/README.opencode.md` 的说明仍写着 `experimental.chat.system.transform`，但当前代码实际使用的是 `experimental.chat.messages.transform`。如果维护 OpenCode 文档，应同步这个描述。

---

## 九、历史演进中解决的问题

从 `RELEASE-NOTES.md` 可以看出，hooks 这块不是一次写完的，而是在多个平台真实踩坑后逐步收敛出来的。

### 9.1 防止恢复会话重复注入

旧行为包含 `resume`。问题是恢复会话时，历史上下文里已经有 bootstrap，再注入一次会造成重复。

当前做法：

```text
startup|clear|compact
```

不再包含 `resume`。

### 9.2 同步执行确保第一轮可用

曾经 `SessionStart` 是异步执行。问题是第一轮模型响应可能比 hook 完成更早，导致 Agent 第一条回复还不知道 Superpowers。

当前做法：

```json
"async": false
```

这牺牲一点启动等待时间，换来第一轮行为稳定。

### 9.3 双引号修复 Windows/Linux 路径

单引号在 Windows CMD 中不是路径引用语法，在 Linux shell 中又会阻止变量展开。当前命令使用转义双引号：

```json
"\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start"
```

这能同时处理：

- Windows CMD
- Windows Git Bash
- macOS Bash
- Linux shell
- 路径中含空格

### 9.4 polyglot wrapper 恢复与扩展名规避

Windows 曾出现 `.sh` 自动检测与 polyglot wrapper 冲突的问题。当前组合是：

```text
run-hook.cmd + extensionless session-start
```

这样既保留 CMD 入口，又避免 `.sh` 被 Claude Code 自动前置 `bash`。

### 9.5 Cursor 单独格式

Cursor 加入后，需要：

- 新增 `hooks/hooks-cursor.json`
- 在 `.cursor-plugin/plugin.json` 中引用它
- 输出 `additional_context`
- 优先检测 `CURSOR_PLUGIN_ROOT`

### 9.6 Copilot CLI 顶层 `additionalContext`

Copilot CLI v1.0.11+ 支持 sessionStart hook 输出顶层 `additionalContext`。`session-start` 因此加入 `COPILOT_CLI` 检测，并把 Copilot 与未知平台放在 SDK 标准格式分支。

### 9.7 Bash 5.3 heredoc hang

历史上脚本使用 heredoc 输出大块 JSON，在 macOS Homebrew Bash 5.3+ 上遇到大变量展开挂起问题。

当前做法：用 `printf` 输出 JSON，避免 heredoc。

### 9.8 POSIX 与可移植性

相关改动包括：

- 用 `#!/usr/bin/env bash`，不要写死 `/bin/bash`。
- 用 `$0` 和 `dirname "$0"` 路径解析，减少 shell 差异问题。
- 尽量用 Bash 内建替换做 JSON escape，减少对 `sed`、`awk` 等外部工具的依赖。

---

## 十、维护 checklist

修改 hooks 时建议按这个顺序检查：

1. **先确认目标平台输出字段**
   - Claude Code：`hookSpecificOutput.additionalContext`
   - Cursor：`additional_context`
   - Copilot CLI/SDK：`additionalContext`

2. **不要同时输出多个上下文字段**
   - 尤其是 Claude Code 会读取多个字段且不去重。

3. **不要轻易把 `session-start` 改回 `.sh`**
   - 这会重新触发 Windows `.sh` 自动检测相关风险。

4. **不要把 `hooks.json` 命令改成单引号**
   - 单引号会破坏 Windows CMD，也会阻止 Unix 变量展开。

5. **保持 `run-hook.cmd` 可执行**
   - 当前 `run-hook.cmd` 与 `session-start` 都是 `rwxr-xr-x`。

6. **新增脚本时走 `run-hook.cmd <script-name>` 模式**
   - 不要让平台配置直接调用 Bash 脚本。

7. **大块 JSON 输出用 `printf`**
   - 避免 Bash heredoc 大变量展开问题。

8. **JSON 字符串必须转义**
   - Markdown 内容直接塞入 JSON 前必须处理反斜杠、双引号、换行、回车、tab。

9. **验证配置文件是合法 JSON**
   - 可用：

```bash
jq . hooks/hooks.json
jq . hooks/hooks-cursor.json
```

10. **如果改 OpenCode 注入机制，同步文档**
    - 当前代码是 `experimental.chat.messages.transform`。
    - 文档中仍有旧的 `experimental.chat.system.transform` 表述。

---

## 十一、这套 hooks 的架构价值

Superpowers 的核心内容是 `skills/`，但 hooks 让这些技能从“可被读取的文档”变成“会话一开始就能塑造 Agent 行为的系统”。

它的价值主要体现在：

1. **降低用户操作成本**：用户不需要每次提醒 Agent “请使用 Superpowers”。
2. **保证第一轮行为正确**：`async: false` 让 Agent 第一条回复前就拿到 bootstrap。
3. **平台差异被隔离**：Claude、Cursor、Copilot 的输出格式差异集中在 `session-start` 的最后几行。
4. **跨操作系统差异被隔离**：Windows CMD、Git Bash、Unix shell 的执行差异集中在 `run-hook.cmd`。
5. **技能内容保持平台无关**：`using-superpowers/SKILL.md` 不需要知道自己由哪个平台注入。
6. **升级提醒可以自然触达用户**：legacy skills 目录提醒通过会话启动上下文触发，不依赖用户阅读 release notes。

从架构上看，`hooks/` 是一个很小但很深的模块：接口只是几个 JSON 配置和一个命令入口，背后封装了平台 hook API、shell 差异、路径引用、JSON 转义、上下文字段协议和历史兼容性。

---

## 十二、速记

```text
hooks.json
  Claude Code SessionStart: startup | clear | compact
  command: "${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd" session-start
  async: false

hooks-cursor.json
  Cursor sessionStart
  command: ./hooks/run-hook.cmd session-start

run-hook.cmd
  Windows: find Git Bash or bash on PATH, then run hook script
  Unix: skip CMD block, exec bash hooks/<script-name>
  Missing bash on Windows: exit 0 silently

session-start
  reads skills/using-superpowers/SKILL.md
  adds legacy ~/.config/superpowers/skills warning when present
  escapes Markdown for JSON
  emits one platform-specific context field:
    Cursor -> additional_context
    Claude Code -> hookSpecificOutput.additionalContext
    Copilot/unknown -> additionalContext
```
