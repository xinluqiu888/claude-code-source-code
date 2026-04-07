# UI.tsx — 团队创建工具 UI 组件

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamCreateTool/UI.tsx`
- **类型**: TypeScript/React UI 模块
- **功能**: 提供 TeamCreate 工具的 UI 渲染函数

## 核心内容详解

### 导出函数

```typescript
export function renderToolUseMessage(input: Partial<Input>): React.ReactNode
```

### 核心逻辑

返回格式化的创建团队消息：
```
create team: {team_name}
```

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `input` | `Partial<Input>` | 工具输入参数的部分对象 |
| `input.team_name` | `string` | 团队名称 |

## 与其他文件的关系

### 被依赖
- `TeamCreateTool.ts` — 工具主实现文件
