# status/status.tsx

## 文件描述
Status 组件 - 系统状态展示界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 渲染 Settings 组件，默认显示状态标签 |
| 组件 | Settings |
| 默认标签 | Status |

## 函数概述

### call
包装函数，渲染 Settings 组件并设置默认标签：
- 返回 Settings 组件
- 设置 defaultTab="Status"

### 函数签名
```typescript
export const call: LocalJSXCommandCall = async (onDone, context) => {
  return <Settings onClose={onDone} context={context} defaultTab="Status" />;
};
```

## 核心内容

### 组件渲染
```typescript
return (
  <Settings
    onClose={onDone}
    context={context}
    defaultTab="Status"
  />
);
```

### Settings 组件
- 多标签页界面
- Status 标签显示系统状态
- 支持切换其他标签

## 设计点

1. **组件复用**：复用 Settings 组件避免重复
2. **标签指定**：通过 defaultTab 指定默认标签
3. **完整功能**：保留 Settings 的所有功能
4. **上下文传递**：传递完整的 context

## 与其他文件的关系

- 导入 `Settings` 组件从 `../../components/Settings/Settings.js`
- 使用 `LocalJSXCommandCall` 类型
- 使用 `LocalJSXCommandContext` 类型

## 注意事项

- Settings 组件处理状态显示逻辑
- 用户可以切换到其他标签
- context 需要包含必要的属性
- onClose 回调在关闭时触发
