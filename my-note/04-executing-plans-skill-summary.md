# Executing-Plans Skill 笔记

> 原始技能：`skills/executing-plans/SKILL.md`  
> 作用：在已有实施计划的情况下，按计划逐项执行，并在完成后进入分支收尾流程。

---

## 一、核心定位

`executing-plans` 是没有或不使用子代理时的计划执行方式。它要求先读取并批判性审查计划，再逐项执行任务、运行验证，最后调用 `finishing-a-development-branch` 完成收尾。

它不是用来“边想边做”的技能。前置条件是已经有一份书面 implementation plan。

## 二、何时使用

适用场景：

- 已有明确的实施计划文件
- 需要在当前会话或另一个会话中按计划执行
- 没有可用子代理，或用户选择 inline execution

如果平台支持子代理，技能明确建议改用 `subagent-driven-development`，因为每个任务有独立上下文和评审门槛，质量更高。

## 三、流程

| 阶段 | 要求 |
|------|------|
| Step 1 | 读取计划文件 |
| Step 2 | 批判性审查计划，发现疑问先问用户 |
| Step 3 | 无疑问时创建任务清单并开始执行 |
| Step 4 | 每个任务按计划步骤执行，运行对应验证 |
| Step 5 | 所有任务完成后调用 `finishing-a-development-branch` |

执行任务时，每项都要经历：

```text
标记 in_progress -> 按计划执行步骤 -> 运行验证 -> 标记 completed
```

## 四、停止条件

遇到以下情况必须停下，而不是猜：

- 缺依赖或环境不可用
- 测试失败且无法按计划解决
- 计划指令不清楚
- 不理解某一步
- 验证反复失败

这个技能强调“不要强行穿过 blocker”。计划不清楚时继续写代码，只会把错误固化。

## 五、与其他技能的关系

| 关联 Skill | 关系 |
|------------|------|
| `using-git-worktrees` | 执行计划前应确保隔离工作区 |
| `writing-plans` | 本技能执行它产出的计划 |
| `finishing-a-development-branch` | 所有任务完成后的强制收尾技能 |
| `subagent-driven-development` | 子代理可用时的推荐替代方案 |

## 六、红旗

- 没读完整计划就开始执行
- 计划有疑问但自行补完
- 跳过验证步骤
- 在 `main` 或 `master` 上直接开始实现，且没有用户明确同意
- 完成任务后不进入分支收尾流程

## 七、一句话记忆

`executing-plans` 的价值是把“计划”变成“有验证的连续执行”，但它不替代设计、计划或收尾。
