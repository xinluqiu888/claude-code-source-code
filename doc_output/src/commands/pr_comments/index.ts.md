# pr_comments/index.ts

## 文件描述
PR Comments 命令配置 - 获取 PR 评论（已移至插件）

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 插件命令 |
| 名称 | pr-comments |
| 描述 | Get comments from a GitHub pull request |
| 插件名称 | pr-comments |

## 核心内容

### 命令配置
```typescript
export default createMovedToPluginCommand({
  name: 'pr-comments',
  description: 'Get comments from a GitHub pull request',
  progressMessage: 'fetching PR comments',
  pluginName: 'pr-comments',
  pluginCommand: 'pr-comments',
  async getPromptWhileMarketplaceIsPrivate(args) {
    return [
      {
        type: 'text',
        text: `...`, // 详细提示词
      },
    ];
  },
})
```

### 功能说明
- 使用 gh CLI 获取 PR 信息
- 获取 PR 级别评论和代码审查评论
- 格式化输出评论内容

## 设计点

1. **插件化**：功能已移至插件
2. **过渡命令**：提供向后兼容
3. **详细提示**：包含完整操作指南

## 与其他文件的关系

- 导入 `createMovedToPluginCommand` 从 `../createMovedToPluginCommand.js`

## 注意事项

- 功能已迁移到插件
- 需要 GitHub CLI
- 需要适当权限
