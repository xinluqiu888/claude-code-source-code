# plugin/index.tsx

## 文件描述
Plugin 命令配置 - 管理 Claude Code 插件系统

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | plugin |
| 别名 | plugins, marketplace |
| 描述 | Manage your Claude Code plugins |
| 可用性 | 全局可用 |
| 参数 | 支持子命令参数 |

## 函数概述

### parseArgs
解析插件命令参数，支持多种子命令：
- menu：默认菜单界面
- help：显示帮助信息
- install：安装插件
- manage：管理插件
- uninstall：卸载插件
- enable：启用插件
- disable：禁用插件
- validate：验证插件配置
- marketplace：浏览市场

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'plugin',
  aliases: ['plugins', 'marketplace'],
  description: 'Manage your Claude Code plugins',
  availability: [GLOBAL_AVAILABILITY],
  args: { ... },
  load: () => import('./plugin.js'),
}
```

### 参数定义
支持单个子命令参数，带有详细的参数说明：
- name: 'subcommand'
- required: false
- description: 描述支持的子命令
- suggest: 提供子命令建议列表

## 设计点

1. **多别名支持**：提供 `plugins` 和 `marketplace` 两个别名
2. **子命令系统**：支持丰富的子命令参数
3. **参数建议**：提供自动补全功能
4. **全局可用**：不限制特定环境

## 与其他文件的关系

- 导入 `./plugin.js` 获取实际组件
- 导入 `./parseArgs.js` 获取参数解析逻辑
- 依赖 `../../commands.js` 中的类型定义

## 注意事项

- 参数是可选的（required: false）
- 支持通过 `suggest` 提供智能提示
- 子命令参数可以精确匹配建议列表
