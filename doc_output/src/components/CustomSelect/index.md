# index.ts

CustomSelect 模块的入口文件，统一导出所有公共 API。

## 基本信息

- **文件路径**: `src/components/CustomSelect/index.ts`
- **类型**: 模块入口文件

## 导出内容

| 导出项 | 来源文件 | 说明 |
|--------|----------|------|
| SelectMulti | `./SelectMulti.js` | 多选组件 |
| OptionWithDescription | `./select.js` | 选项类型定义 |
| Select | `./select.js` | 单选组件 |
| SelectProps | `./select.js` | 单选组件属性 |
| useSelectState | `./select.js` | 选择状态 hook |

## 使用方式

```typescript
// 导入单选组件和类型
import { Select, type OptionWithDescription } from './CustomSelect/index.js'

// 导入多选组件
import { SelectMulti } from './CustomSelect/index.js'
```

## 备注

- 该文件作为模块的公共 API 入口，集中管理导出
- 未来添加新组件或 hook 时，应在此文件中添加导出
