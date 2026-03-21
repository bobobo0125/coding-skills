# Coding Skills - Claude Code 后端开发工具集

一套 Claude Code 专用的后端开发 Skills，包含需求分析、项目分析、代码开发、代码审查和文档同步五大核心能力。

## Skills 概览

| Skill | 用途 | 触发关键词 |
|-------|------|-----------|
| `prd-analysis` | 需求分析 - 将模糊需求转化为可测试的需求文档 | 需求分析、PRD、功能请求、问题陈述 |
| `project-analyzer` | 项目分析 - 分析现有项目的架构和技术栈 | 分析项目、审查代码、评估仓库 |
| `coding-skill` | 代码开发 - 基于阿里规范进行Java开发 | 编写代码、开发功能、编码规范 |
| `coding-review-skill` | 代码审查 - 在合并前验证代码质量 | 代码审查、请求审查、合并前检查 |
| `sync-doc` | 文档同步 - 读写飞书文档 | 飞书文档、云文档、docx链接 |

## 工作流程

```
┌─────────────┐    ┌───────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  需求分析    │ -> │   项目分析    │ -> │   代码开发   │ -> │   代码审查   │ -> │  文档同步    │
│ PRD Analysis│    │Project Analyzer│   │Coding Skill │    │Code Review  │    │  Sync Doc   │
└─────────────┘    └───────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### 1. 需求阶段 (PRD Analysis)

使用 `prd-analysis` skill 将模糊需求转化为经过验证的需求文档。

- 识别问题陈述
- 定义可测试的需求（R-ID格式）
- 盘点约束和假设
- 界定V1作用域
- 获得用户批准后进入下一阶段

### 2. 项目分析阶段 (Project Analyzer)

使用 `project-analyzer` skill 分析现有项目：

- 12步标准分析（基本信息、结构、技术栈、功能、架构等）
- 20步深度分析（源码、安全、性能等）
- 6步快速评估
- Mermaid图表可视化

### 3. 开发阶段 (Coding)

使用 `coding-skill` skill 进行Java开发：

- **MVC架构**：适用于业务逻辑简单的小型项目
- **DDD架构**：适用于业务复杂的中大型项目
- 遵循阿里巴巴Java开发规范

### 4. 审查阶段 (Code Review)

使用 `coding-review-skill` skill 进行代码审查：

- 调用 code-reviewer 子代理
- 问题分级：Critical / Important / Minor
- 修复后继续，确保代码质量

### 5. 文档同步阶段 (Sync Doc)

使用 `sync-doc` skill 操作飞书文档：

- 读取/写入飞书文档
- 创建/追加/更新内容
- 评论管理

## 项目结构

```
coding-skills/
├── CLAUDE.md                        # 主配置文件
├── README.md                        # 本文件
├── prd-analysis/                    # 需求分析技能
│   ├── SKILL.md
│   └── REFERENCE/
│       ├── task.md
│       ├── 新增需求技术方案.md
│       └── 修改需求技术方案.md
├── project-analyzer/                # 项目分析技能
│   ├── SKILL.md
│   └── REFERENCE/
│       ├── 工作流程指南.md
│       ├── 文档组织指南.md
│       ├── 路径指南.md
│       ├── 模板.md
│       ├── 示例工作流.md
│       └── 集成总结.md
├── coding-skill/                    # 代码开发技能
│   ├── SKILL.md
│   └── REFERENCE/
│       ├── MVC开发模板.md
│       ├── DDD开发模板.md
│       └── 阿里开发规范速查.md
├── coding-review-skill/             # 代码审查技能
│   ├── SKILL.md
│   └── REFERENCE/
│       └── code-reviewer.md
└── sync-doc/                        # 文档同步技能
    ├── SKILL.md
    └── REFERENCE/
        └── block-type.md
```

## 安装

将本项目克隆到 Claude Code 的 skills 目录：

```bash
# 克隆项目到 Claude Code skills 目录
git clone https://github.com/your-repo/coding-skills.git ~/.claude/skills/coding-skills
```

或使用符号链接：

```bash
# 或者创建符号链接（推荐，便于更新）
ln -s /path/to/coding-skills ~/.claude/skills/coding-skills
```

## 使用方式

通过 Task 工具调用对应 skill：

```bash
# 需求分析
Task --subagent_type prd-analysis --prompt "分析用户的需求..."

# 项目分析
Task --subagent_type project-analyzer --prompt "分析项目的架构..."

# 代码开发
Task --subagent_type coding-skill --prompt "实现用户认证功能..."

# 代码审查
Task --subagent_type coding-review-skill --prompt "审查刚完成的代码..."

# 文档同步
Task --subagent_type sync-doc --prompt "读取飞书文档内容..."
```

## 核心原则

1. **需求优先** - 任何开发工作开始前必须有经过验证的需求文档
2. **审查必要** - 所有代码在合并前必须经过审查
3. **规范遵守** - 开发过程必须遵循编码规范
4. **文档同步** - 重要的需求和技术文档应同步到飞书

## 禁止行为

- 跳过需求分析直接编码
- 跳过代码审查直接合并
- 忽略编码规范
- 在需求未验证时提出技术方案
- 使用未批准的方案进行开发
