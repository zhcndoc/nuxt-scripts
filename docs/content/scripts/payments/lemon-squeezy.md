---
title: Lemon Squeezy
description: 在你的 Nuxt 应用中使用 Lemon Squeezy。
links:
  - label: useScriptLemonSqueezy
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/lemon-squeezy.ts
    size: xs
  - label: "<ScriptLemonSqueezy>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptLemonSqueezy.vue
    size: xs
---

[Lemon Squeezy](https://www.lemonsqueezy.com/) 是一个流行的支付网关，允许你在线接受付款。

Nuxt Scripts 提供了一个 [useScriptLemonSqueezy](#usescriptlemonsqueezy) 组合式函数和一个无头门面组件 [ScriptLemonSqueezy](#scriptlemonsqueezy) 组件，以便与 Lemon Squeezy 交互。

## ScriptLemonSqueezy

`ScriptLemonSqueezy` 组件是一个无头的[门面组件](/docs/guides/facade-components)，封装了 [useScriptLemonSqueezy](#useScriptLemonSqueezy) 组合式函数，提供了一种简单且性能优化的方式在你的 Nuxt 应用中加载 Lemon Squeezy。

```vue
<template>
<ScriptLemonSqueezy>
  <NuxtLink href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1">
    买我 - $9.99
  </NuxtLink>
</ScriptLemonSqueezy>
</template>
```

它通过向组件内的所有 `a` 标签注入 `.lemonsqueezy-button` 类名，然后使用 `visibility` [元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 来加载 Lemon Squeezy 脚本进行工作。

### 演示

::code-group

:lemon-squeezy-demo{label="输出"}

```vue [输入]
<script lang="ts" setup>
const ready = ref(false)
const events = ref([])
</script>

<template>
<div class="not-prose w-full">
  <div class="flex items-center justify-center p-5">
    <ScriptLemonSqueezy @lemon-squeezy-event="e => events.push(e)" @ready="ready = true">
      <UButton to="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1" class="block mb-3">
        买我 - $9.99
      </UButton>
      <UButton to="https://harlantest.lemonsqueezy.com/buy/76bbfa74-a81a-4111-8449-4f5ad564ed76?embed=1" class="block">
        买我 - 想付多少就付多少
      </UButton>
    </ScriptLemonSqueezy>
  </div>
  <div>
    <UAlert v-if="!ready" class="mb-5" size="sm" color="blue" variant="soft" title="Lemon Squeezy 尚未加载" description="它会在 DOM 元素进入视口时加载。" />
    <UAlert v-else color="green" variant="soft" title="Lemon Squeezy 已加载">
      <template #description>
      <div class="mb-2">
        按钮已激活，将打开模态窗口，事件跟踪：
      </div>
      <div v-for="event in events" class="text-xs">
        {{ event.event }}
      </div>
      </template>
    </UAlert>
  </div>
</div>
</template>
```

::

### 组件 API

完整的 props、事件和插槽请参阅 [门面组件 API](/docs/guides/facade-components#facade-components-api)。

### 事件

***`lemon-squeezy-event`***

由 Lemon.js 脚本发出的事件会通过此事件转发。负载是一个带有 `event` 和 `data` 键的对象。

```ts
export type LemonSqueezyEventPayload = { event: 'Checkout.Success', data: Record<string, any> }
  & { event: 'Checkout.ViewCart', data: Record<string, any> }
  & { event: 'GA.ViewCart', data: Record<string, any> }
  & { event: 'PaymentMethodUpdate.Mounted' }
  & { event: 'PaymentMethodUpdate.Closed' }
  & { event: 'PaymentMethodUpdate.Updated' }
  & { event: string }
  ```
  
## useScriptLemonSqueezy

`useScriptLemonSqueezy` 组合式函数让你可以更细粒度地控制 Lemon Squeezy SDK。它提供了一种加载 Lemon Squeezy SDK 并以编程方式与之交互的方式。

```ts
export function useScriptLemonSqueezy<T extends LemonSqueezyApi>(_options?: LemonSqueezyInput) {}
```

请参阅 [Registry Scripts](/docs/guides/registry-scripts) 指南以了解更多高级用法。

### LemonSqueezyApi

```ts
export interface LemonSqueezyApi {
  /**
   * 在页面上初始化 Lemon.js。
   * @param options - 一个带有单个属性 eventHandler 的对象，当 Lemon.js 发出事件时会调用该函数。
   */
  Setup: (options: {
    eventHandler: (event: { event: 'Checkout.Success', data: Record<string, any> }
      & { event: 'Checkout.ViewCart', data: Record<string, any> }
      & { event: 'GA.ViewCart', data: Record<string, any> }
      & { event: 'PaymentMethodUpdate.Mounted' }
      & { event: 'PaymentMethodUpdate.Closed' }
      & { event: 'PaymentMethodUpdate.Updated' }
      & { event: string }
    ) => void
  }) => void
  /**
   * 刷新页面上所有 `lemonsqueezy-button` 监听器。
   */
  Refresh: () => void

  Url: {
    /**
     * 打开指定的 Lemon Squeezy URL，通常是结账或支付详情更新的覆盖层。
     * @param url - 要打开的 URL。
     */
    Open: (url: string) => void

    /**
     * 关闭当前打开的 Lemon Squeezy 结账覆盖窗口。
     */
    Close: () => void
  }
  Affiliate: {
    /**
     * 获取联盟追踪 ID
     */
    GetID: () => string

    /**
     * 向指定 URL 添加联盟追踪参数
     * @param url - 要添加联盟追踪参数的 URL。
     */
    Build: (url: string) => string
  }
  Loader: {
    /**
     * 显示 Lemon.js 加载器。
     */
    Show: () => void

    /**
     * 隐藏 Lemon.js 加载器。
     */
    Hide: () => void
  }
}
```

## 示例

使用 Lemon Squeezy SDK 和支付链接。

```vue
<script setup lang="ts">
const { proxy } = useScriptLemonSqueezy()
onMounted(() => {
  proxy.Setup()
})
</script>

<template>
  <a href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1" class="lemonsqueezy-button" />
</template>
```