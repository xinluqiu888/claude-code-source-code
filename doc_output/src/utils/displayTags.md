# displayTags.ts — 显示标签处理工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/displayTags.ts`
- **主要功能**: 剥离系统注入的 XML 标签，用于 UI 标题显示
- **关键依赖**: 无

## 功能概述

该模块提供函数用于剥离系统注入的 XML-like 标签：
1. 通用标签剥离（用于 UI 标题）
2. 允许空结果的标签剥离
3. IDE 上下文标签专用剥离

系统注入的上下文（IDE 元数据、Hook 输出、任务通知等）以标签形式到达，不应在标题中显示。

## 核心内容详解

### 标签匹配模式

```typescript
const XML_TAG_BLOCK_PATTERN = /<([a-z][\w-]*)(?:\s[^>]*)?>[\s\S]*?<\/\1>\n?/g
```

匹配规则：
- 小写标签名（`[a-z][\w-]*`）
- 可选属性
- 多行内容（非贪婪匹配）
- 通过反向引用确保开闭标签匹配

**为什么只匹配小写标签**：
- 用户可能提到 JSX/HTML 组件（如 `<Button>`、`<!DOCTYPE html>`）
- 这些以大写字母或 `!` 开头，不会误匹配

### 核心函数

#### stripDisplayTags

```typescript
export function stripDisplayTags(text: string): string
```

剥离 XML-like 标签块，用于 UI 标题。

- 如果剥离后为空，返回原始文本（避免显示空白）
- 用于 `/rewind`, `/resume`, 会话标题等

#### stripDisplayTagsAllowEmpty

```typescript
export function stripDisplayTagsAllowEmpty(text: string): string
```

类似 `stripDisplayTags`，但允许返回空字符串。

用于：
- `getLogDisplayTitle` 检测纯命令提示（如 `/clear`）
- `extractTitleText` 跳过纯 XML 消息

#### stripIdeContextTags

```typescript
export function stripIdeContextTags(text: string): string
```

仅剥离 IDE 注入的上下文标签（`ide_opened_file`, `ide_selection`）。

用于 `textForResubmit`，确保 UP 箭头重新提交时：
- 保留用户输入的小写 HTML（如 `<code>foo</code>`）
- 移除 IDE 噪音

## 设计要点

1. **通用模式**: 避免维护不断增长的允许列表
2. **大小写敏感**: 小写标签匹配避免误删用户内容
3. **非贪婪匹配**: 相邻标签块正确分离
4. **回退处理**: `stripDisplayTags` 在结果为空时返回原文

## 与其他文件的关系

该模块无外部依赖，可被任何模块安全导入。

## 注意事项

1. **标签格式**: 假设系统标签为有效 XML 格式
2. **大小写**: 用户内容中的大写标签名保留
3. **多行**: 支持跨多行标签
4. **性能**: 正则表达式使用 `g` 标志全局匹配
