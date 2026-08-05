---
title: YouTube 播放器
description: 在您的 Nuxt 应用中展示性能优化的 YouTube 视频。
links:
  - label: useScriptYouTubePlayer
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/youtube-player.ts
    size: xs
  - label: "<ScriptYouTubePlayer>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptYouTubePlayer.vue
    size: xs
---

[YouTube](https://youtube.com/) 托管视频并提供 iframe 播放器 API。

Nuxt Scripts 提供了一个用于控制 YouTube 播放器的 [`useScriptYouTubePlayer()`{lang="ts"}](/scripts/youtube-player){lang="ts"} 组合式函数，以及一个无头 [`<ScriptYouTubePlayer>`{lang="html"}](/scripts/youtube-player){lang="html"} 组件。

::script-stats
::

::script-docs
::

## 类型

安装 `@types/youtube` 以获得完整的 TypeScript 支持。

```bash
pnpm add -D @types/youtube
```

## [`<ScriptYouTubePlayer>`{lang="html"}](/scripts/youtube-player){lang="html"}

[`<ScriptYouTubePlayer>`{lang="html"}](/scripts/youtube-player){lang="html"} 使用懒加载缩略图和无头播放器 UI 封装了 [`useScriptYouTubePlayer()`{lang="ts"}](/scripts/youtube-player){lang="ts"}。

[元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 会延迟加载 iframe API 和播放器，直到配置的事件触发。

默认事件是 `mousedown`。

### 演示

::code-group

:youtube-demo{label="输出"}

```vue [输入]
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
            <svg height="100%" version="1.1" viewBox="0 0 68 48" width="100%"><path d="M66.52,7.74c-0.78-2.93-2.49-5.41-5.42-6.19C55.79,.13,34,0,34,0S12.21,.13,6.9,1.55 C3.97,2.33,2.27,4.81,1.48,7.74C0.06,13.05,0,24,0,24s.06,10.95,1.48,16.26c0.78,2.93,2.49,5.41,5.42,6.19 C12.21,47.87,34,48,34,48s21.79-0.13,27.1-1.55c2.93-0.78,4.64-3.26,5.42-6.19C67.94,34.95,68,24,68,24S67.94,13.05,66.52,7.74z" fill="#f00" /><path d="M 45,24 27,14 27,34" fill="#fff" /></svg>
          </div>
        </template>
      </ScriptYouTubePlayer>
    </div>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击加载" description="点击视频将加载 YouTube iframe 并开始播放。" />
      <UButton v-if="isLoaded && !isPlaying" @click="play">
        播放视频
      </UButton>
    </div>
  </div>
</template>
```

::

### 属性

`<ScriptYouTubePlayer>`{lang="html"} 组件默认使用 YouTube 的增强隐私保护主机 `https://www.youtube-nocookie.com`。详情请参阅 YouTube 的[嵌入说明](https://support.google.com/youtube/answer/171780)。

如需使用启用 Cookie 的标准主机，请设置 `cookies` 属性。

```vue
<ScriptYouTubePlayer video-id="d_IFKP1Ofq0" cookies />
```

### 占位符

YouTube 播放器占位符是一张 1280x720 的 WebP 图片，默认采用懒加载。

设置 `thumbnailSize` 可更改占位符尺寸。将 `webp` 设置为 `false` 可使用 JPEG 缩略图。

```vue
<ScriptYouTubePlayer video-id="d_IFKP1Ofq0" thumbnail-size="maxresdefault" />
```

如需更精细的控制，请设置 `placeholderAttrs`，或通过 `#placeholder` 插槽替换图片。

#### 提前加载

对于首屏视频，可以立即加载缩略图，或通过 `#placeholder` 插槽替换缩略图。

::code-group

```vue [占位符属性]
<ScriptYouTubePlayer above-the-fold />
```

```vue [占位符插槽]
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

该组件会转发以下六个事件。在运行时，每个处理函数只会接收此处所示的事件对象。加载 iframe API 失败时，也会触发不带参数的 `error` 事件。组件当前的 TypeScript 声明为其中五个事件列出了第二个 `YT.Player` 参数，但实际上不会发出该参数。YouTube API 还定义了 `onAutoplayBlocked`，但组件目前不会转发该事件。有关负载详情，请参阅[播放器事件](https://developers.google.com/youtube/iframe_api_reference#Events)。

```ts
const emits = defineEmits<{
  'ready': [e: YT.PlayerEvent]
  'state-change': [e: YT.OnStateChangeEvent]
  'playback-quality-change': [e: YT.OnPlaybackQualityChangeEvent]
  'playback-rate-change': [e: YT.OnPlaybackRateChangeEvent]
  'error': [e: YT.OnErrorEvent]
  'api-change': [e: YT.PlayerEvent]
}>()
```

### 插槽

使用插槽控制播放器周围的门面。

**default**

始终可见。

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

组件等待元素触发器时显示。

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

iframe API 加载时显示。

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

替换默认的 YouTube 缩略图。该插槽会接收计算出的 `placeholder` URL。

```vue
<template>
  <ScriptYouTubePlayer video-id="d_IFKP1Ofq0">
    <template #placeholder="{ placeholder }">
      <img :src="placeholder" alt="视频占位符">
    </template>
  </ScriptYouTubePlayer>
</template>
```

## [`useScriptYouTubePlayer()`{lang="ts"}](/scripts/youtube-player){lang="ts"}

当你需要加载 iframe API 并以编程方式创建播放器时，请使用 [`useScriptYouTubePlayer()`{lang="ts"}](/scripts/youtube-player){lang="ts"}。

```ts
export function useScriptYouTubePlayer<T extends YouTubePlayerApi>(_options: YouTubePlayerInput) {}
```

有关触发器、代理和其他脚本选项，请参阅[注册表脚本](/docs/guides/registry-scripts)。

::script-types
::

## 示例

加载 YouTube 播放器 SDK 并以编程方式与之交互。

```vue
<script setup lang="ts">
const video = ref()
const { onLoaded } = useScriptYouTubePlayer({})

const player = ref(null)
onLoaded(async ({ YT }) => {
  // 我们需要等待内部 YouTube API 准备好
  const YouTube = await YT
  await new Promise<void>((resolve) => {
    if (typeof YouTube.Player === 'undefined')
      YouTube.ready(resolve)
    else
      resolve()
  })
  // 加载 API
  player.value = new YouTube.Player(video.value, {
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
