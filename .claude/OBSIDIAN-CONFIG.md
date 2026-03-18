# Obsidian 配置说明

## 当前配置（2026-03-11 更新）

### 已禁用
- ~~Obsidian MCP Server~~ - 已移除 `.mcp.json` 配置

### 使用中

#### 1. Skills（优先使用）

| Skill | 用途 |
|-------|------|
| `obsidian-markdown` | 创建和编辑 Obsidian Flavored Markdown |
| `obsidian-cli` | 使用 Obsidian CLI 与运行中的 Obsidian 交互 |
| `json-canvas` | 创建和编辑 JSON Canvas 文件 |
| `obsidian-bases` | 创建和编辑 Bases 数据库文件 |
| `defuddle` | 从网页提取干净的 Markdown |

#### 2. Obsidian CLI 常用命令

```bash
# 读取笔记
obsidian read file="笔记名"

# 创建笔记
obsidian create name="新笔记" content="内容"

# 追加内容
obsidian append file="笔记名" content="新内容"

# 搜索
obsidian search query="关键词"

# 每日笔记
obsidian daily:read
obsidian daily:append content="新内容"

# 设置属性
obsidian property:set name="status" value="done" file="笔记名"
```

### 工具选择原则

当需要操作 Obsidian 时，优先顺序：

1. **Skills** - `obsidian-markdown`（直接读写文件）
2. **Obsidian CLI** - 需要与运行中的 Obsidian 交互时
3. ~~MCP~~ - 不再使用

---

## 变更日志

| 日期 | 变更 |
|------|------|
| 2026-03-11 | 移除 Obsidian MCP，改用 Skills + CLI |
