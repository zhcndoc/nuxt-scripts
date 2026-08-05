---
title: Vimeo 播放器
description: 在你的 Nuxt 应用中展示性能优化的 Vimeo 视频。
links:
  - label: useScriptVimeoPlayer
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/vimeo-player.ts
    size: xs
  - label: "<ScriptVimeoPlayer>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptVimeoPlayer.vue
    size: xs
---

[Vimeo](https://vimeo.com/) 托管视频并提供 JavaScript 播放器 API。

Nuxt Scripts 提供了一个 [`useScriptVimeoPlayer()`{lang="ts"}](/scripts/vimeo-player){lang="ts"} 可组合函数，以及一个无头 [`<ScriptVimeoPlayer>`{lang="html"}](/scripts/vimeo-player){lang="html"} 组件，用于与 Vimeo 播放器交互。

::script-stats
::

::script-docs
::

## 类型

安装包含自身类型的 `@vimeo/player`，以获得完整的 TypeScript 支持。

```bash
pnpm add -D @vimeo/player
```

## [`<ScriptVimeoPlayer>`{lang="html"}](/scripts/vimeo-player){lang="html"}

[`<ScriptVimeoPlayer>`{lang="html"}](/scripts/vimeo-player){lang="html"} 使用延迟占位符和无头播放器 UI 封装了 [`useScriptVimeoPlayer()`{lang="ts"}](/scripts/vimeo-player){lang="ts"}。

[元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 会将 Vimeo 播放器的加载延迟到配置的事件触发之后。

默认事件为 `mousedown`。

::callout{color="amber"}
播放器接受 `id` 或 `url`，但当前的 oEmbed 缩略图请求只读取 `id`。当你仅传入 `url` 时，请提供自己的 `#placeholder` 内容，或同时传入数字格式的视频 ID。
::

### 演示

::code-group

:vimeo-demo{label="输出"}

```vue [输入]
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
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击视频！" description="点击视频将加载 Vimeo iframe 并开始播放视频。" />
      <UButton v-if="isLoaded && !isPlaying" @click="play">
        播放视频
      </UButton>
    </div>
  </div>
</template>
```

::

### 属性（Props）

默认情况下，Vimeo 视频占位图会延迟加载。对于首屏视频，请立即加载图像，或通过 `#placeholder` 插槽替换它。

::code-group

```vue [占位图属性]
<ScriptVimeoPlayer above-the-fold />
```

```vue [占位图插槽]
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

该组件会转发以下 Vimeo Player SDK 事件。有关负载详情，请参阅[播放器事件](https://developer.vimeo.com/player/sdk/reference#about-player-events)。加载 SDK 失败时也会触发 `error`，但不会提供 Vimeo 自有 `error` 事件所声明的事件和播放器参数。

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

使用插槽来控制播放器周围的 facade。

**default**

始终可见。

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

组件等待元素触发器时显示。

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

Vimeo SDK 加载时显示。

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

替换默认的 Vimeo 缩略图。该插槽会接收解析后的 `placeholder` URL。

```vue
<template>
  <ScriptVimeoPlayer :id="331567154">
    <template #placeholder="{ placeholder }">
      <img :src="placeholder" alt="视频占位图">
    </template>
  </ScriptVimeoPlayer>
</template>
```

## [`useScriptVimeoPlayer()`{lang="ts"}](/scripts/vimeo-player){lang="ts"}

当你需要加载 Vimeo Player SDK 并以编程方式创建播放器时，请使用 [`useScriptVimeoPlayer()`{lang="ts"}](/scripts/vimeo-player){lang="ts"}。

```ts
export function useScriptVimeoPlayer<T extends VimeoPlayerApi>(_options?: VimeoPlayerInput) {}
```

有关触发器、代理和其他脚本选项，请参阅[注册脚本](/docs/guides/registry-scripts)。

::script-types
::

## 示例

```ts
export interface VimeoPlayerApi {
  Vimeo: {
    Player: ScriptVimeoPlayer
  }
}
```

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

function play() {
  player?.play()
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
