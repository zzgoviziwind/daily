# Obsidian + Claude Code 使用指南

## 快速开始

### 1. 完成 MCP 配置

在使用 MCP 功能前，需要先在 Obsidian 中安装 **Local REST API** 插件：

1. 打开 Obsidian
2. 进入 `设置` → `第三方插件` → `浏览`
3. 搜索 `Local REST API` (作者：coddingtonbear)
4. 安装并启用插件
5. 复制插件设置中的 **API Key**
6. 更新 `.mcp.json` 文件，将 `OBSIDIAN_API_KEY` 替换为你的实际 API Key

### 2. Skills 已就绪

已安装以下 Skills，Claude Code 会自动使用：

- **obsidian-markdown** - 编写正确的 Obsidian Markdown 语法
- **json-canvas** - 创建和编辑 Canvas 画布
- **obsidian-bases** - 创建数据库视图
- **obsidian-cli** - 使用 CLI 操作 Obsidian
- **defuddle** - 清理网页内容

### 3. 笔记目录

已创建以下目录结构：

```
📂 00-Inbox/          # 临时收集
📂 10-Daily/          # 每日笔记
📂 20-Projects/       # 项目笔记
📂 30-Areas/          # 责任领域
📂 40-Resources/      # 资源知识库
📂 50-Archives/       # 归档
📂 Templates/         # 模板
📂 Attachments/       # 附件
```

---

## 如何做笔记

### 创建每日笔记

告诉 Claude：
> 创建今天的每日笔记

Claude 会在 `10-Daily/` 目录下创建格式正确的日报，使用 `Templates/Daily.md` 模板。

### 创建项目笔记

告诉 Claude：
> 为新项目创建一个笔记，项目名是 XXX

Claude 会使用 `Templates/Project.md` 模板创建项目笔记。

### 做学习笔记

告诉 Claude：
> 记录关于 XXX 的学习笔记

Claude 会：
1. 使用 `Templates/Note.md` 模板
2. 添加相关的内部链接 `[[ ]]`
3. 使用 callouts 突出重点
4. 添加合适的标签

---

## 看笔记

### 搜索笔记

如果已配置 MCP：
```
搜索关于 XXX 的笔记
```

或者手动在 Obsidian 中使用 `Ctrl/Cmd + O` 快速打开笔记。

### 浏览笔记

按日期浏览：`10-Daily/` 目录下按 `YYYY-MM-DD.md` 格式查看每日笔记

按项目浏览：`20-Projects/` 目录下查看所有项目

---

## Obsidian 语法速查

### 内部链接
```markdown
[[笔记名]]              链接到其他笔记
[[笔记名 | 显示文本]]    自定义显示文本
[[笔记名#标题]]          链接到特定标题
```

### 嵌入内容
```markdown
![[笔记名]]           嵌入整篇笔记
![[图片.png]]         嵌入图片
```

### Callouts（提示框）
```markdown
> [!note]
> 普通提示

> [!warning] 警告
> 重要提醒

> [!tip]- 折叠提示
> 可折叠内容
```

### 高亮和注释
```markdown
==高亮文本==          黄色高亮
%%隐藏注释%%          编辑模式可见
```

### 任务列表
```markdown
- [ ] 待办事项
- [x] 已完成
```

---

## 常用工作流

### 1. 晨间计划

```
1. 创建/打开今日日报
2. 列出今日重点（3 件事）
3. 查看昨日笔记，转移未完成的任务
```

### 2. 会议记录

```
1. 创建会议笔记
2. 记录讨论要点
3. 标记决策和行动项
4. 链接相关项目和人员笔记
```

### 3. 学习笔记

```
1. 阅读/学习内容
2. 创建永久笔记
3. 用自己的话总结
4. 链接到现有知识网络
5. 添加标签便于检索
```

### 4. 项目复盘

```
1. 回顾项目目标
2. 整理项目笔记
3. 提取经验教训
4. 归档到 50-Archives/
```

---

## 与 Claude 协作技巧

### 好的指令

✅ "创建今天的每日笔记，记录我正在做 XXX 项目"
✅ "帮我总结最近一周的学习笔记"
✅ "为我的 XXX 项目创建一个 MOC 索引"
✅ "用 callout 高亮显示这个重要概念"

### 避免的指令

❌ "帮我写笔记"（太模糊）
❌ "整理我的笔记"（没有具体目标）

---

## 下一步

1. **完成 MCP 配置** - 安装 Local REST API 插件
2. **创建第一篇笔记** - 让 Claude 帮你创建今天的日报
3. **探索语法** - 尝试 wikilinks、callouts、embeds
4. **建立习惯** - 每天写日报，逐步构建知识库
