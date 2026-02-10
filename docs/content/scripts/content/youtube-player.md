---
title: YouTube 播放器
description: 在您的 Nuxt 应用中展示性能优化的 YouTube 视频。
links:
  - label: useScriptYouTubePlayer
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/youtube-player.ts
    size: xs
  - label: "<ScriptYouTubePlayer>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptYouTubePlayer.vue
    size: xs
---

[YouTube](https://youtube.com/) 是一个视频托管平台，允许您上传和分享视频。

Nuxt Scripts 提供了 `useScriptYouTubePlayer` 组合函数和无头（headless）的 `ScriptYouTubePlayer` 组件，用于与 YouTube 播放器进行交互。

## 类型

要使用带有完整 TypeScript 支持的 YouTube，您需要安装 `@types/youtube` 依赖。

```bash
pnpm add -D @types/youtube
```

## ScriptYouTubePlayer

`ScriptYouTubePlayer` 组件是基于 `useScriptYouTubePlayer` 组合函数的封装。它提供了一种简单的方法来在您的 Nuxt 应用中嵌入 YouTube 视频。

它通过利用 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 进行性能优化，仅在特定元素发生事件时加载 YouTube 播放器。

默认情况下，它会在 `mousedown` 事件时加载。

### 演示

::code-group

:youtube-demo{label="输出"}

```vue [Input]
<script setup lang="ts">
const isLoaded = ref(false)
const isPlaying = ref(false)
const video = ref()
async function play() {
  await video.value.player.playVideo()
}
function stateChange(event) {
  isPlaying.value = event.data === 1
}
</script>

<template>
  <div>
    <div class="flex items-center justify-center p-5">
      <ScriptYouTubePlayer ref="video" video-id="d_IFKP1Ofq0" @ready="isLoaded = true" @state-change="stateChange">
        <template #awaitingLoad>
          <div class="absolute left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2 h-[48px] w-[68px]">
            <svg height="100%" version="1.1" viewBox="0 0 68 48" width="100%"><path d="M66.52,7.74c-0.78-2.93-2.49-5.41-5.42-6.19C55.79,.13,34,0,34,0S12.21,.13,6.9,1.55 C3.97,2.33,2.27,4.81,1.48,7.74C0.06,13.05,0,24,0,24s0.06,10.95,1.48,16.26c0.78,2.93,2.49,5.41,5.42,6.19 C12.21,47.87,34,48,34,48s21.79-0.13,27.1-1.55c2.93-0.78,4.64-3.26,5.42-6.19C67.94,34.95,68,24,68,24S67.94,13.05,66.52,7.74z" fill="#f00" /><path d="M 45,24 27,14 27,34" fill="#fff" /></svg>
          </div>
        </template>
      </ScriptYouTubePlayer>
    </div>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击以加载" description="点击视频将加载 Youtube iframe 并开始播放视频。" />
      <UButton v-if="isLoaded && !isPlaying" @click="play">
        播放视频
      </UButton>
    </div>
  </div>
</template>
```

::

### Props

`ScriptYouTubePlayer` 组件接受以下 props：

- `trigger`：加载 YouTube 播放器的触发事件。默认是 `mousedown`。欲了解更多信息，请参阅 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。
- `placeholderAttrs`：占位图片的属性。默认是 `{ loading: 'lazy' }`。
- `aboveTheFold`：优化首屏内容的占位图片。默认是 `false`。
- `placeholderObjectFit`：占位图片的 `object-fit` CSS 属性。默认是 `cover`。对非16:9视频（如 YouTube Shorts）很有用。

`playerVars` prop 支持所有来自 [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference) 的脚本选项，请参考 [支持的参数](https://developers.google.com/youtube/player_parameters#Parameters) 以获取完整文档。

```ts
export interface YouTubeProps {
  // YouTube 播放器
  videoId: string
  playerVars?: YT.PlayerVars
  width?: number
  height?: number
  placeholderObjectFit?: 'cover' | 'contain' | 'fill' | 'none' | 'scale-down'
}
```

### 隐私

`<YoutubePlayer>` 组件默认隐私友好，视频托管地址设置为 `https://www.youtube-nocookie.com`。

若要修改此行为，可以将 `host` prop 设置为 `https://www.youtube.com`。

```vue
<ScriptYouTubePlayer video-id="d_IFKP1Ofq0" :player-options="{ host: 'https://www.youtube.com' }" />
```

### 占位符

YouTube 播放器的占位图片是 1280x720 大小的 webp，默认启用懒加载。

若想修改占位符大小，可以设置 `thumbnailSize` prop；如果您更喜欢使用 jpg 格式，可以将 `webp` prop 设为 `false`。

```vue
<ScriptYouTubePlayer video-id="d_IFKP1Ofq0" thumbnail-size="maxresdefault" />
```

如果您需要对占位符进行精细控制，可以设置 `placeholderAttrs` prop，或使用 `#placeholder` 插槽完全覆盖。

#### 提前加载

如果您的视频处于首屏位置，请调整此行为，或考虑使用 `#placeholder` 插槽自定义占位图片。

::code-group

```vue [Placeholder Attrs]
<ScriptYouTubePlayer above-the-fold />
```

```vue [Placeholder Slot]
<ScriptYouTubePlayer>
  <template #placeholder="{ placeholder }">
    <img :src="placeholder" alt="视频占位符">
  </template>
</ScriptYouTubePlayer>
```

::

### 组件 API

请查看 [Facade 组件 API](/docs/guides/facade-components#facade-components-api) 获取完整的 props、事件和插槽说明。

### 事件

`ScriptYouTubePlayer` 组件会发出所有来自 YouTube 播放器 SDK 的事件。完整文档请见 [播放器事件](https://developers.google.com/youtube/iframe_api_reference#Events)。

```ts
const emits = defineEmits<{
  'ready': [e: YT.PlayerEvent]
  'state-change': [e: YT.OnStateChangeEvent, target: YT.Player]
  'playback-quality-change': [e: YT.OnPlaybackQualityChangeEvent, target: YT.Player]
  'playback-rate-change': [e: YT.OnPlaybackRateChangeEvent, target: YT.Player]
  'error': [e: YT.OnErrorEvent, target: YT.Player]
  'api-change': [e: YT.PlayerEvent, target: YT.Player]
}>()
```

### 插槽

由于该组件是无头设计，提供了一些插槽让您能在加载视频之前自定义播放器的内容。

**default**

默认插槽用于显示始终可见的内容。

```vue
<template>
  <ScriptYouTubePlayer video-id="d_IFKP1Ofq0">
    <div class="bg-blue-500 text-white p-5">
      Nuxt 视频
    </div>
  </ScriptYouTubePlayer>
</template>
```

**awaitingLoad**

该插槽用于显示视频加载期间的内容。

```vue
<template>
  <ScriptYouTubePlayer video-id="d_IFKP1Ofq0">
    <template #awaitingLoad>
      <div class="bg-blue-500 text-white p-5">
        点击播放！
      </div>
    </template>
  </ScriptYouTubePlayer>
</template>
```

**loading**

该插槽用于显示视频加载时的内容。

```vue
<template>
  <ScriptYouTubePlayer video-id="d_IFKP1Ofq0">
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptYouTubePlayer>
</template>
```

**placeholder**

该插槽用于在视频加载前显示占位图片。默认显示视频的 YouTube 缩略图。您可以自定义显示内容。

```vue
<template>
  <ScriptYouTubePlayer video-id="d_IFKP1Ofq0">
    <template #placeholder="{ placeholder }">
      <img :src="placeholder" alt="视频占位符">
    </template>
  </ScriptYouTubePlayer>
</template>
```

## useScriptYouTubePlayer

`useScriptYouTubePlayer` 组合函数让您可以更细粒度地控制 YouTube 播放器 SDK。它提供了一种加载 YouTube 播放器 SDK 并以编程方式交互的方式。

```ts
export function useScriptYouTubePlayer<T extends YouTubePlayerApi>(_options?: YouTubePlayerInput) {}
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南，了解更多高级用法。

### YouTubePlayerApi

```ts
/// <reference types="youtube" />
export interface YouTubePlayerApi {
  YT: typeof YT
}
```

## 示例

加载 YouTube 播放器 SDK 并以编程方式与之交互。

```vue
<script setup lang="ts">
const video = ref()
const { onLoaded } = useScriptYouTubePlayer()

const player = ref(null)
onLoaded(async ({ YT }) => {
  // 我们需要等待内部 YouTube API 准备好
  const YouTube = await YT
  await new Promise<void>((resolve) => {
    if (typeof YT.Player === 'undefined')
      YouTube.ready(resolve)
    else
      resolve()
  })
  // 加载 API
  player.value = new YT.Player(video.value, {
    videoId: 'd_IFKP1Ofq0'
  })
})
function play() {
  player.value?.playVideo()
}
</script>

<template>
  <div>
    <div ref="video" />
    <button @click="play">
      播放
    </button>
  </div>
</template>
```