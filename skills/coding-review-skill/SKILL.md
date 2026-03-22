---
name: requesting-code-review
description: 在完成任务、实现主要功能或合并前使用，以验证工作是否符合要求
---

# 请求代码审查

调用 superpowers:code-reviewer 子代理来捕获问题，防止问题蔓延。审查者获得精确构建的评估上下文——而不是你的会话历史。这让审查者专注于工作成果，而不是你的思维过程，并保留你自己的上下文以继续工作。

**核心原则：** 早审查，常审查。

## 何时请求审查

**必须审查：**
- 子代理驱动开发中的每个任务之后
- 完成主要功能之后
- 合并到主分支之前

**可选但有价值：**
- 遇到困难时（新鲜视角）
- 重构之前（基线检查）
- 修复复杂bug之后

## 如何请求审查

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 调用 code-reviewer 子代理：**

使用 Task 工具，类型为 superpowers:code-reviewer，填写 `code-reviewer.md` 中的模板

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 你刚构建的内容
- `{PLAN_OR_REQUIREMENTS}` - 它应该做什么
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 结束提交
- `{DESCRIPTION}` - 简要描述

**3. 处理反馈：**
- 立即修复 Critical 问题
- 继续之前修复 Important 问题
- 记录 Minor 问题以便后续处理
- 如果审查者错了，反驳并说明理由

## 示例

```
[刚完成 Task 2：添加验证函数]

你：让我在继续之前请求代码审查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[调用 superpowers:code-reviewer 子代理]
  WHAT_WAS_IMPLEMENTED: 对话索引的验证和修复函数
  PLAN_OR_REQUIREMENTS: 来自 docs/superpowers/plans/deployment_plan.md 的 Task 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 添加了 verifyIndex() 和 repairIndex()，包含4种问题类型

[子代理返回]：
  优点：清晰的架构，真实的测试
  问题：
    Important: 缺少进度指示器
    Minor: 报告间隔的魔法数字 (100)
  评估：可以继续

你：[修复进度指示器]
[继续 Task 3]
```

## 与工作流程的集成

**子代理驱动开发：**
- 每个任务后都审查
- 在问题积累之前捕获
- 修复后再进行下一个任务

**执行计划：**
- 每批任务（3个）后审查
- 获取反馈，应用，继续

**临时开发：**
- 合并前审查
- 遇到困难时审查

## 危险信号

**永远不要：**
- 因为"很简单"而跳过审查
- 忽略 Critical 问题
- 在 Important 问题未修复的情况下继续
- 对有效的技术反馈进行争论

**如果审查者错了：**
- 用技术理由反驳
- 展示证明它工作的代码/测试
- 请求澄清

请参阅模板：REFERENCE/code-reviewer.md
