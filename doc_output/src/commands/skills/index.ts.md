# skills/index.ts

## 文件描述
Skills 命令配置 - 列出可用技能

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | skills |
| 描述 | List available skills |
| 可用性 | 全局可用 |
| 功能 | 显示所有可用的技能列表 |

## 函数概述

### 命令配置
配置技能列表命令：
- 简单的功能展示
- 无参数需求

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'skills',
  description: 'List available skills',
  availability: [GLOBAL_AVAILABILITY],
  load: () => import('./skills.js'),
}
```

## 设计点

1. **简洁配置**：无参数，单一功能
2. **全局可用**：所有环境都可以使用
3. **技能展示**：列出系统支持的所有技能
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./skills.js` 获取组件实现
- 使用 `GLOBAL_AVAILABILITY` 定义全局可用性

## 注意事项

- 技能列表需要实时更新
- 某些技能可能有权限限制
- 列表需要分类展示
- 支持技能搜索和筛选
