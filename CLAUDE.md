# 后端开发 Skills 使用指南

本文档定义了 Claude Code 在进行后端开发时如何使用项目中的 Skills。

## Skills 概览

| Skill | 用途 | 触发关键词 |
|-------|------|-----------|
| `prd-analysis` | 需求分析 - 将模糊需求转化为可测试的需求文档 | 需求分析、PRD、功能请求、问题陈述 |
| `design-solution` | 技术方案设计 - 将需求文档转化为可执行的技术方案 | 技术方案、设计方案、技术选型、架构设计 |
| `project-analyzer` | 项目分析 - 分析现有项目的架构和技术栈 | 分析项目、审查代码、评估仓库 |
| `coding-skill` | 代码开发 - 基于阿里规范进行Java开发 | 编写代码、开发功能、编码规范 |
| `coding-review-skill` | 代码审查 - 在合并前验证代码质量 | 代码审查、请求审查、合并前检查 |
| `sync-doc` | 文档同步 - 读写飞书文档 | 飞书文档、云文档、docx链接 |

## 强制工作流程

### 1. 需求阶段 (PRD Analysis)

**必须使用 `prd-analysis` skill 的场景：**

- 用户提出新的功能需求
- 用户描述一个问题或痛点
- 用户说"我想构建X"、"需要添加Y功能"
- 存在模糊或不完整的需求描述

**执行步骤：**

1. 使用 `prd-analysis` skill 进行需求分析
2. 生成需求文档并获得用户批准
3. 只有在需求验证通过后才能进入设计阶段

**硬约束：**
- 需求验证通过前禁止进行任何设计或编码
- 禁止跳过需求分析直接询问"如何实现"

### 2. 设计阶段 (Design Solution)

**必须使用 `design-solution` skill 的场景：**

- 需求已通过验证，需要设计技术实现方案
- 需要进行架构设计、技术选型
- 需要定义接口、数据模型、业务流程

**执行步骤：**

1. 使用 `design-solution` skill 进行技术方案设计
2. 生成技术方案文档并获得团队评审确认
3. 只有在技术方案评审通过后才能进入开发阶段

**硬约束：**
- 设计必须可追溯到需求（每个设计决策对应需求 ID）
- 技术选型必须有明确理由
- 方案必须经过评审才能进入开发

### 3. 项目分析阶段 (Project Analyzer)

**可选使用 `project-analyzer` skill 的场景：**

- 需要理解现有项目的架构
- 需要评估技术栈和依赖
- 需要审查现有代码结构

**执行步骤：**
1. 使用 `project-analyzer` skill 进行项目分析
2. 理解项目结构和约束后再进行开发

### 3. 开发阶段 (Coding)

**必须使用 `coding-skill` skill 的场景：**

- 编写新的 Java 代码
- 修改现有代码
- 需要遵循编码规范
- 构建项目结构

**执行步骤：**

1. 根据项目架构选择模板（MVC 或 DDD）
   - 业务逻辑简单 → MVC
   - 业务复杂、需要领域建模 → DDD
2. 参考 `REFERENCE/` 目录下的模板进行开发
3. 遵循阿里开发规范

### 4. 审查阶段 (Code Review)

**必须使用 `coding-review-skill` skill 的场景：**

- 完成一个功能模块后
- 合并到主分支前
- 任何重要代码变更完成后

**执行步骤：**

1. 获取 git SHA（BASE_SHA 和 HEAD_SHA）
2. 调用 `code-reviewer` 子代理进行审查
3. 根据反馈修复问题
   - Critical: 立即修复
   - Important: 继续前修复
   - Minor: 记录后续处理

**硬约束：**
- 禁止因为"很简单"而跳过审查
- 禁止在 Critical 问题未修复时继续
- 禁止在 Important 问题未修复时合并

### 5. 文档同步阶段 (Sync Doc)

**可选使用 `sync-doc` skill 的场景：**

- 用户提供飞书文档链接
- 需要将需求/技术文档同步到飞书
- 需要读取飞书文档内容

## Skills 调用方式

```bash
# 使用 Task 工具调用对应 skill
Task --subagent_type <skill-name> --prompt <task-description>
```

例如：
```bash
# 需求分析
Task --subagent_type prd-analysis --prompt "分析用户的需求..."

# 技术方案设计
Task --subagent_type design-solution --prompt "基于需求文档设计技术方案..."

# 代码开发
Task --subagent_type coding-skill --prompt "实现用户认证功能..."

# 代码审查
Task --subagent_type coding-review-skill --prompt "审查刚完成的代码..."
```

## 优先级规则

1. **需求优先**：任何开发工作开始前必须有经过验证的需求文档
2. **审查必要**：所有代码在合并前必须经过审查
3. **规范遵守**：开发过程必须遵循 `coding-skill` 中的编码规范
4. **文档同步**：重要的需求和技术文档应同步到飞书

## 禁止行为

- ❌ 跳过需求分析直接编码
- ❌ 跳过技术方案设计直接编码
- ❌ 跳过代码审查直接合并
- ❌ 忽略编码规范
- ❌ 在需求未验证时提出技术方案
- ❌ 在方案未评审时进行开发
