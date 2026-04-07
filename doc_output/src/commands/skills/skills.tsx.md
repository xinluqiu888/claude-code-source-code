# skills/skills.tsx

## 文件描述
Skills 组件 - 可用技能列表展示界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 渲染技能菜单组件 |
| 组件 | SkillsMenu |

## 函数概述

### call
简单的包装函数，渲染 SkillsMenu 组件：
- 直接返回 SkillsMenu 组件
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
return <SkillsMenu onDone={onDone} />;
```

### SkillsMenu 组件
- 显示所有可用技能
- 支持技能分类
- 提供技能详情查看

## 设计点

1. **组件复用**：直接复用 SkillsMenu 组件
2. **简化实现**：无需额外逻辑
3. **回调传递**：确保完成时通知父组件
4. **异步支持**：函数设计为 async

## 与其他文件的关系

- 导入 `SkillsMenu` 组件从 `../../components/SkillsMenu.js`
- 使用 `LocalJSXCommandOnDone` 类型

## 注意事项

- SkillsMenu 处理所有展示逻辑
- 需要处理无技能的情况
- 技能分类由 SkillsMenu 管理
- 支持技能详情展开
