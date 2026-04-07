# symbolContext.ts — 符号上下文提取

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/symbolContext.ts`
- **作用**: 从文件中提取特定位置的符号/单词

## 核心内容详解

### 核心函数

```typescript
export function getSymbolAtPosition(
  filePath: string,
  line: number,      // 0-based
  character: number, // 0-based
): string | null
```

### 实现细节

1. **性能优化**: 只读取前64KB（约1000行代码）
2. **符号模式**: 支持多种编程语言
   - 标准标识符: `[\w$'!]+`
   - Rust生命周期: `'a, 'static`
   - Rust宏: `macro_name!`
   - 操作符: `[+\-*/%&|^~<>=]+`
3. **长度限制**: 最大30字符

### 使用场景

在UI中显示操作上下文：
```typescript
const symbol = getSymbolAtPosition(filePath, line - 1, character - 1)
// 显示: operation: "goToDefinition", symbol: "myFunction", in: "file.ts"
```

## 设计要点

1. **同步读取**: 从React渲染函数调用，必须使用同步API
2. **优雅降级**: 错误时返回null，UI回退到位置显示
3. **调试日志**: 记录提取失败的错误

## 与其他文件的关系

- **UI.tsx**: 调用此函数获取符号上下文
- **utils/fsOperations.ts**: 文件系统操作
