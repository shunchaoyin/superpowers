# Finishing a Development Branch Skill 笔记

> 原始技能：`skills/finishing-a-development-branch/SKILL.md`  
> 作用：实现完成且测试通过后，引导用户选择合并、开 PR、保留或丢弃，并安全清理工作区。

---

## 一、核心定位

`finishing-a-development-branch` 是开发结束时的收尾技能。它不是简单说“完成了”，而是先验证测试，再检测 Git 环境，最后给用户结构化选项。

核心原则：

```text
Verify tests -> Detect environment -> Present options -> Execute choice -> Clean up.
```

## 二、Step 1：先验证测试

在展示选项前必须运行项目测试。测试失败时停止，不能进入合并或 PR 流程。

如果失败，应报告失败数量和摘要，并说明必须先修复。

## 三、Step 2：检测环境

通过 `git-dir` 和 `git-common-dir` 判断当前状态：

| 状态 | 菜单 | 清理方式 |
|------|------|----------|
| 普通 repo | 4 个选项 | 无 worktree 清理 |
| linked worktree + 命名分支 | 4 个选项 | 按 provenance 清理 |
| linked worktree + detached HEAD | 3 个选项 | 通常不清理，外部管理 |

这一步决定是否可以本地 merge，以及是否能删除 worktree。

## 四、用户选项

普通 repo 或命名分支 worktree 展示：

```text
1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is
4. Discard this work
```

detached HEAD 展示：

```text
1. Push as new branch and create a Pull Request
2. Keep as-is
3. Discard this work
```

技能要求菜单保持简洁，不要加多余解释。

## 五、各选项处理

| 选项 | 行为 | worktree |
|------|------|----------|
| Merge locally | 切回 base，pull，merge，合并后再测 | 成功后清理 |
| Push and PR | push 分支并创建 PR | 保留，便于继续迭代 |
| Keep as-is | 报告分支和路径 | 保留 |
| Discard | 要求用户输入 `discard` 确认 | 确认后删除 |

丢弃是破坏性操作，必须列出会删除的分支、提交和路径，并等待精确确认。

## 六、清理规则

只在 Option 1 和 Option 4 清理 worktree。

只有以下位置的 worktree 视为 Superpowers 拥有，可清理：

- `.worktrees/`
- `worktrees/`
- `~/.config/superpowers/worktrees/`

其他位置可能由 harness 管理，不要删除。

## 七、红旗

- 测试失败还展示合并/PR 选项
- 合并后不重新运行测试
- 未确认就丢弃工作
- 创建 PR 后清理 worktree
- 删除分支前没成功 merge
- 从待删除 worktree 内运行 `git worktree remove`
- 清理不是自己创建的 worktree

## 八、与其他技能的关系

| Skill | 关系 |
|-------|------|
| `executing-plans` | 全部任务完成后强制调用 |
| `subagent-driven-development` | 最终评审后调用 |
| `verification-before-completion` | 测试验证体现同一原则 |
| `using-git-worktrees` | 收尾清理依赖其 worktree provenance |

## 九、一句话记忆

收尾不是一句“完成了”，而是测试证据、Git 环境判断和用户明确选择。
