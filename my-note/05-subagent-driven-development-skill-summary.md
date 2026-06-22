# Subagent-Driven Development Skill 笔记

> 原始技能：`skills/subagent-driven-development/SKILL.md`  
> 作用：按实施计划执行任务，每个任务派发独立子代理，并经过规格符合性和代码质量两轮评审。

---

## 一、核心定位

`subagent-driven-development` 是 Superpowers 推荐的主执行引擎。它把主代理变成协调者，把每个实施任务交给一个新鲜上下文的子代理，避免主会话上下文污染，并在每个任务后设置两道质量门槛。

核心原则：

```text
每任务一个新子代理 + 先规格评审 + 再代码质量评审
```

## 二、何时使用

适用场景：

- 已有 implementation plan
- 任务大多可以按顺序独立完成
- 当前平台支持子代理
- 用户希望在同一会话中持续推进，不频繁人工检查

不适用场景：

- 没有计划，需要先 brainstorm 或写 plan
- 任务高度耦合，无法拆成独立小任务
- 需要在另一个并行会话执行，可能用 `executing-plans`

## 三、流程

整体流程：

```text
读取计划并提取所有任务
  -> 为每个任务派发 implementer
  -> implementer 完成、测试、提交、自审
  -> spec reviewer 检查是否符合计划/规格
  -> code quality reviewer 检查实现质量
  -> 两轮评审都通过后标记任务完成
  -> 全部任务结束后做最终代码评审
  -> 调用 finishing-a-development-branch
```

关键点：不能让子代理自己去读完整计划。主代理应该把当前任务的完整文本和必要上下文提供给它。

## 四、子代理状态处理

| 状态 | 处理方式 |
|------|----------|
| `DONE` | 进入规格评审 |
| `DONE_WITH_CONCERNS` | 先读 concerns，必要时解决后再评审 |
| `NEEDS_CONTEXT` | 补充上下文后重新派发 |
| `BLOCKED` | 判断是上下文不足、模型能力不足、任务过大，还是计划错误 |

不要忽略子代理的升级信号。它说卡住了，就说明需要改变上下文、模型、任务粒度或计划。

## 五、两阶段评审

| 阶段 | 目的 | 通过条件 |
|------|------|----------|
| 规格符合性评审 | 检查是否完全实现计划，且没有额外功能 | 无遗漏、无多做 |
| 代码质量评审 | 检查实现是否健壮、清晰、可维护 | 无 Critical/Important 问题 |

顺序不能反。必须先确认“做对了事”，再检查“事做得好不好”。

## 六、提示模板

目录下有三个模板：

- `implementer-prompt.md`：实现子代理
- `spec-reviewer-prompt.md`：规格符合性评审
- `code-quality-reviewer-prompt.md`：代码质量评审

这些模板确保不同角色只拿到需要的上下文。

## 七、红旗

- 在 `main` 或 `master` 上开始实现，且无明确许可
- 并行派发多个实现子代理，导致文件冲突
- 跳过规格评审或代码质量评审
- 规格评审没通过就开始代码质量评审
- reviewer 发现问题后不重新评审
- 接受“差不多符合计划”
- 子代理失败后主代理直接手动修，污染上下文

## 八、与其他技能的关系

| Skill | 关系 |
|-------|------|
| `using-git-worktrees` | 执行前需要隔离工作区 |
| `writing-plans` | 提供可执行计划 |
| `test-driven-development` | 子代理执行任务时应遵循 |
| `requesting-code-review` | 代码评审模板来源 |
| `finishing-a-development-branch` | 全部任务完成后的收尾 |

## 九、一句话记忆

`subagent-driven-development` 把实现拆成“独立执行 + 严格验收”的流水线，主代理负责协调和质量门槛。
