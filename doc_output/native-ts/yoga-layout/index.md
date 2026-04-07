# yoga-layout/index.ts — 纯TypeScript Yoga Flexbox布局引擎

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/native-ts/yoga-layout/index.ts`
- **类型**: TypeScript 模块
- **作用**: Meta Yoga布局引擎的纯TypeScript移植

## 功能概述

本模块是Meta Yoga Flexbox引擎的纯TypeScript实现，匹配`yoga-layout/load` API表面。原始C++代码仅CalculateLayout.cpp就有约2500行，此移植是简化但完整的单遍flexbox实现，覆盖Ink实际使用的所有功能。

## 核心内容详解

### 支持的完整功能

**Flex核心**: flex-direction、flex-grow/shrink/basis、align-items/self、justify-content
**间距**: margin、padding、border、gap
**尺寸**: width/height、min/max（支持point、percent、auto）
**定位**: relative/absolute、display: flex/none、measure functions（文本节点）

**规格兼容（Ink未使用但已实现）**:
- margin: auto（主轴和交叉轴）
- min/max约束下的flex clamping
- flex-wrap（多行flex）
- align-content（多行定位）
- display: contents（子元素提升到祖父节点）
- baseline alignment

**未实现（Ink未使用）**:
- aspect-ratio
- box-sizing: content-box
- RTL方向（Ink始终使用LTR）

### 核心类

#### `Node` 类

Yoga布局节点，支持完整的flexbox属性和布局计算。

**样式属性（输入）**:
```typescript
type Style = {
  direction: Direction
  flexDirection: FlexDirection
  justifyContent: Justify
  alignItems/alignSelf/alignContent: Align
  flexWrap: Wrap
  display: Display
  positionType: PositionType
  flexGrow/flexShrink: number
  flexBasis: Value
  margin/padding/border/position: Value[]  // 9边数组
  gap: Value[]  // 3间距数组
  width/height/minWidth/minHeight/maxWidth/maxHeight: Value
}
```

**布局结果（输出）**:
```typescript
type Layout = {
  left: number
  top: number
  width: number
  height: number
  border: [number, number, number, number]
  padding: [number, number, number, number]
  margin: [number, number, number, number]
}
```

**快速路径标志**（性能优化）:
- `_hasAutoMargin`: 是否有auto margin
- `_hasPosition`: 是否有position insets
- `_hasPadding/Border/Margin`: 是否有对应边距

### 缓存系统

**单槽缓存** (`_hasL`/`_hasM`):
- 存储layout/measure调用的输入输出
- 相同输入直接返回缓存结果

**多槽缓存** (`_cIn`/`_cOut`，4个槽位):
- 存储4组不同的输入输出
- 处理scroll场景下同一clean子节点收到N>1种不同输入

**flexBasis缓存** (`_fbBasis`):
- 存储computeFlexBasis结果
- 容器inner dimensions未变时跳过递归

**代际标记** (`_generation`):
- 每次calculateLayout递增
- 同一代的缓存条目即使isDirty也为新鲜
- 避免virtual scroll时2^depth次重复计算

### 核心算法流程

```typescript
calculateLayout(ownerWidth?, ownerHeight?, direction?): void
```

**布局节点算法** (`layoutNode`):

1. **缓存检查**: 匹配输入输出直接返回
2. **解析边距**: padding/border/margin（支持%）
3. **解析尺寸**: width/height（支持%和auto）
4. **应用min/max约束**
5. **测量函数节点**: 调用measureFunc获取尺寸
6. **叶子节点**: 直接计算尺寸
7. **容器节点**:
   - 分区flow/absolute子节点
   - 处理display: contents
   - **STEP 1**: 计算flex-basis并分行
   - **STEP 2+3**: 每行解析flex长度并测量交叉尺寸
   - **STEP 4**: 确定容器交叉尺寸
   - **STEP 5**: 主轴定位（justify）
   - **STEP 6**: 交叉轴定位（align）
   - **STEP 7**: 绝对定位子节点
8. **提交缓存输出并写入缓存**

### 边缘解析

Yoga使用9边模型（物理边+逻辑边）：
```typescript
const EDGE_LEFT = 0, EDGE_TOP = 1, EDGE_RIGHT = 2, EDGE_BOTTOM = 3

// 9边索引: Left(0), Top(1), Right(2), Bottom(3),
//           Start(4), End(5), Horizontal(6), Vertical(7), All(8)
```

解析优先级（以Left为例）:
```
edges[0] (Left) → edges[6] (Horizontal) → edges[8] (All) → edges[4] (Start)
```

### 性能计数器

```typescript
export function getYogaCounters(): {
  visited: number    // 访问节点数
  measured: number   // measure调用数
  cacheHits: number  // 缓存命中数
  live: number       // 存活节点数
}
```

## 设计要点

1. **Hot path优化**: `_hasXxx`标志跳过不必要的计算
2. **批量边缘解析**: `resolveEdges4Into`一次解析4边，避免多次函数调用
3. **NaN安全相等**: `a === b || (a !== a && b !== b)`
4. **预分配数组**: layout.border/padding/margin预分配避免GC
5. **Float64Array缓存**: 多槽缓存使用TypedArray提高性能

## 与其他文件的关系

- **被 Ink 布局使用**: 终端UI使用Yoga进行flex布局
- **导入 enums.ts**: 使用枚举定义
- **替代原生模块**: 取代原生Yoga绑定

## 注意事项

1. **LTR假设**: Ink始终使用LTR，Start/End映射到Left/Right
2. **像素取整**: `roundLayout`在最后进行像素网格对齐
3. **measure函数**: 文本节点需要设置measureFunc
4. **内存管理**: 调用`free()`或`freeRecursive()`释放节点
5. **Dirty传播**: markDirty()向上传播到根节点
