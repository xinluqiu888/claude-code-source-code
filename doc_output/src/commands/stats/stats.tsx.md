# stats/stats.tsx

## 文件描述
Stats 组件 - 使用统计和活动展示界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 渲染 Stats 组件 |
| 组件 | Stats |

## 函数概述

### call
简单的包装函数，渲染 Stats 组件：
- 直接返回 Stats 组件
- 传递 onDone 回调

### 函数签名
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
): Promise<React.ReactNode | null>
```

## 核心内容

### 组件渲染
```typescript
return <Stats onDone={onDone} />;
```

### Stats 组件
- 显示使用统计
- 展示活动历史
- 提供数据可视化

## 设计点

1. **组件复用**：直接复用 Stats 组件
2. **简化实现**：无需额外逻辑
3. **回调传递**：确保完成时通知父组件
4. **异步支持**：函数设计为 async

## 与其他文件的关系

- 导入 `Stats` 组件从 `../../components/Stats.js`
- 使用 `LocalJSXCommandOnDone` 类型

## 注意事项

- Stats 组件处理所有展示逻辑
- 需要处理无数据的情况
- 统计数据可能需要加载
- 支持不同时间范围筛选
