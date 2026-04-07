# review/reviewRemote.ts

## 文件描述
Review Remote 实现 - 远程代码审查（Ultrareview）功能

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 工具模块 |
| 导出 | ultrareview |
| 功能 | 执行远程代码审查 |
| 特性 | 支持PR模式和分支模式，处理超额计费 |

## 函数概述

### ultrareview
主函数，处理远程代码审查流程：
1. 验证用户权限
2. 处理超额计费对话框
3. 根据模式执行不同审查策略
4. PR模式：基于Pull Request审查
5. 分支模式：基于分支差异审查

### 核心流程
```typescript
export async function ultrareview(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
  mode: 'pr' | 'branch',
): Promise<React.ReactNode | null>
```

## 核心内容

### 支持的审查模式
| 模式 | 描述 |
|------|------|
| pr | 基于 Pull Request 的审查 |
| branch | 基于分支差异的审查 |

### 计费处理
- 检查用户是否有足够的配额
- 显示超额计费对话框
- 处理API计费逻辑

### 权限验证
- 验证用户访问权限
- 检查仓库访问权限
- 验证GitHub集成状态

## 设计点

1. **双模式支持**：支持PR模式和分支模式满足不同场景
2. **计费透明**：明确显示费用和超额处理
3. **权限安全**：多层次权限验证确保安全
4. **异步处理**：完整的异步流程处理

## 与其他文件的关系

- 被 `ultrareviewCommand.tsx` 调用
- 使用 `LocalJSXCommandContext` 和 `LocalJSXCommandOnDone` 类型
- 集成计费系统处理费用
- 集成GitHub API进行代码获取

## 注意事项

- 需要处理用户拒绝超额计费的情况
- PR模式需要有效的Pull Request信息
- 分支模式需要比较的两个分支
- 费用计算基于代码变更量
