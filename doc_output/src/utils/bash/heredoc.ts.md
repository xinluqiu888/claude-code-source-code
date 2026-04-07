# heredoc.ts — Heredoc语法处理器

> **一句话总结**：提取和恢复Shell命令中的heredoc语法，解决shell-quote无法正确解析`<<`操作符的问题。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/heredoc.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~734行 |
| 主要职责 | Heredoc提取、占位符替换和安全验证 |

---

## 功能概述

Heredoc（`<<EOF`）是Shell的多行输入语法，但shell-quote库将其解析为两个`<`重定向操作符，导致后续分析出错。

该模块提供：
1. **提取**：从命令中提取heredoc内容，替换为占位符
2. **恢复**：将占位符还原为原始heredoc内容
3. **安全验证**：多层安全检查防止解析差异被利用

支持格式：
- `<<EOF` - 基本heredoc
- `<<'EOF'` / `<<"EOF"` - 带引号定界符（无变量扩展）
- `<<-EOF` - 带-前缀（去除前导tab）
- `<<-\'EOF\'` - 组合形式

---

## 核心内容详解

### 主要导出类型

- **`HeredocInfo`** - Heredoc信息
  - `fullText`: 完整文本（含操作符和内容）
  - `delimiter`: 定界符名称
  - 多个索引位置用于精确定位

- **`HeredocExtractionResult`**
  - `processedCommand`: 替换占位符后的命令
  - `heredocs`: Map<占位符, HeredocInfo>

### 主要导出函数

- **`extractHeredocs`**
  - 参数：`command`（string）, `options?`（{quotedOnly?: boolean}）
  - 返回值：`HeredocExtractionResult`
  - 用途：提取所有heredoc，替换为占位符
  - 安全：多层验证防止解析差异攻击

- **`restoreHeredocs`**
  - 参数：`parts`（string[]）, `heredocs`（Map）
  - 返回值：`string[]`
  - 用途：将占位符恢复为原始heredoc

- **`containsHeredoc`**
  - 参数：`command`（string）
  - 返回值：`boolean`
  - 用途：快速检测是否含heredoc语法

### 安全特性

1. **算术上下文检测**：`((1<<2))`中的`<<`是位操作符，非heredoc
2. **引号状态跟踪**：逐字符扫描确定是否在引号内
3. **注释跳过**：`# <<EOF`是注释，非heredoc
4. **反斜杠处理**：`\<<`是转义，非heredoc
5. **嵌套过滤**：heredoc内的`<<`只是文本
6. **重叠检测**：多个heredoc共享同一内容起始位置时拒绝

---

## 与其他文件的关系

- **依赖**：
  - `crypto` - 生成随机salt
- **被依赖**：
  - `commands.ts` - 命令分割
  - `ast.ts` - 安全验证
  - `shellQuoting.ts` - heredoc检测

---

## 注意事项

- 使用随机salt防止占位符与命令文本冲突
- 引号跟踪是安全关键，必须准确
- 处理失败时返回原始命令（安全降级）
- <<-（带-前缀）只去除前导tab，不是空格
