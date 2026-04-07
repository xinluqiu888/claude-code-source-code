# stats/index.ts

## 文件描述
Stats 命令配置 - 显示使用统计和活动

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | stats |
| 描述 | Show your Claude Code usage statistics and activity |
| 可用性 | claude-ai |
| 功能 | 展示用户使用统计和活动数据 |

## 函数概述

### 命令配置
配置统计信息命令：
- 仅用于显示统计数据
- 需要 claude-ai 环境

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'stats',
  description: 'Show your Claude Code usage statistics and activity',
  availability: ['claude-ai'],
  load: () => import('./stats.js'),
}
```

## 设计点

1. **简洁配置**：无参数，单一功能
2. **环境限制**：仅在 claude-ai 环境可用
3. **数据展示**：展示详细的使用统计
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./stats.js` 获取组件实现
- 依赖 `Command` 类型定义

## 注意事项

- 需要用户登录状态
- 统计数据需要实时计算
- 可能涉及隐私数据处理
- 某些统计可能延迟更新
