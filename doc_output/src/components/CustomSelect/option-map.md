# option-map.ts

选项映射类，用于高效管理选项的双向链表关系。

## 基本信息

- **文件路径**: `src/components/CustomSelect/option-map.ts`
- **导出类**: OptionMap<T>
- **类型**: TypeScript 类

## 功能概述

OptionMap 继承自 Map，用于存储选项值到选项项的映射。它在构建时自动创建选项的双向链表关系，提供 O(1) 时间复杂度的选项查找和导航。

## 核心内容

### OptionMapItem<T> 类型

```typescript
type OptionMapItem<T> = {
  label: ReactNode        // 选项标签
  value: T                // 选项值
  description?: string    // 选项描述
  previous: OptionMapItem<T> | undefined  // 上一个选项
  next: OptionMapItem<T> | undefined      // 下一个选项
  index: number           // 选项索引
}
```

### 类属性

| 属性 | 类型 | 说明 |
|------|------|------|
| first | OptionMapItem<T> \| undefined | 第一个选项 |
| last | OptionMapItem<T> \| undefined | 最后一个选项 |

### 构造函数

```typescript
constructor(options: OptionWithDescription<T>[])
```

构建 OptionMap 时执行以下操作：
1. 遍历选项数组
2. 为每个选项创建 OptionMapItem
3. 建立双向链表关系（previous 和 next）
4. 记录 first 和 last 选项
5. 将所有项存入 Map

## 设计要点

- 继承 Map 类，保留 Map 的所有方法（get、has、forEach 等）
- 使用双向链表支持向前和向后导航
- 在构造函数中一次性构建所有关系，后续查询为 O(1)
- 不可变性：first 和 last 属性为 readonly

## 与其他文件的关系

- **被导入**:
  - `use-select-navigation.ts` - 导航 hook 使用 OptionMap 管理选项关系

- **导入依赖**:
  - `./select.js` - OptionWithDescription 类型

## 备注

- 适用于需要频繁前后导航的场景
- 相比在数组中查找，链表导航效率更高
- 选项变化时需要重新创建 OptionMap 实例
