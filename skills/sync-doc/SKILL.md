---
name: feishu-doc
description: |
  飞书文档读写操作 + 评论管理。当用户提及飞书文档、云文档、docx链接或文档评论时激活。
---

# 飞书文档工具

单一工具 `feishu_doc`，通过 action 参数执行所有文档操作，包括评论管理。

## Token 提取

从 URL `https://xxx.feishu.cn/docx/ABC123def` → `doc_token` = `ABC123def`
从 URL `https://xxx.feishu.cn/docs/doccn123c` → `doc_token` = `doccn123c`

## 操作

### 读取文档

```json
{ "action": "read", "doc_token": "ABC123def" }
```

返回：标题、纯文本内容、块统计信息。检查 `hint` 字段 - 如果存在，表示有结构化内容（表格、图片），需要使用 `list_blocks` 获取。

### 写入文档（替换全部）

```json
{ "action": "write", "doc_token": "ABC123def", "content": "# 标题\n\nMarkdown内容..." }
```

用 markdown 内容替换整个文档。支持：标题、列表、代码块、引用、链接、图片（`![](url)` 自动上传）、粗体/斜体/删除线。

**限制：** 不支持 Markdown 表格。

### 创建 + 写入（原子操作，推荐）

```json
{
  "action": "create_and_write",
  "title": "新文档",
  "content": "# 标题\n\nMarkdown内容..."
}
```

指定文件夹：
```json
{
  "action": "create_and_write",
  "title": "新文档",
  "content": "# 标题\n\nMarkdown内容...",
  "folder_token": "fldcnXXX"
}
```

在一次调用中创建文档并写入内容。优先使用此方式，而不是分开的 `create` + `write`。

### 追加内容

```json
{ "action": "append", "doc_token": "ABC123def", "content": "追加内容" }
```

在文档末尾追加 markdown 内容。

### 创建文档

```json
{ "action": "create", "title": "新文档" }
```

指定文件夹：
```json
{ "action": "create", "title": "新文档", "folder_token": "fldcnXXX" }
```

创建一个空文档（仅有标题）。

### 获取块列表

```json
{ "action": "list_blocks", "doc_token": "ABC123def" }
```

返回完整的块数据，包括表格、图片。使用此方法读取结构化内容。

### 获取单个块

```json
{ "action": "get_block", "doc_token": "ABC123def", "block_id": "doxcnXXX" }
```

### 更新块文本

```json
{ "action": "update_block", "doc_token": "ABC123def", "block_id": "doxcnXXX", "content": "新文本" }
```

### 删除块

```json
{ "action": "delete_block", "doc_token": "ABC123def", "block_id": "doxcnXXX" }
```

### 获取评论列表

```json
{ "action": "list_comments", "doc_token": "ABC123def", "page_size": 50 }
```

返回文档的所有评论。使用 `page_token` 进行分页。评论包含 `is_whole` 字段，用于区分全文评论（true）和块级评论（false）。

### 获取单条评论

```json
{ "action": "get_comment", "doc_token": "ABC123def", "comment_id": "comment_xxx" }
```

### 创建评论

```json
{ "action": "create_comment", "doc_token": "ABC123def", "content": "评论内容" }
```

### 获取评论回复列表

```json
{ "action": "list_comment_replies", "doc_token": "ABC123def", "comment_id": "comment_xxx", "page_size": 50 }
```

`page_size` 应为正整数。如果省略，工具默认为 `50`。

### 评论写入范围

当前工具提供文档化的评论写入操作 `create_comment`（全局评论创建）。
对于回复，使用 `list_comment_replies` 获取；回复创建端点未在当前 SDK 中暴露。

## 读取工作流程

1. 从 `action: "read"` 开始 - 获取纯文本 + 统计信息
2. 检查响应中的 `block_types` 是否包含 Table、Image、Code 等
3. 如果存在结构化内容，使用 `action: "list_blocks"` 获取完整数据

## 配置

```yaml
channels:
  feishu:
    tools:
      doc: true  # 默认: true
```

**注意：** `feishu_wiki` 依赖此工具 - wiki 页面内容通过 `feishu_doc` 读写。

## 权限

必需：`docx:document`、`docx:document:readonly`、`docx:document.block:convert`、`drive:drive`

评论操作：
- 读取评论：`docx:document.comment:read`
- 写入评论：`docx:document.comment`（可选，用于 create_comment）

---

## 本地文档保存

从飞书同步或创建的文档，在本地保存时统一存放在项目根目录的 `doc/` 目录下：

| 文档类型 | 输出路径 |
|---------|---------|
| 飞书同步文档 | `doc/<文档名>.md` |
