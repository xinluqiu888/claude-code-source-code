# types.ts — 伙伴系统类型定义

> **一句话总结**：定义伙伴（Companion）系统的所有类型、稀有度、物种和外观选项。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/buddy/types.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 149 |
| 主要职责 | 伙伴系统的核心类型定义和常量 |

---

## 功能概述

该文件定义了Claude Code中"伙伴"（Buddy/Companion）功能的所有类型和数据结构。伙伴是一个基于用户ID确定性生成的小精灵角色，显示在输入框旁边，偶尔以对话气泡形式发表评论。

系统设计特点：
- 伙伴的"骨骼"（外观和属性）基于用户ID哈希确定性生成
- 伙伴的"灵魂"（名字和个性）由模型生成并持久化存储
- 用户无法通过编辑配置来伪造稀有度

---

## 核心内容详解

### 稀有度系统

#### `RARITIES` 常量数组
`['common', 'uncommon', 'rare', 'epic', 'legendary']`

#### `RARITY_WEIGHTS` - 生成权重
| 稀有度 | 权重 | 概率 |
|--------|------|------|
| common | 60 | ~60% |
| uncommon | 25 | ~25% |
| rare | 10 | ~10% |
| epic | 4 | ~4% |
| legendary | 1 | ~1% |

#### `RARITY_STARS` - 星级显示
| 稀有度 | 显示 |
|--------|------|
| common | ★ |
| uncommon | ★★ |
| rare | ★★★ |
| epic | ★★★★ |
| legendary | ★★★★★ |

#### `RARITY_COLORS` - 主题颜色映射
| 稀有度 | 颜色键 |
|--------|--------|
| common | inactive |
| uncommon | success |
| rare | permission |
| epic | autoAccept |
| legendary | warning |

### 物种系统

**18个可用物种**：duck（鸭）、goose（鹅）、blob（史莱姆）、cat（猫）、dragon（龙）、octopus（章鱼）、owl（猫头鹰）、penguin（企鹅）、turtle（海龟）、snail（蜗牛）、ghost（幽灵）、axolotl（美西螈）、capybara（水豚）、cactus（仙人掌）、robot（机器人）、rabbit（兔子）、mushroom（蘑菇）、chonk（胖墩）

**编码技巧**：物种名称使用 `String.fromCharCode` 运行时构造，避免与模型代号冲突的静态字符串检查。

### 外观选项

#### `EYES` - 眼睛样式
`['·', '✦', '×', '◉', '@', '°']`

#### `HATS` - 帽子样式
`['none', 'crown', 'tophat', 'propeller', 'halo', 'wizard', 'beanie', 'tinyduck']`

### 属性系统

#### `STAT_NAMES` - 五项属性
`['DEBUGGING', 'PATIENCE', 'CHAOS', 'WISDOM', 'SNARK']`

每项属性值范围：1-100

### 核心类型

#### `CompanionBones` - 确定性部分
基于 `hash(userId)` 生成：
- `rarity`: 稀有度
- `species`: 物种
- `eye`: 眼睛
- `hat`: 帽子
- `shiny`: 是否闪光（1%概率）
- `stats`: 五项属性值

#### `CompanionSoul` - 模型生成部分
- `name`: 名字
- `personality`: 个性描述

#### `Companion` - 完整伙伴
继承 `CompanionBones` + `CompanionSoul`，额外包含：
- `hatchedAt`: 孵化时间戳

#### `StoredCompanion` - 持久化存储
仅存储 `CompanionSoul` + `hatchedAt`，骨骼每次从userId重新生成

---

## 设计要点

1. **防篡改设计**: 骨骼不持久化，防止用户编辑配置伪造稀有度
2. **确定性生成**: 相同userId始终生成相同伙伴
3. **运行时编码**: 物种名称使用char code构造，避免构建时字符串检查
4. **权重系统**: 稀有度使用加权随机，legendary约1%概率

---

## 与其他文件的关系

- **被依赖**: 
  - `companion.ts` - 伙伴生成逻辑
  - `prompt.ts` - 提示词生成
  - `sprites.ts` - 精灵渲染

---

## 注意事项

1. **物种命名**: 某些物种名称可能与内部模型代号冲突，使用char code编码
2. **稀有度保底**: 属性值根据稀有度有最低保底值（common: 5, legendary: 50）
3. **common无帽子**: 只有common稀有度的伙伴帽子强制为'none'
