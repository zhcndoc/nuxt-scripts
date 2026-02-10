---
title: Intercom
description: 在你的 Nuxt 应用中使用 Intercom。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/intercom.ts
    size: xs
  - label: "<ScriptIntercom>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptIntercom.vue
    size: xs
---

[Intercom](https://www.intercom.com/) 是一款客户消息平台，帮助你建立更好的客户关系。

Nuxt Scripts 提供了一个 [useScriptIntercom](#usescriptintercom) 组合式函数和一个无头门面组件 [ScriptIntercom](#scriptintercom) 用于与 Intercom 交互。

## ScriptIntercom

`ScriptIntercom` 组件是一个无头门面组件，封装了 [useScriptIntercom](#usescriptintercom) 组合式函数，提供了一种简单、高性能的方式在你的 Nuxt 应用中加载 Intercom。

它通过利用 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 实现性能优化，仅在特定元素事件发生时才加载 Intercom。

默认情况下，它会在 `click` DOM 事件时加载。

### 演示

::code-group

:intercom-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
const isLoaded = ref(false)
</script>

<template>
<div>
  <ScriptIntercom @ready="isLoaded = true" app-id="akg5rmxb" api-base="https://api-iam.intercom.io" alignment="left" :horizontal-padding="50" class="intercom">
    <div style="display: flex; align-items: center; justify-content: center; width: 48px; height: 48px;">
      <svg width="24" height="24" xmlns="http://www.w3.org/2000/svg" fill="white" viewBox="0 0 28 32"><path d="M28 32s-4.714-1.855-8.527-3.34H3.437C1.54 28.66 0 27.026 0 25.013V3.644C0 1.633 1.54 0 3.437 0h21.125c1.898 0 3.437 1.632 3.437 3.645v18.404H28V32zm-4.139-11.982a.88.88 0 00-1.292-.105c-.03.026-3.015 2.681-8.57 2.681-5.486 0-8.517-2.636-8.571-2.684a.88.88 0 00-1.29.107 1.01 1.01 0 00-.219.708.992.992 0 00.318.664c.142.128 3.537 3.15 9.762 3.15 6.226 0 9.621-3.022 9.763-3.15a.992.992 0 00.317-.664 1.01 1.01 0 00-.218-.707z" /></svg>
    </div>
  </ScriptIntercom>
  <div class="text-center">
    <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击加载" description="点击右侧按钮将加载 Intercom 脚本" />
    <UAlert v-else color="green" variant="soft" title="Intercom 已加载" description="Intercom 门面组件已不再显示。" />
  </div>
</div>
</template>

<style>
.intercom {
  display: block;
  position: relative; /* 可更改为 fixed */
  z-index: 100000;
  width: 48px;
  align-items: center;
  justify-content: center;
  height: 48px;
  border-radius: 50%;
  cursor: pointer;
  background-color: #0071b2;
  filter: drop-shadow(rgba(0, 0, 0, 0.06) 0px 1px 6px) drop-shadow(rgba(0, 0, 0, 0.16) 0px 2px 32px);
}
</style>
```

::

### 组件 API

完整的属性、事件和插槽请参见 [门面组件 API](/docs/guides/facade-components#facade-components-api)。

### Props

- `trigger`：触发事件以加载 Intercom。默认是 `click`。详情参见 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。
- `app-id`：Intercom 应用 ID。
- `api-base`：Intercom API 基础 URL。
- `name`：用户姓名。
- `email`：用户邮箱。
- `user-id`：用户 ID。
- `alignment`：消息组件对齐位置，可选 `left` 或 `right`，默认 `right`。
- `horizontal-padding`：消息组件的水平内边距，默认 `20`。
- `vertical-padding`：消息组件的垂直内边距，默认 `20`。

完整细节请参见 [配置 Schema](#config-schema)。

#### 使用环境变量配置

如果你更倾向用环境变量配置 app ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      intercom: true,
    }
  },
  // 你需要提供一个运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        intercom: {
          app_id: '', // NUXT_PUBLIC_SCRIPTS_INTERCOM_APP_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_INTERCOM_APP_ID=<YOUR_APP_ID>
```

### 事件

`ScriptIntercom` 组件在 Intercom 加载完成时会触发一个 `ready` 事件。

```ts
const emits = defineEmits<{
  ready: [intercom: Intercom]
}>()
```

```vue
<script setup lang="ts">
function onReady(intercom) {
  console.log('Intercom 已准备好', intercom)
}
</script>

<template>
  <ScriptIntercom @ready="onReady" />
</template>
```

### Intercom API

该组件暴露了 `intercom` 实例，可用来访问底层 Intercom API。

```vue
<script setup lang="ts">
const intercomEl = ref()
onMounted(() => {
  intercomEl.value.intercom.do('chat:open')
})
</script>

<template>
  <ScriptIntercom ref="intercomEl" />
</template>
```

### 插槽

组件默认提供了最小化的 UI，仅保证功能性和可访问性。你可以通过多个插槽自由定制界面。

**default**

默认插槽，用于显示始终可见的内容。

**awaitingLoad**

当 Intercom 未加载时显示的内容。

```vue
<template>
  <ScriptIntercom>
    <template #awaitingLoad>
    <div style="width: 54px; height: 54px; border-radius: 54px; cursor: pointer; background-color: #1972F5;">
      聊天！
    </div>
    </template>
  </ScriptIntercom>
</template>
```

**loading**

Intercom 加载过程中显示的内容。

提示：一般建议默认用 `ScriptLoadingIndicator` 以提升可访问性和用户体验。

```vue
<template>
  <ScriptIntercom>
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptIntercom>
</template>
```

## useScriptIntercom

`useScriptIntercom` 组合式函数允许你细粒度控制 Intercom 在你网站上的加载时机和方式。

```ts
const { proxy } = useScriptIntercom({
  app_id: 'YOUR_APP_ID'
})

// 示例
proxy.Intercom('show')
proxy.Intercom('update', { name: 'John Doe' })
```

请参考 [脚本注册指南](/docs/guides/registry-scripts) 了解更多高级用法。

### IntercomApi

```ts
export interface IntercomApi {
  Intercom: ((event: 'boot', data?: Input<typeof IntercomOptions>) => void)
  & ((event: 'shutdown') => void)
  & ((event: 'update', options?: Input<typeof IntercomOptions>) => void)
  & ((event: 'hide') => void)
  & ((event: 'show') => void)
  & ((event: 'showSpace', spaceName: 'home' | 'messages' | 'help' | 'news' | 'tasks' | 'tickets' | string) => void)
  & ((event: 'showMessages') => void)
  & ((event: 'showNewMessage', content?: string) => void)
  & ((event: 'onHide', fn: () => void) => void)
  & ((event: 'onShow', fn: () => void) => void)
  & ((event: 'onUnreadCountChange', fn: () => void) => void)
  & ((event: 'trackEvent', eventName: string, metadata?: Record<string, any>) => void)
  & ((event: 'getVisitorId') => Promise<string>)
  & ((event: 'startTour', tourId: string | number) => void)
  & ((event: 'showArticle', articleId: string | number) => void)
  & ((event: 'showNews', newsItemId: string | number) => void)
  & ((event: 'startSurvey', surveyId: string | number) => void)
  & ((event: 'startChecklist', checklistId: string | number) => void)
  & ((event: 'showTicket', ticketId: string | number) => void)
  & ((event: 'showConversation', conversationId: string | number) => void)
  & ((event: 'onUserEmailSupplied', fn: () => void) => void)
  & ((event: string, ...params: any[]) => void)
}
```

### 配置 Schema

```ts
export const IntercomOptions = object({
  app_id: string(),
  api_base: optional(union([literal('https://api-iam.intercom.io'), literal('https://api-iam.eu.intercom.io'), literal('https://api-iam.au.intercom.io')])),
  name: optional(string()),
  email: optional(string()),
  user_id: optional(string()),
  // 定制消息组件
  alignment: optional(union([literal('left'), literal('right')])),
  horizontal_padding: optional(number()),
  vertical_padding: optional(number()),
})
```

## 示例

仅在生产环境中使用 Intercom。

::code-group

```vue [IntercomButton.vue]
<script setup lang="ts">
const { proxy } = useScriptIntercom()

// 开发环境和 SSR 中为 noop
// 生产环境客户端中正常工作
function showIntercom() {
  proxy.Intercom('show')
}
</script>

<template>
  <div>
    <button @click="showIntercom">
      与我们聊天
    </button>
  </div>
</template>
```

::