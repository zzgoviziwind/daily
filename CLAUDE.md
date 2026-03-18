# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Obsidian 笔记库配置

这是一个 Obsidian 笔记库目录，已配置 **Skills** 和 **MCP** 与 Claude Code 集成。

---

## 安装内容

### 1. Skills (已安装)

已安装 [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)，包含 5 个技能：

| 技能 | 描述 |
|------|------|
| `obsidian-markdown` | 创建和编辑 Obsidian Flavored Markdown（维基链接、嵌入、callouts、properties） |
| `obsidian-bases` | 创建和编辑 Bases 数据库文件 |
| `json-canvas` | 创建和编辑 JSON Canvas 文件（.canvas） |
| `obsidian-cli` | 使用 Obsidian CLI 与运行中的 Obsidian 交互 |
| `defuddle` | 从网页提取干净的 Markdown |

### 2. MCP Server (配置中)

MCP 配置在 `.mcp.json` 中，需要完成以下步骤才能使用：

1. **在 Obsidian 中安装 Local REST API 插件**
   - 打开 Obsidian → 设置 → 第三方插件 → 浏览
   - 搜索 "Local REST API" (作者：coddingtonbear)
   - 安装并启用
   - 复制插件设置中的 API Key

2. **更新 `.mcp.json`**
   - 将 `OBSIDIAN_API_KEY` 替换为你的实际 API Key

3. **重启 Claude Code** 使配置生效

---

## 笔记系统架构

### 目录结构

```
D:/myfiles/obsidian/
├── 00-Inbox/          # 临时收集区 - 快速捕获想法
├── 10-Daily/          # 每日笔记 - 日报、日志
├── 20-Projects/       # 项目笔记 - 按项目组织
├── 30-Areas/          # 责任领域 - 长期关注的领域
├── 40-Resources/      # 资源参考 - 知识库、笔记
├── 50-Archives/       # 归档 - 完成的项目
├── Templates/         # 模板 - 笔记模板
└── Attachments/       # 附件 - 图片、PDF 等
```

### 笔记类型

| 类型 | 位置 | 用途 |
|------|------|------|
| 每日笔记 | `10-Daily/YYYY-MM-DD.md` | 记录当天工作、想法 |
| 项目笔记 | `20-Projects/项目名.md` | 项目规划、进度追踪 |
| 永久笔记 | `40-Resources/主题.md` | 知识点、概念解释 |
| MOC | `40-Resources/MOC-主题.md` | 索引相关笔记 |

---

## 与 Claude Code 协作的工作流

### 1. 开始会话

使用 `/init` 开始新会话时，Claude 会：
- 读取最近的每日笔记了解上下文
- 检查活跃项目的笔记
- 在当日日报中记录会话目标

### 2. 创建笔记

让 Claude 创建笔记时，使用 Obsidian Flavored Markdown：

```markdown
---
title: 笔记标题
date: 2024-01-15
tags:
  - 主题
  - 状态
aliases:
  - 别名
---

# 标题

内容使用 [[内部链接]] 关联其他笔记

> [!note] 重点
> 使用 callout 突出重要信息

- [ ] 任务列表
```

### 3. 搜索和读取

```bash
# 搜索笔记（需要 MCP）
mcp obsidian simple_search query="关键词"

# 读取笔记
mcp obsidian get_file_contents path="10-Daily/2024-01-15.md"

# 列出所有笔记
mcp obsidian list_files_in_vault
```

### 4. 日常笔记模板

**每日笔记模板** (`Templates/Daily.md`)：

```markdown
---
date: {{date}}
tags:
  - daily
---

# {{date:YYYY-MM-DD}}

## 今日重点
-

## 工作日志
-

## 学习笔记
-

## 明日计划
- [ ]
```

---

## Obsidian 语法快速参考

### 内部链接

```markdown
[[笔记名]]              # 基本链接
[[笔记名 | 显示文本]]    # 自定义显示
[[笔记名#标题]]          # 链接到标题
[[#同笔记标题]]          # 同笔记内链接
```

### 嵌入

```markdown
![[笔记名]]           # 嵌入笔记
![[图片.png]]         # 嵌入图片
![[PDF.pdf#page=3]]   # 嵌入 PDF 页面
```

### Callouts

```markdown
> [!note]
> 普通笔记

> [!warning] 警告标题
> 警告内容

> [!tip]- 折叠提示
> 可折叠的内容
```

### 高亮和注释

```markdown
==高亮文本==          # 高亮
%%隐藏注释%%          # 仅编辑模式可见
```

### 标签

```markdown
#标签名
#嵌套/标签
```

---

## 常用命令

### Claude Code 命令

```
/init          # 初始化会话，读取最近的笔记
/note          # 创建新笔记（需配置）
/search        # 搜索笔记
```

### Obsidian CLI (需要安装)

```bash
# 读取笔记
obsidian read file="笔记名"

# 创建笔记
obsidian create name="新笔记" content="内容"

# 搜索
obsidian search query="关键词"

# 每日笔记
obsidian daily:read
obsidian daily:append content="新内容"
```

---

## 最佳实践

### 1. 双向链接

- 使用 `[[wikilinks]]` 链接内部笔记
- 创建 MOC (Map of Content) 索引相关笔记
- 定期检查反向链接发现关联

### 2. 渐进式总结

1. 捕获想法到 `00-Inbox/`
2. 整理到对应分类
3. 添加链接和标签
4. 定期归档

### 3. AI 辅助

- 让 Claude 总结长笔记
- 自动生成笔记链接建议
- 创建索引和目录
- 提取关键概念

### 4. 模板使用

为常用笔记类型创建模板：
- 每日笔记
- 项目笔记
- 会议记录
- 读书笔记

---

## 故障排查

### MCP 连接问题

1. 确认 Obsidian 正在运行
2. 检查 Local REST API 插件已启用
3. 验证 API Key 正确
4. 确认端口 27124 未被占用

### Skills 未加载

1. 确认技能文件在 `D:/myfiles/obsidian/.claude/` 目录下
2. 重启 Claude Code
3. 检查技能文件格式是否正确

---

## 参考资源

- [Obsidian Help](https://help.obsidian.md/)
- [Obsidian Flavored Markdown](https://help.obsidian.md/obsidian-flavored-markdown)
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)
- [MCP-Obsidian](https://github.com/shaike1/obsidian-mcp)
