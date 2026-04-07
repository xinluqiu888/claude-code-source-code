# keybindings.ts — 打开键盘快捷键配置文件

> **一句话总结**：打开或创建用户键盘快捷键配置文件，使用系统编辑器编辑。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/keybindings/keybindings.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 53 行 |
| 主要职责 | 管理用户键盘快捷键配置文件 |

---

## 功能概述

该文件实现了 `/keybindings` 命令，用于打开或创建用户的键盘快捷键配置文件。如果不存在配置文件，会创建一个带有模板的文件；如果已存在，则直接打开。文件使用系统默认编辑器打开。

---

## 核心内容详解

### 导入与依赖

```typescript
import { mkdir, writeFile } from 'fs/promises'
import { dirname } from 'path'
import {
  getKeybindingsPath,
  isKeybindingCustomizationEnabled,
} from '../../keybindings/loadUserBindings.js'
import { generateKeybindingsTemplate } from '../../keybindings/template.js'
import { getErrnoCode } from '../../utils/errors.js'
import { editFileInEditor } from '../../utils/promptEditor.js'
```

### 主要函数

#### call(): Promise<{ type: 'text'; value: string }>

- **类型**: `async function`
- **返回值**: `Promise<{ type: 'text'; value: string }>`
- **用途**: 主入口函数

**执行流程**:

1. **检查功能启用**: 如果 `isKeybindingCustomizationEnabled()` 返回false，返回功能未启用消息

2. **获取配置文件路径**: `getKeybindingsPath()`

3. **创建目录**: 使用 `mkdir` 递归创建配置目录

4. **创建文件（如果不存在）**:
   - 使用 `flag: 'wx'` 独占创建模式
   - 写入 `generateKeybindingsTemplate()` 生成的模板内容
   - 捕获 `EEXIST` 错误表示文件已存在

5. **在编辑器中打开**: `editFileInEditor(keybindingsPath)`

6. **返回结果**: 根据文件是否已存在返回相应消息

---

## 设计要点

1. **功能开关**: 通过 `isKeybindingCustomizationEnabled()` 控制功能可用性。

2. **原子创建**: 使用 `wx` 标志确保文件不存在时才创建，避免竞争条件。

3. **模板内容**: 使用 `generateKeybindingsTemplate()` 提供默认配置示例。

4. **系统编辑器**: 使用 `editFileInEditor` 调用系统默认编辑器。

5. **目录递归创建**: 使用 `recursive: true` 确保父目录存在。

---

## 与其他文件的关系

**依赖**:
- `getKeybindingsPath`, `isKeybindingCustomizationEnabled` (`../../keybindings/loadUserBindings.js`)
- `generateKeybindingsTemplate` (`../../keybindings/template.js`)
- `getErrnoCode` (`../../utils/errors.js`)
- `editFileInEditor` (`../../utils/promptEditor.js`)

**被依赖**:
- `keybindings/index.ts` - 导出为命令配置

---

## 注意事项

1. **预览功能**: `isKeybindingCustomizationEnabled` 表明这是预览功能。

2. **wx标志**: 独占创建模式确保不会意外覆盖已有配置。

3. **EEXIST错误**: 专门处理文件已存在的情况，这不是错误而是预期情况。

4. **编辑器选择**: `$VISUAL` 或 `$EDITOR` 环境变量决定使用哪个编辑器。
