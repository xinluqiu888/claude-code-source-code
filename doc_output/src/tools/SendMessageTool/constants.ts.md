# constants.ts — SendMessage工具常量

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SendMessageTool/constants.ts`
- **作用**: 定义SendMessage工具名称常量

## 核心内容详解

### 工具名称常量

```typescript
export const SEND_MESSAGE_TOOL_NAME = 'SendMessage'
```

## 设计要点

1. **单一名称**: 该文件仅包含工具名称常量
2. **导出常量**: 使用大写命名风格
3. **集中管理**: 避免在多处硬编码字符串

## 与其他文件的关系

- **SendMessageTool.ts**: 导入使用工具名称
- **工具注册表**: 可能用于工具发现和注册
