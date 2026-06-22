# Verification Before Completion Skill 笔记

> 原始技能：`skills/verification-before-completion/SKILL.md`  
> 作用：在声称完成、修复、通过、可合并之前，先运行能证明该声明的验证命令。

---

## 一、核心定位

`verification-before-completion` 是所有完成声明前的证据门槛。它反对“应该好了”“看起来通过了”“上次跑过了”这类没有新鲜证据的说法。

铁律：

```text
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

## 二、Gate Function

在任何成功声明之前必须按顺序做：

| 步骤 | 问题 |
|------|------|
| IDENTIFY | 哪条命令能证明这个声明？ |
| RUN | 现在运行完整命令 |
| READ | 读完整输出、退出码、失败数量 |
| VERIFY | 输出是否真的支持声明？ |
| CLAIM | 只有证据支持时才声明 |

如果输出不支持声明，就报告真实状态，而不是包装成成功。

## 三、不同声明需要的证据

| 声明 | 需要证据 | 不够充分的东西 |
|------|----------|----------------|
| 测试通过 | 测试命令输出 0 failures | 上次运行结果 |
| Linter 干净 | linter 输出 0 errors | 编译通过 |
| Build 成功 | build 命令 exit 0 | linter 通过 |
| Bug 修复 | 原始症状测试通过 | 代码改了 |
| 回归测试有效 | RED-GREEN 已验证 | 只看到测试通过 |
| Agent 完成 | 检查 diff 和验证命令 | Agent 自称成功 |
| 需求满足 | 对照需求逐项核查 | 测试通过 |

## 四、红旗

- 使用 “should/probably/seems”
- 在验证前说 “Done/Great/Perfect”
- 准备 commit、push 或开 PR 但没跑验证
- 信任子代理成功报告
- 只做了部分验证
- 太累了想直接结束

## 五、为什么重要

这个技能把“诚实”落到工程动作上：运行命令、读输出、再说结论。它防止 undefined function、缺需求、假通过等问题被带到用户面前。

## 六、与其他技能的关系

| Skill | 关系 |
|-------|------|
| `test-driven-development` | TDD 的绿色仍需最终验证 |
| `systematic-debugging` | 修复后证明原症状消失 |
| `requesting-code-review` | 评审前后都应验证 |
| `finishing-a-development-branch` | 收尾前强制跑测试 |

## 七、一句话记忆

先证据，后结论；没有刚跑完并读过的验证输出，就不要声称完成。
