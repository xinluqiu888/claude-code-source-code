# classifyForCollapse.ts — MCP工具分类器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/MCPTool/classifyForCollapse.ts`
- **作用**: 将MCP工具分类为搜索/读取操作，用于UI折叠

## 核心内容详解

### 分类逻辑

```typescript
export function classifyMcpToolForCollapse(
  _serverName: string,
  toolName: string,
): { isSearch: boolean; isRead: boolean }
```

### 工具列表

维护两个工具名称集合：

1. **SEARCH_TOOLS**: 搜索类工具
   - Slack: slack_search_public, slack_search_channels...
   - GitHub: search_code, search_repositories...
   - Linear: search_documentation
   - Datadog: search_logs, search_spans...
   - Notion: search
   - Google Drive: google_drive_search
   - 等等...

2. **READ_TOOLS**: 读取类工具
   - Slack: slack_read_channel, slack_read_thread...
   - GitHub: get_file_contents, list_branches...
   - Filesystem: read_file, list_directory...
   - Git: git_status, git_diff...
   - 等等...

### 名称规范化

```typescript
function normalize(name: string): string {
  return name
    .replace(/([a-z])([A-Z])/g, '$1_$2')  // camelCase → snake_case
    .replace(/-/g, '_')                    // kebab-case → snake_case
    .toLowerCase()
}
```

## 设计要点

1. **显式允许列表**: 使用白名单而非黑名单
2. **工具名稳定**: 匹配基于工具名而非服务器名
3. **大小写不敏感**: 统一转换为小写snake_case
4. **保守策略**: 未知工具不折叠

## 与其他文件的关系

- **collapseReadSearch.ts**: 调用分类器决定是否折叠

## 注意事项

- 需要持续更新以支持新的MCP服务器
- 社区服务器可能使用不同的命名约定
