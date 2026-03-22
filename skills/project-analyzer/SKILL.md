---
name: project-analyzer
description: 使用结构化模板全面分析开源项目。用于分析开源项目、审查代码架构或评估GitHub仓库。
license: MIT
metadata:
  author: https://github.com/mjczz
  version: "1.0.0"
  domain: language
  triggers: 项目分析
  role: specialist
  scope: implementation
  related-skills: github, pretty-mermaid, coding-router
---

# 项目分析器

使用结构化报告和可视化图表系统化分析项目的方法。

## 何时使用此技能

当用户需要时使用：

- "分析"、"审查"或"评估"某个项目
- 想要了解代码库的架构
- 需要详细评估项目（本地或远程）
- 请求项目报告或摘要
- 提及"我想分析 [项目名/路径]"
- 请求关于特定项目的建议

## 支持的项目类型

**本地项目**（主要）：
- 本地目录路径：`~/work/my-project`
- 当前目录：`.`
- 相对路径：`./my-project`
- 绝对路径：`/Users/ccc/work/todo/kubernetes`

**远程项目**（可选）：
- GitHub仓库：`owner/repo`
- Git URL：`https://github.com/owner/repo`

## 分析流程概述

分析遵循**12步顺序流程**：

1. **项目基本信息** - 基本元数据（语言、文件数量、结构）
2. **项目结构** - 目录结构和模块关系
3. **技术栈** - 依赖项和框架
4. **核心功能** - 关键功能（带时序图）
5. **架构设计** - 架构模式（带图表）
6. **代码质量** - 代码风格、测试、复杂度
7. **文档质量** - README、API文档、指南
8. **项目活跃度** - 提交、issue、PR（从git获取）
9. **优缺点** - 优点和弱点
10. **使用场景** - 何时使用/不使用
11. **学习价值** - 值得学习的内容
12. **总结** - 最终结论

## 深度分析模式

对于复杂或技术性项目，启用**深度分析模式**，增加以下内容：

13. **源代码深入分析** - 关键代码路径、函数调用链
14. **实现机制** - 内部机制和数据流
15. **组件分析** - 关键组件深入分析
16. **协议与接口分析** - API契约和协议
17. **工作流追踪** - 端到端流程分析
18. **安全分析** - 安全机制和漏洞
19. **性能分析** - 性能瓶颈和优化
20. **测试策略分析** - 测试方法和覆盖率

## 分析步骤

### 步骤0：准备

1. 读取模板文件
2. 在当前工程目录下创建 `doc/` 目录
3. 创建分析任务清单

### 步骤1-12：顺序分析

对每个主题依次执行：

1. 开始分析时告知用户当前进度
2. 分析该主题（收集信息，根据需要创建图表）
3. 为每个主题创建独立文档
4. 更新主分析文件
5. 完成后向用户报告关键发现
6. 自动继续下一个主题

### 最终步骤：完成

1. 展示总结和关键洞察
2. 显示文件位置
3. 询问是否需要深入了解特定领域

## 信息收集策略

### 本地项目

**基本信息**：
```bash
ls -la [project-path]
find [project-path] -type f | wc -l
find [project-path] -name "README*" -o -name "readme*"

# 语言检测
find [project-path] -name "*.go" | wc -l
find [project-path] -name "*.js" | wc -l
find [project-path] -name "*.py" | wc -l
```

**项目结构**：
```bash
tree -L 3 [project-path]
find [project-path] -type d | head -20
```

**技术栈**：
```bash
cat [project-path]/package.json
cat [project-path]/go.mod
cat [project-path]/requirements.txt
cat [project-path]/Cargo.toml
```

### 远程项目

```bash
gh api repos/owner/repo
gh api repos/owner/repo/git/trees/main?recursive=1
gh api repos/owner/repo/contents/package.json
```

## Mermaid图表指南

| 主题 | 图表类型 | 使用场景 |
|------|---------|---------|
| 项目结构 | 模块图 | 始终使用 - 展示依赖关系 |
| 技术栈 | 依赖图 | 始终使用 - 展示技术栈层次 |
| 核心功能 | 时序图 | 用户流程清晰时使用 |
| 架构设计 | 架构流程图 | 始终使用 - 展示分层 |
| 总结 | 状态图 | FSM/基于状态的项目 |
| 总结 | ER图 | 数据库密集型项目 |

## 分析模式

### 标准分析模式（12步）
用于：
- "analyze [project]"
- "review [project]"
- "evaluate [project]"
- 一般项目理解

### 深度分析模式（20步）
用于：
- "deep dive into [project]"
- "source code analysis of [project]"
- "how does [project] work internally"
- 技术架构评估
- 性能/安全分析需求

### 快速评估模式（6步）
用于：
- "quick overview of [project]"
- "brief analysis of [project]"
- "should I use [project]"

## 重要说明

- 始终完成所有12个主题，除非用户说"停止"
- 每个主题开始时报告进度
- 每个主题完成后立即报告
- 自动继续下一个主题，无需用户确认
- 使用Mermaid图表增强表达
- 提供具体细节，避免泛泛而谈
- 注明信息来源（GitHub、文档等）

## 输出示例

用户说"分析 /Users/ccc/work/todo/kubernetes"：

```
开始分析 /Users/ccc/work/todo/kubernetes 项目...

🔵 项目基本信息 (进度 1/12)
分析范围：项目元数据、语言统计、文件结构概览
🔄 开始分析...

✅ 项目基本信息 (进度 1/12)
- 主要语言：Go (95%+)
- 总文件数：50,000+
- 项目路径：/Users/ccc/work/todo/kubernetes

继续下一个主题...

🔵 项目结构 (进度 2/12)
分析范围：目录组织、模块关系、组件布局
🔄 开始分析...

✅ 项目结构 (进度 2/12)
- 主要目录：cmd/, pkg/, staging/
- 核心组件：kube-apiserver, kubelet, kube-proxy

继续下一个主题...
[... 继续完成所有12个主题 ...]

✅ 分析完成！
分析文档保存位置：/Users/ccc/work/todo/kubernetes/doc/kubernetes-analysis.md

是否需要深入了解某个特定领域？
```
