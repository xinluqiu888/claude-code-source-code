# color-diff/index.ts — 语法高亮与彩色差异显示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/native-ts/color-diff/index.ts`
- **类型**: TypeScript 模块
- **作用**: vendor/color-diff-src（Rust NAPI模块）的纯TypeScript移植

## 功能概述

本模块是Rust语法高亮和差异显示模块的纯TypeScript重新实现。原始版本使用syntect+bat进行语法高亮和similar crate进行单词差异。此移植使用highlight.js（已通过cli-highlight依赖）和diff npm包的diffArrays。

## 核心内容详解

### 公共API

```typescript
export class ColorDiff {
  constructor(hunk: Hunk, firstLine: string | null, filePath: string, prefixContent?: string | null)
  render(themeName: string, width: number, dim: boolean): string[] | null
}

export class ColorFile {
  constructor(code: string, filePath: string)
  render(themeName: string, width: number, dim: boolean): string[] | null
}

export function getSyntaxTheme(themeName: string): SyntaxTheme
export function getNativeModule(): NativeModule | null
```

### Hunk类型

```typescript
type Hunk = {
  oldStart: number      // 旧文件起始行
  oldLines: number      // 旧文件行数
  newStart: number      // 新文件起始行
  newLines: number      // 新文件行数
  lines: string[]       // 差异行（以+/-/ 开头）
}
```

### 颜色模式

支持三种终端颜色模式：
- `truecolor`: 24位真彩色（COLORTERM=truecolor或24bit）
- `color256`: 256色（xterm-256色板）
- `ansi`: 16色ANSI

### 主题系统

**内置主题**:
- Monokai Extended（暗色主题，daltonized变体支持色盲）
- GitHub（亮色主题，daltonized变体）
- Ansi（16色降级主题）

**差异颜色**:
- Add: 绿色背景/装饰（暗色: rgb(2,40,0)，亮色: rgb(220,255,220)）
- Delete: 红色背景/装饰（暗色: rgb(61,1,0)，亮色: rgb(255,220,220)）
- Daltonized变体使用蓝色替代绿色，适配绿色色盲

### 语法高亮

**语言检测**:
1. 文件名匹配（Dockerfile、Makefile等）
2. 扩展名匹配（.ts, .js, .py等）
3. Shebang检测（#!行）
4. 首行特征（<?php, <?xml等）

**Scope映射**:
```typescript
const MONOKAI_SCOPES: Record<string, Color> = {
  keyword: rgb(249, 38, 114),      // 粉色
  _storage: rgb(102, 217, 239),    // 青色
  built_in: rgb(166, 226, 46),     // 绿色
  type: rgb(166, 226, 46),         // 绿色
  number: rgb(190, 132, 255),      // 紫色
  string: rgb(230, 219, 116),      // 黄色
  // ...
}
```

**Storage关键词特殊处理**:
const/function/class等关键词映射到_storage（青色）而非keyword（粉色）。

### 单词级差异

**Token化**:
- 单词字符: `[\p{L}\p{N}_]`
- 空白字符: `\s`
- 单个标点符号

**相邻配对**: 寻找删除行后的添加行，计算单词级差异

**变化阈值**: `CHANGE_THRESHOLD = 0.4`
- 如果变化超过40%，不显示单词级差异（整行高亮）

### 渲染流程

**ColorDiff渲染**:
1. 检测语言和颜色模式
2. 解析hunk行，分配行号和标记
3. 计算单词级差异（非dim模式）
4. 语法高亮每行
5. 应用差异背景
6. 文本折行到终端宽度
7. 添加行号和差异标记

**ColorFile渲染**:
1. 检测语言
2. 语法高亮每行
3. 文本折行
4. 添加行号（无差异标记）

### 懒加载highlight.js

```typescript
function hljs(): HLJSApi {
  if (cachedHljs) return cachedHljs
  const mod = require('highlight.js')
  cachedHljs = 'default' in mod && mod.default ? mod.default : mod
  return cachedHljs!
}
```

- 延迟加载直到首次渲染
- 避免50MB+的启动开销
- 处理CJS/ESM互操作

## 设计要点

1. **ANSI256近似**: 使用感知距离的RGB到256色转换
2. **Unicode宽度**: 使用`stringWidth`处理全角字符
3. **代码点安全**: 正确处理UTF-16代理对
4. **Emitter形状检测**: 验证highlight.js版本兼容性

## 与其他文件的关系

- **被 StructuredDiff.tsx 使用**: 差异显示组件
- **被测试预加载导入**: test/preload.ts通过StructuredDiff.tsx导入
- **使用 highlight.js**: 语法高亮库

## 注意事项

1. **语法差异**: highlight.js的语法范围与syntect不完全一致
   - 普通标识符和运算符（=、:）无范围，显示默认前景色
   - 输出结构（行号、标记、背景、单词差异）与原始一致

2. **BAT_THEME**: 环境变量支持是stub，highlight.js无bat主题集

3. **性能**: 首次渲染延迟加载highlight.js，后续复用

4. **测试导出**: `__test`对象包含内部函数供测试
