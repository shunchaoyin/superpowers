# Requesting Code Review Skill 笔记

> 原始技能：`skills/requesting-code-review/SKILL.md`  
> 作用：完成任务、重大功能或合并前，派发代码评审子代理提前发现问题。

---

## 一、核心定位

`requesting-code-review` 是主动请评审的技能。它通过子代理拿到一个隔离视角，让评审者只看到工作成果、需求和 diff，而不是主会话的思考过程。

核心原则：

```text
Review early, review often.
```

## 二、何时使用

必须使用：

- subagent-driven development 中每个任务后
- 完成重大功能后
- 合并到主分支前

可选但有价值：

- 卡住时获得新视角
- 重构前建立基线检查
- 修复杂 Bug 后复查

## 三、请求评审的方式

基本步骤：

1. 获取 base 和 head 的 git SHA
2. 使用 `requesting-code-review/code-reviewer.md` 模板派发评审子代理
3. 提供 `{DESCRIPTION}`、`{PLAN_OR_REQUIREMENTS}`、`{BASE_SHA}`、`{HEAD_SHA}`
4. 根据评审反馈处理问题

评审重点是 diff 与需求是否匹配，而不是让 reviewer 继承主会话上下文。

## 四、反馈处理优先级

| 等级 | 处理方式 |
|------|----------|
| Critical | 立即修复，不能继续 |
| Important | 前进前修复 |
| Minor | 可记录稍后处理 |
| 错误反馈 | 用技术证据反驳或澄清 |

## 五、与工作流集成

| 工作流 | 用法 |
|--------|------|
| Subagent-Driven Development | 每个任务后评审，防止问题累积 |
| Executing Plans | 每个任务后或自然检查点评审 |
| Ad-Hoc Development | 合并前或卡住时评审 |

## 六、红旗

- 因为“很简单”跳过评审
- 忽略 Critical 问题
- 带着未修复 Important 问题继续推进
- 对正确技术反馈争辩
- 不提供 plan 或需求，导致 reviewer 无法判断对错

## 七、与 `receiving-code-review` 的关系

`requesting-code-review` 负责“发起评审”；收到评审后，应该用 `receiving-code-review` 的态度处理：先理解和验证，再实现或技术性反驳。

## 八、一句话记忆

代码评审不是最后的礼节，而是每个任务之间的质量刹车。
