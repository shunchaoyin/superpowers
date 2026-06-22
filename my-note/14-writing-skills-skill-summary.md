# Writing Skills Skill 笔记

> 原始技能：`skills/writing-skills/SKILL.md`  
> 作用：创建、修改或部署 skill 时，把 TDD 应用于流程文档本身。

---

## 一、核心定位

`writing-skills` 把 skill 看作会影响 Agent 行为的“代码”。因此写 skill 也必须先测试失败场景，再写最小 skill，再通过压力测试补漏洞。

核心原则：

```text
Writing skills IS Test-Driven Development applied to process documentation.
```

铁律：

```text
NO SKILL WITHOUT A FAILING TEST FIRST
```

新建 skill 和修改现有 skill 都适用。

## 二、什么是 Skill

Skill 是可复用的技术、模式、工具或参考指南。

Skill 不是：

- 一次性问题复盘
- 项目专用约定
- 已经能用自动化强制的机械规则
- 叙事式“我曾经怎么解决”

如果只是某项目专属，应写进项目指令文件，而不是核心 skill。

## 三、Skill TDD 映射

| TDD 概念 | Skill 创作 |
|----------|------------|
| Test case | 压力场景 |
| Production code | `SKILL.md` |
| RED | 没有 skill 时 Agent 违规 |
| GREEN | 有 skill 时 Agent 遵守 |
| Refactor | 发现新借口后补漏洞 |

关键是先观察 Agent 自然会怎么失败。没有 baseline failure，就不知道 skill 是否真的教会了正确行为。

## 四、何时创建 Skill

适合创建：

- 这个技巧不是显而易见的
- 你以后跨项目还会引用
- 模式广泛适用
- 其他人也会受益

不适合创建：

- 一次性解决方案
- 标准实践已有好文档
- 项目特定流程
- 可以用脚本或校验器强制的规则

## 五、SKILL.md 结构

基本结构：

```text
---
name: skill-name
description: Use when ...
---

# Skill Name

## Overview
## When to Use
## Core Pattern
## Quick Reference
## Implementation
## Common Mistakes
## Real-World Impact
```

Frontmatter 必须包含：

- `name`：只用字母、数字、连字符
- `description`：第三人称，只描述触发条件，不总结流程

## 六、Claude Search Optimization

描述字段是 skill 能否被正确发现的关键。

好描述：

- 以 `Use when...` 开头
- 描述触发条件、症状、上下文
- 包含用户和 Agent 可能搜索的关键词
- 不总结 workflow

坏描述：

- 过于抽象
- 第一人称
- 总结了流程，导致 Agent 只看 description 不读正文
- 绑定不必要的具体技术

## 七、不同 Skill 的测试方式

| Skill 类型 | 测试方式 | 成功标准 |
|------------|----------|----------|
| 纪律强制类 | 学术问题、压力场景、多重压力 | 高压下仍遵守规则 |
| 技术指南类 | 应用场景、变体、缺失信息测试 | 能在新场景正确应用 |
| 模式类 | 识别场景、应用场景、反例 | 知道何时用和何时不用 |
| 参考类 | 检索场景、应用场景、常见用例 | 能找到并正确使用信息 |

## 八、反合理化设计

纪律类 skill 要主动封堵 Agent 的借口：

- 明确无例外
- 写出常见 rationalization 表
- 添加 Red Flags
- 加上“违反字面规则就是违反精神”这类原则
- 把 baseline 测试中真实出现的借口写进文档

这不是修辞，而是行为塑形。

## 九、文件组织

| 类型 | 结构 |
|------|------|
| 自包含 skill | 只有 `SKILL.md` |
| 带工具 skill | `SKILL.md` + 可复用脚本 |
| 重参考 skill | `SKILL.md` + 大型参考文档 + scripts |

原则：主 skill 保持可读，重参考和可执行工具拆出去。

## 十、部署清单

每个 skill 都要按清单验证：

- RED：创建压力场景，无 skill 运行并记录失败
- GREEN：写最小 skill，确认 Agent 现在遵守
- REFACTOR：识别新借口，补反合理化内容，重新测试
- Quality：检查结构、示例、flowchart、quick reference、common mistakes
- Deployment：提交并推送，必要时考虑贡献回上游

技能明确禁止批量创建多个 skill 后统一测试。每个 skill 都要单独完成部署流程。

## 十一、与其他技能的关系

| Skill | 关系 |
|-------|------|
| `test-driven-development` | 必备背景，提供 RED-GREEN-REFACTOR 思想 |
| `verification-before-completion` | 部署前证明 skill 有效 |
| `requesting-code-review` | 重要 skill 修改可请求评审 |

## 十二、一句话记忆

Skill 是塑造 Agent 行为的代码；没做失败场景测试，就不要相信它能工作。
