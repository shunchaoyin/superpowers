# Superpowers 笔记索引

本目录是对 Superpowers 项目和 `skills/` 目录的中文笔记整理。项目核心不是传统代码模块，而是一组用于塑造编码 Agent 行为的工作流技能。

## 项目总览

- [项目架构概览](00-ARCHITECTURE_OVERVIEW.zh.md)

## Skill 笔记

| 序号 | Skill | 笔记 |
|------|-------|------|
| 01 | `using-superpowers` | [using-superpowers 技能深度解读](01-using-superpowers-SKILL_GUIDE.zh.md) |
| 02 | `brainstorming` | [Brainstorming Skill 文档总结](02-brainstorming-skill-summary.md) |
| 03 | `writing-plans` | [Writing-Plans Skill 文档总结](03-writing-plans-skill-summary.md) |
| 04 | `executing-plans` | [Executing-Plans Skill 笔记](04-executing-plans-skill-summary.md) |
| 05 | `subagent-driven-development` | [Subagent-Driven Development Skill 笔记](05-subagent-driven-development-skill-summary.md) |
| 06 | `dispatching-parallel-agents` | [Dispatching Parallel Agents Skill 笔记](06-dispatching-parallel-agents-skill-summary.md) |
| 07 | `test-driven-development` | [Test-Driven Development Skill 笔记](07-test-driven-development-skill-summary.md) |
| 08 | `systematic-debugging` | [Systematic Debugging Skill 笔记](08-systematic-debugging-skill-summary.md) |
| 09 | `verification-before-completion` | [Verification Before Completion Skill 笔记](09-verification-before-completion-skill-summary.md) |
| 10 | `requesting-code-review` | [Requesting Code Review Skill 笔记](10-requesting-code-review-skill-summary.md) |
| 11 | `receiving-code-review` | [Receiving Code Review Skill 笔记](11-receiving-code-review-skill-summary.md) |
| 12 | `using-git-worktrees` | [Using Git Worktrees Skill 笔记](12-using-git-worktrees-skill-summary.md) |
| 13 | `finishing-a-development-branch` | [Finishing a Development Branch Skill 笔记](13-finishing-a-development-branch-skill-summary.md) |
| 14 | `writing-skills` | [Writing Skills Skill 笔记](14-writing-skills-skill-summary.md) |

## 主工作流速记

Superpowers 的典型开发链路是：

```text
using-superpowers
  -> brainstorming
  -> using-git-worktrees
  -> writing-plans
  -> subagent-driven-development 或 executing-plans
  -> test-driven-development / requesting-code-review / verification-before-completion
  -> finishing-a-development-branch
```

调试链路通常是：

```text
systematic-debugging
  -> test-driven-development
  -> verification-before-completion
```

技能创作链路是：

```text
writing-skills
  -> test-driven-development 的 RED-GREEN-REFACTOR 思想
  -> 压力场景测试
  -> 部署前验证
```
