# status/index.ts

## 文件描述
Status 命令配置 - 显示系统状态信息

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | status |
| 描述 | Show Claude Code status including version, model, account, API connectivity, and tool statuses |
| 可用性 | 全局可用 |
| 功能 | 显示系统状态、版本、模型等信息 |

## 函数概述

### 命令配置
配置状态显示命令：
- 显示综合状态信息
- 包括版本、模型、账户等

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'status',
  description: 'Show Claude Code status including version, model, account, API connectivity, and tool statuses',
  availability: [GLOBAL_AVAILABILITY],
  load: () => import('./status.js'),
}
```

### 显示内容
- 版本信息
- 当前模型
- 账户信息
- API 连接状态
- 工具状态

## 设计点

1. **信息丰富**：展示多方面的状态信息
2. **全局可用**：所有环境都可以使用
3. **诊断用途**：便于排查问题
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./status.js` 获取组件实现
- 使用 `GLOBAL_AVAILABILITY` 定义全局可用性

## 注意事项

- 状态信息需要实时获取
- 某些状态可能需要网络请求
- 错误状态需要友好显示
- 可能需要缓存部分信息
