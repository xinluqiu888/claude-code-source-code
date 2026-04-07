# resume/index.ts

## 文件描述
Resume 命令配置 - 恢复之前的会话

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | resume |
| 别名 | history, logs |
| 描述 | Resume a previous conversation |
| 可用性 | 全局可用 |
| 参数 | 可选的会话ID参数 |

## 函数概述

### 命令配置
配置恢复会话命令，支持别名：
- resume：主命令
- history：别名，查看历史
- logs：别名，查看日志

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'resume',
  aliases: ['history', 'logs'],
  description: 'Resume a previous conversation',
  availability: [GLOBAL_AVAILABILITY],
  args: {
    name: 'sessionId',
    required: false,
    description: 'The session to resume',
    suggest: suggestSessions,
  },
  load: () => import('./resume.js'),
}
```

### 参数定义
- name: 'sessionId'
- required: false（可选参数）
- description: 要恢复的会话
- suggest: 提供会话建议列表

## 设计点

1. **多别名支持**：提供 `history` 和 `logs` 两个别名，符合用户习惯
2. **可选参数**：会话ID为可选，不指定时显示选择界面
3. **智能建议**：提供 `suggestSessions` 自动补全功能
4. **懒加载**：动态导入组件代码

## 与其他文件的关系

- 导入 `suggestSessions` 提供会话建议
- 导入 `./resume.js` 获取实际组件实现
- 依赖 `GLOBAL_AVAILABILITY` 定义全局可用性

## 注意事项

- 参数是可选的，支持交互式选择
- 需要 `suggestSessions` 提供历史会话列表
- 别名设计符合不同用户的使用习惯
- 组件实现需要处理无参数的情况
