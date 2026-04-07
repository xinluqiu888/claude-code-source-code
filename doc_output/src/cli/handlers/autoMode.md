# handlers/autoMode.ts — 自动模式子命令处理器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/handlers/autoMode.ts` |
| **文件类型** | TypeScript 处理器模块 |
| **行数** | 171 行 |
| **职责** | 实现 `claude auto-mode` 子命令，管理自动模式分类器规则 |

## 功能概述

本模块实现 `claude auto-mode` 子命令集，用于查看、分析和调试自动模式（auto mode）分类器规则。自动模式是 Claude Code 的一项功能，使用 AI 分类器决定工具调用是否应自动批准或需要用户确认。

模块提供三个子命令：
- **`defaults`**：显示默认的分类器规则
- **`config`**：显示当前生效的配置（用户设置 + 默认回退）
- **`critique`**：使用 AI 分析用户编写的自定义规则，提供改进建议

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `errorMessage` | `../../utils/errors.js` | 错误信息提取 |
| `getMainLoopModel`, `parseUserSpecifiedModel` | `../../utils/model/model.js` | 模型获取和解析 |
| `AutoModeRules`, `buildDefaultExternalSystemPrompt`, `getDefaultExternalAutoModeRules` | `../../utils/permissions/yoloClassifier.js` | 自动模式分类器规则 |
| `getAutoModeConfig` | `../../utils/settings/settings.js` | 获取自动模式配置 |
| `sideQuery` | `../../utils/sideQuery.js` | 侧边查询（用于 AI 分析）|
| `jsonStringify` | `../../utils/slowOperations.js` | JSON 序列化 |

### 类型定义

#### `AutoModeRules`
自动模式规则对象，包含三个分类：
- `allow`: 应自动批准的操作
- `soft_deny`: 应阻止（需要用户确认）的操作
- `environment`: 帮助分类器做决策的环境上下文

### 辅助函数

#### `writeRules(rules: AutoModeRules): void`
将规则对象以格式化的 JSON 输出到 stdout。

#### `formatRulesForCritique(section, userRules, defaultRules): string`
格式化规则用于 AI 分析输出。

**格式示例**：
```
## allow (custom rules replacing defaults)
Custom:
- rule1
- rule2

Defaults being replaced:
- default1
- default2
```

### 核心函数

#### `export function autoModeDefaultsHandler(): void`

处理 `claude auto-mode defaults` 命令。

**功能**：
- 获取并显示默认的自动模式规则
- 使用 `getDefaultExternalAutoModeRules()` 获取规则
- 调用 `writeRules()` 格式化输出

**输出示例**：
```json
{
  "allow": [
    "读取文件",
    "列出目录"
  ],
  "soft_deny": [
    "删除文件",
    "执行命令"
  ],
  "environment": [
    "开发环境"
  ]
}
```

#### `export function autoModeConfigHandler(): void`

处理 `claude auto-mode config` 命令。

**功能**：
显示当前生效的配置，采用 "REPLACE" 语义：
- 如果用户在某分类下有非空规则，完全使用该用户规则
- 如果用户某分类为空或缺失，使用默认规则

**逻辑**：
```typescript
writeRules({
  allow: config?.allow?.length ? config.allow : defaults.allow,
  soft_deny: config?.soft_deny?.length ? config.soft_deny : defaults.soft_deny,
  environment: config?.environment?.length ? config.environment : defaults.environment,
})
```

**使用场景**：
- 验证当前配置是否与预期一致
- 了解哪些默认规则被用户规则替换

#### `export async function autoModeCritiqueHandler(options)`

处理 `claude auto-mode critique` 命令。

**参数**：
- `model` (可选): 用于分析的模型名称

**功能**：
使用 AI 分析和评判用户编写的自定义规则，提供改进建议。

**执行流程**：

1. **检查是否有自定义规则**
   ```typescript
   const hasCustomRules =
     (config?.allow?.length ?? 0) > 0 ||
     (config?.soft_deny?.length ?? 0) > 0 ||
     (config?.environment?.length ?? 0) > 0
   ```
   如果没有自定义规则，提示用户如何添加。

2. **准备模型**
   - 如果指定了模型，解析并使用
   - 否则使用主循环模型

3. **构建提示**
   - 获取默认规则和分类器系统提示
   - 格式化用户规则和默认规则的对比

4. **发送分析请求**
   ```typescript
   response = await sideQuery({
     querySource: 'auto_mode_critique',
     model,
     system: CRITIQUE_SYSTEM_PROMPT,
     skipSystemPromptPrefix: true,
     max_tokens: 4096,
     messages: [...]
   })
   ```

5. **输出结果**
   - 提取文本响应
   - 输出到 stdout

**系统提示 (CRITIQUE_SYSTEM_PROMPT)**：

提示要求 AI 扮演自动模式分类器规则专家，从以下维度评估每个规则：

1. **Clarity（清晰度）**：规则是否明确无歧义？分类器会误解吗？
2. **Completeness（完整性）**：是否有未覆盖的边界情况？
3. **Conflicts（冲突）**：规则之间是否相互矛盾？
4. **Actionability（可执行性）**：规则是否足够具体让分类器执行？

要求简洁且有建设性，只对可改进的规则发表评论。

**错误处理**：
- 分析失败时显示错误信息
- 设置 `process.exitCode = 1`

## 设计要点

1. **REPLACE 语义**：用户规则完全替换对应分类的默认规则，而非合并
2. **AI 驱动的规则审查**：利用 Claude 本身分析和改进用户编写的规则
3. **透明性**：用户可以清楚看到当前生效的配置
4. **模型灵活**：支持指定模型进行分析，可使用更强的模型获得更好的建议
5. **配置优先**：`getAutoModeConfig()` 获取的用户配置优先于默认配置

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/utils/permissions/yoloClassifier.js` | 自动模式规则定义和默认规则 |
| **导入** | `src/utils/settings/settings.js` | 自动模式配置获取 |
| **导入** | `src/utils/model/model.js` | 模型选择和解析 |
| **导入** | `src/utils/sideQuery.js` | 侧边查询功能（用于 AI 分析）|
| **被导入** | `src/main.tsx` | `claude auto-mode` 命令处理（动态导入）|

## 注意事项

1. **动态导入**：模块注释说明它是动态导入的，仅在 `claude auto-mode` 运行时加载
2. **配置层级**：用户配置在设置文件中，与默认规则分离
3. **空规则处理**：空数组被视为 "有意清空该分类"，而非 "使用默认"
4. **分析限制**：分析结果依赖 AI 的理解，可能需要多次迭代优化规则
5. **模型选择**：使用 `--model` 可以指定更强的模型进行分析，但不影响实际自动模式使用的模型
6. **REPLACE vs MERGE**：当前实现是 REPLACE 语义，用户规则完全替换默认规则；这不是 MERGE 语义，不会合并用户和默认规则
