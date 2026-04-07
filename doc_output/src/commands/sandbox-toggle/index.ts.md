# sandbox-toggle/index.ts

## 文件描述
Sandbox Toggle 命令配置 - 沙盒功能开关

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | sandbox |
| 别名 | toggle, sandboxes |
| 描述 | 动态描述（基于沙盒启用状态） |
| 可用性 | 全局可用 |

## 函数概述

### getDescription
动态生成命令描述：
- 沙盒已启用：`Toggle the sandboxing feature (currently enabled)`
- 沙盒已禁用：`Toggle the sandboxing feature (currently disabled)`

### 命令配置
配置沙盒功能切换命令：
- 支持别名 toggle 和 sandboxes
- 动态描述显示当前状态

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'sandbox',
  aliases: ['toggle', 'sandboxes'],
  description: 'Toggle the sandboxing feature (currently enabled/disabled)',
  availability: [GLOBAL_AVAILABILITY],
  load: () => import('./sandbox-toggle.js'),
}
```

### 状态检测
```typescript
const getDescription = () => {
  const isEnabled = isSandboxingEnabled();
  return `Toggle the sandboxing feature (currently ${isEnabled ? 'enabled' : 'disabled'})`;
};
```

## 设计点

1. **动态描述**：实时显示沙盒功能的当前状态
2. **多别名支持**：提供 toggle 和 sandboxes 别名
3. **状态感知**：检测当前沙盒配置状态
4. **简洁直观**：用户一眼就能看到当前状态

## 与其他文件的关系

- 导入 `isSandboxingEnabled` 检测沙盒状态
- 导入 `./sandbox-toggle.js` 获取组件实现
- 依赖 `GLOBAL_AVAILABILITY` 定义全局可用性

## 注意事项

- 描述需要实时反映最新状态
- 状态检测可能涉及配置读取
- 别名设计便于用户记忆
- 组件负责实际的切换逻辑
