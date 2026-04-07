# voice.tsx — 语音状态管理上下文

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/voice.tsx`
- **类型**: React Context 组件 + 状态存储
- **语言**: TypeScript + React

## 功能概述

提供语音功能的状态管理，包括录音状态、错误处理、临时转录文本、音频电平和预热状态。使用自定义存储模式 (类似 Zustand) 实现高效的状态订阅和更新。

## 核心内容详解

### VoiceState 类型

```typescript
export type VoiceState = {
  voiceState: 'idle' | 'recording' | 'processing'
  voiceError: string | null
  voiceInterimTranscript: string
  voiceAudioLevels: number[]
  voiceWarmingUp: boolean
}
```

### 主要导出

1. **VoiceProvider** — 语音状态提供者
   - 使用 useState 创建单例 VoiceStore
   - 稳定的上下文值避免 Provider 重新渲染

2. **useVoiceState(selector)** — 订阅语音状态切片
   - 仅当选择的值变化时重新渲染
   - 使用 Object.is 进行相等比较

3. **useSetVoiceState()** — 获取状态设置器
   - 返回稳定的 setState 引用
   - 永远不会导致重新渲染
   - 同步更新，可立即读取新值

4. **useGetVoiceState()** — 获取状态读取器
   - 返回 getState 函数
   - 不引起重新渲染
   - 用于事件处理器中读取状态

### 默认状态

```typescript
const DEFAULT_STATE: VoiceState = {
  voiceState: 'idle',
  voiceError: null,
  voiceInterimTranscript: '',
  voiceAudioLevels: [],
  voiceWarmingUp: false
}
```

## 设计要点

1. **自定义存储模式**:
   - 使用 `createStore` 创建类似 Zustand 的存储
   - 支持订阅/发布模式
   - 使用 useSyncExternalStore 进行 React 集成

2. **切片订阅**:
   - useVoiceState 接受 selector 函数
   - 仅订阅需要的部分状态
   - 避免不必要地重新渲染

3. **稳定引用**:
   - useSetVoiceState 返回稳定的 setState
   - useGetVoiceState 返回稳定的 getState
   - 可用于依赖数组而不触发效果

4. **同步更新**:
   - setState 是同步的
   - 调用后可立即使用 getVoiceState 读取新值
   - VoiceKeybindingHandler 依赖此行为

5. **单例 Store**:
   - Provider 只创建一次 store
   - useState 的初始化函数确保单例

## 与其他文件的关系

- **../state/store.js**: createStore 实现
- **VoiceKeybindingHandler**: 使用 useSetVoiceState 和 useGetVoiceState
- **语音 UI 组件**: 使用 useVoiceState 订阅特定状态

## 注意事项

1. 必须在 VoiceProvider 内部使用语音 hooks
2. useVoiceState 使用 Object.is 进行相等比较
3. store.setState 同步更新，可立即读取
4. Provider 本身不会因状态变化而重新渲染
5. 错误边界：useVoiceStore 在没有 Provider 时抛出错误
