---
title: Vimeo 播放器
description: 在你的 Nuxt 应用中展示性能优化的 Vimeo 视频。
links:
  - label: useScriptVimeoPlayer
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/vimeo-player.ts
    size: xs
  - label: "<ScriptVimeoPlayer>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptVimeoPlayer.vue
    size: xs
---

[Vimeo](https://vimeo.com/) 是一个视频托管平台，允许你上传和分享视频。

Nuxt Scripts 提供了一个 `useScriptVimeoPlayer` 组合式函数和一个无 UI 的 `ScriptVimeoPlayer` 组件，用于与 Vimeo 播放器交互。

## 类型

若想使用具备完整 TypeScript 支持的视频播放器，你需要安装 `@types/vimeo__player` 依赖。

```bash
pnpm add -D @types/vimeo__player
```

## ScriptVimeoPlayer

`ScriptVimeoPlayer` 组件是 `useScriptVimeoPlayer` 组合式函数的封装。它提供了一种简便方式在你的 Nuxt 应用中嵌入 Vimeo 视频。

通过利用 [元素事件触发](/docs/guides/script-triggers#element-event-triggers)，仅在特定元素事件发生时加载 Vimeo 播放器，从而优化性能。

默认情况下，它会在 `mousedown` 事件时加载。

### 演示

::code-group

:vimeo-demo{label="输出"}

```vue [Input]
<script setup lang="ts">
const isLoaded = ref(false)
const isPlaying = ref(false)
const video = ref()
async function play() {
  await video.value.play()
}
</script>

<template>
  <div>
    <div class="flex items-center justify-center p-5">
      <ScriptVimeoPlayer :id="331567154" ref="video" class="group" @loaded="isLoaded = true" @play="isPlaying = true" @pause="isPlaying = false">
        <template #awaitingLoad>
          <div class="absolute left-1/2 top-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-blue-500 group-hover:bg-blue-700 transition rounded px-6 py-2">
            <svg width="24" height="24" viewBox="0 0 24 24" class="w-10 h-10 text-white" xmlns="http://www.w3.org/2000/svg"><path d="M19 12C19 12.3557 18.8111 12.6846 18.5039 12.8638L6.50387 19.8638C6.19458 20.0442 5.81243 20.0455 5.50194 19.8671C5.19145 19.6888 5 19.3581 5 19L5 5C5 4.64193 5.19145 4.3112 5.50194 4.13286C5.81243 3.95452 6.19458 3.9558 6.50387 4.13622L18.5039 11.1362C18.8111 11.3154 19 11.6443 19 12Z" fill="currentColor" /></svg>
          </div>
        </template>
      </ScriptVimeoPlayer>
    </div>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击视频！" description="点击视频会加载 Vimeo iframe 并开始播放视频。" />
      <UButton v-if="isLoaded && !isPlaying" @click="play">
        播放视频
      </UButton>
    </div>
  </div>
</template>
```

::

### 属性（Props）

`ScriptVimeoPlayer` 组件接收以下属性：

- `trigger`：触发加载 Vimeo 播放器的事件，默认是 `mousedown`。更多信息请参见 [元素事件触发](/docs/guides/script-triggers#element-event-triggers)。
- `aboveTheFold`：优化折叠上方内容的占位图像，默认是 `false`。
- `rootAttrs`：覆盖自动设置的根节点属性。
- `placeholderAttrs`：占位图像的属性，默认是 `{ loading: 'lazy' }`。
- `id`：`vimeoOptions.id` 的简写。
- `url`：`vimeoOptions.url` 的简写。
- `vimeoOptions`：支持 Player SDK 的所有选项，完整文档请参考 [嵌入选项](https://developer.vimeo.com/player/sdk/embed)。

```ts
interface VimeoPlayerProps {
  id: number | undefined
  url?: string | undefined
  autopause?: boolean | undefined
  autoplay?: boolean | undefined
  background?: boolean | undefined
  byline?: boolean | undefined
  color?: string | undefined
  controls?: boolean | undefined
  dnt?: boolean | undefined
  height?: number | undefined
  interactive_params?: string | undefined
  keyboard?: boolean | undefined
  loop?: boolean | undefined
  maxheight?: number | undefined
  maxwidth?: number | undefined
  muted?: boolean | undefined
  pip?: boolean | undefined
  playsinline?: boolean | undefined
  portrait?: boolean | undefined
  responsive?: boolean | undefined
  speed?: boolean | undefined
  quality?: VimeoVideoQuality | undefined
  texttrack?: string | undefined
  title?: boolean | undefined
  transparent?: boolean | undefined
  width?: number | undefined
}
```

#### 预加载（Eager Loading）占位图

Vimeo 视频的占位图默认是懒加载的。如果你的视频是折叠上方内容，建议修改此行为，或者使用 `#placeholder` 插槽自定义占位图。

::code-group

```vue [Placeholder Attrs]
<ScriptVimeoPlayer above-the-fold />
```

```vue [Placeholder Slot]
<ScriptVimeoPlayer>
  <template #placeholder="{ placeholder }">
    <img :src="placeholder" alt="视频占位图">
  </template>
</ScriptVimeoPlayer>
```

::

### 组件 API

完整的属性、事件和插槽信息请参阅 [Facade 组件 API](/docs/guides/facade-components#facade-components-api)。

### 事件

`ScriptVimeoPlayer` 组件会触发 Vimeo 播放器 SDK 的所有事件。完整文档请查看 [播放器事件](https://developer.vimeo.com/player/sdk/reference#about-player-events)。

```ts
const emits = defineEmits<{
  play: [e: EventMap['play'], player: Player]
  playing: [e: EventMap['playing'], player: Player]
  pause: [e: EventMap['pause'], player: Player]
  ended: [e: EventMap['ended'], player: Player]
  timeupdate: [e: EventMap['timeupdate'], player: Player]
  progress: [e: EventMap['progress'], player: Player]
  seeking: [e: EventMap['seeking'], player: Player]
  seeked: [e: EventMap['seeked'], player: Player]
  texttrackchange: [e: EventMap['texttrackchange'], player: Player]
  chapterchange: [e: EventMap['chapterchange'], player: Player]
  cuechange: [e: EventMap['cuechange'], player: Player]
  cuepoint: [e: EventMap['cuepoint'], player: Player]
  volumechange: [e: EventMap['volumechange'], player: Player]
  playbackratechange: [e: EventMap['playbackratechange'], player: Player]
  bufferstart: [e: EventMap['bufferstart'], player: Player]
  bufferend: [e: EventMap['bufferend'], player: Player]
  error: [e: EventMap['error'], player: Player]
  loaded: [e: EventMap['loaded'], player: Player]
  durationchange: [e: EventMap['durationchange'], player: Player]
  fullscreenchange: [e: EventMap['fullscreenchange'], player: Player]
  qualitychange: [e: EventMap['qualitychange'], player: Player]
  camerachange: [e: EventMap['camerachange'], player: Player]
  resize: [e: EventMap['resize'], player: Player]
  enterpictureinpicture: [e: EventMap['enterpictureinpicture'], player: Player]
  leavepictureinpicture: [e: EventMap['leavepictureinpicture'], player: Player]
}>()
```

### 插槽（Slots）

由于该组件为无 UI 组件，你可以通过多个插槽自定义播放器的加载前展示。

**default**

默认插槽用来显示始终可见的内容。

```vue
<template>
  <ScriptVimeoPlayer :id="331567154">
    <div class="bg-blue-500 text-white p-5">
      视频提供者：NuxtJS
    </div>
  </ScriptVimeoPlayer>
</template>
```

**awaitingLoad**

该插槽用于视频加载时显示内容。

```vue
<template>
  <ScriptVimeoPlayer :id="331567154">
    <template #awaitingLoad>
      <div class="bg-blue-500 text-white p-5">
        点击播放！
      </div>
    </template>
  </ScriptVimeoPlayer>
</template>
```

**loading**

该插槽用于视频加载过程中显示的内容。

```vue
<template>
  <ScriptVimeoPlayer :id="331567154">
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptVimeoPlayer>
</template>
```

**placeholder**

该插槽用于视频加载前显示的占位图。默认情况下，它会展示 Vimeo 视频的缩略图。你可以根据需求自定义显示。

```vue
<template>
  <ScriptVimeoPlayer :id="331567154">
    <template #placeholder="{ placeholder }">
      <img :src="placeholder" alt="视频占位图">
    </template>
  </ScriptVimeoPlayer>
</template>
```

## useScriptVimeoPlayer

`useScriptVimeoPlayer` 组合式函数让你可以更细粒度地控制 Vimeo 播放器 SDK。它提供了加载 Vimeo 播放器 SDK 并以编程方式交互的方式。

```ts
export function useScriptVimeoPlayer<T extends VimeoPlayerApi>(_options?: VimeoPlayerInput) {}
```

请参考 [Registry Scripts](/docs/guides/registry-scripts) 指南以获取更多高级用法。

### VimeoPlayerApi

```ts
export interface VimeoPlayerApi {
  Vimeo: {
    Player: ScriptVimeoPlayer
  }
}
```

## 示例

加载 Vimeo 播放器 SDK 并以编程方式进行交互。

```vue
<script setup lang="ts">
const video = ref()
const { onLoaded } = useScriptVimeoPlayer()

let player
onLoaded(({ Vimeo }) => {
  player = new Vimeo.Player(video.value, {
    id: 331567154
  })
})
</script>

<template>
  <div>
    <div ref="video" />
    <button @click="player.play()">
      播放
    </button>
  </div>
</template>
```
