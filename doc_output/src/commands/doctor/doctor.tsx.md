# doctor.tsx — 诊断工具界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/doctor/doctor.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 行数 | 6 行 |
| 主要职责 | 提供 Claude Code 安装和设置诊断界面 |

## 功能概述

该文件实现了 `/doctor` 命令，用于诊断和验证 Claude Code 的安装和设置。这是一个简单的包装组件，实际功能由 `Doctor` 屏幕组件实现。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export const call: LocalJSXCommandCall = (onDone, _context, _args) => {
  return Promise.resolve(<Doctor onDone={onDone} />);
}
```

### Doctor 屏幕组件

实际功能包括：
- 检查 Claude Code 安装完整性
- 验证配置文件
- 检查依赖项
- 测试网络连接
- 提供修复建议

## 设计要点

1. **简单包装**：仅作为 Doctor 屏幕的入口
2. **Promise 包装**：返回 Promise.resolve 以符合接口
3. **忽略额外参数**：不使用 context 和 args

## 与其他文件的关系

- **Doctor.tsx**: 实际的诊断界面实现
- **index.ts**: 命令注册
