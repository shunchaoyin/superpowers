# Receiving Code Review Skill 笔记

> 原始技能：`skills/receiving-code-review/SKILL.md`  
> 作用：收到代码评审反馈时，先理解和验证，再逐项实现或技术性反驳。

---

## 一、核心定位

`receiving-code-review` 强调技术判断，而不是表演式认同。它要求 Agent 不要因为 reviewer 提了建议就立刻实现，也不要用“你完全正确”这类话掩盖自己还没验证。

核心原则：

```text
Verify before implementing. Ask before assuming. Technical correctness over social comfort.
```

## 二、响应模式

收到反馈后按以下顺序：

| 步骤 | 内容 |
|------|------|
| READ | 完整阅读反馈 |
| UNDERSTAND | 用自己的话复述要求，或提问 |
| VERIFY | 对照代码库事实检查 |
| EVALUATE | 判断对当前代码库是否技术正确 |
| RESPOND | 技术确认或有理由地反驳 |
| IMPLEMENT | 一次处理一项，并测试 |

## 三、禁止的回应

不要写：

- “You're absolutely right!”
- “Great point!”
- “Excellent feedback!”
- “Let me implement that now.”（尚未验证）
- 各种感谢式、表演式认同

应该改为：

- 复述技术需求
- 问澄清问题
- 给出技术推理
- 直接修复并说明改了什么

## 四、处理不清楚的反馈

如果多项反馈中有任何项不清楚，应先停下询问。不要先实现自己懂的部分，因为反馈之间可能相关，部分理解容易导致错误实现。

## 五、外部 reviewer 的反馈要审查

外部反馈是建议，不是命令。实现前需要检查：

- 是否适合当前代码库
- 是否会破坏现有功能
- 当前实现是否有历史或兼容原因
- 是否适用于所有平台/版本
- reviewer 是否掌握完整上下文

如果建议违反 YAGNI，应先搜索实际用法。没人调用的“专业功能”可能应该删除，而不是补全。

## 六、何时反驳

应该技术性反驳的情况：

- 建议会破坏现有功能
- reviewer 缺上下文
- 建议实现未使用功能
- 对当前技术栈不正确
- 与用户已定的架构决策冲突

反驳方式应引用代码、测试或兼容性事实，而不是防御性语气。

## 七、实现顺序

多项反馈按顺序处理：

```text
先澄清所有不清楚项
  -> Blocking/security/破坏性问题
  -> 简单修复
  -> 复杂重构或逻辑修复
  -> 每项单独测试
  -> 最终验证无回归
```

## 八、一句话记忆

收到评审先做工程判断，不做情绪表演；对的修，错的用证据说明。
