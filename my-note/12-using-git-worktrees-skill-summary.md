# Using Git Worktrees Skill 笔记

> 原始技能：`skills/using-git-worktrees/SKILL.md`  
> 作用：开始功能开发或执行计划前，确保工作发生在隔离工作区。

---

## 一、核心定位

`using-git-worktrees` 用来保护当前分支和用户现有改动。它优先检测现有隔离环境，再使用平台原生 worktree 工具，最后才回退到手动 `git worktree`。

核心原则：

```text
Detect existing isolation first. Then use native tools. Then fall back to git.
```

## 二、Step 0：先检测

创建任何东西之前，先检查：

- `git rev-parse --git-dir`
- `git rev-parse --git-common-dir`
- 当前分支名
- 是否处于 submodule

如果 `GIT_DIR != GIT_COMMON` 且不是 submodule，说明已经在 linked worktree 中，不要再创建嵌套 worktree。

## 三、是否需要征得同意

如果当前是普通仓库 checkout，且用户没有预先声明 worktree 偏好，应询问：

```text
Would you like me to set up an isolated worktree? It protects your current branch from changes.
```

用户拒绝时，在当前目录继续，并跳到项目设置和基线验证。

## 四、创建机制优先级

| 优先级 | 机制 | 说明 |
|--------|------|------|
| 1 | 平台原生 worktree 工具 | 首选，避免和 harness 状态打架 |
| 2 | 手动 `git worktree add` | 仅在没有原生工具时使用 |

不能在有原生工具时直接调用 `git worktree add`，否则可能造成平台不可见的 phantom state。

## 五、目录选择

手动 git worktree 时按优先级选择目录：

1. 用户或项目指令指定的位置
2. 项目内已有 `.worktrees/`
3. 项目内已有 `worktrees/`
4. 旧版全局路径 `~/.config/superpowers/worktrees/<project>`
5. 默认 `.worktrees/`

项目内目录必须先确认被 `.gitignore` 忽略。否则应添加忽略规则并提交，再创建 worktree。

## 六、项目设置和基线验证

进入隔离工作区后自动检测项目类型：

- `package.json` -> `npm install`
- `Cargo.toml` -> `cargo build`
- `requirements.txt` -> `pip install -r requirements.txt`
- `pyproject.toml` -> `poetry install`
- `go.mod` -> `go mod download`

之后运行项目测试，确认 clean baseline。基线失败时要报告并询问是否继续或先调查。

## 七、红旗

- 不检测现有 worktree 就创建新 worktree
- 在 submodule 中误判为 worktree
- 有平台原生工具却手动 `git worktree add`
- 创建项目内 worktree 前不检查 ignore
- 跳过依赖安装或基线测试
- 基线失败仍静默继续

## 八、与其他技能的关系

| Skill | 关系 |
|-------|------|
| `brainstorming` | 设计批准后进入实现前应隔离工作区 |
| `writing-plans` | 计划执行时提到隔离上下文 |
| `subagent-driven-development` | 执行计划前需要隔离 |
| `executing-plans` | 执行计划前需要隔离 |
| `finishing-a-development-branch` | 收尾时根据 worktree provenance 决定是否清理 |

## 九、一句话记忆

先判断自己是不是已经在隔离区，再创建；不要和 harness 抢 worktree 管理权。
