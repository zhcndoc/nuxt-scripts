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

Nuxt Scripts 提供了一个 [`useScriptLemonSqueezy()`{lang="ts"}](#usescriptlemonsqueezy){lang="ts"} 组合函数和一个无头的外观组件 [`<ScriptLemonSqueezy>`{lang="html"}](#scriptlemonsqueezy){lang="html"} 组件，用于与 Lemon Squeezy 交互。


::script-stats
::

::script-types
::

## [`<ScriptLemonSqueezy>`{lang="html"}](/scripts/lemon-squeezy){lang="html"}

[`<ScriptLemonSqueezy>`{lang="html"}](/scripts/lemon-squeezy){lang="html"} 组件是一个无头的 [外观组件](/docs/guides/facade-components)，封装了 [`useScriptLemonSqueezy()`{lang="ts"}](/scripts/lemon-squeezy){lang="ts"} 组合函数，提供了一种简单且性能优化的方式在你的 Nuxt 应用中加载 Lemon Squeezy。

```vue
<template>
  <ScriptLemonSqueezy>
    <NuxtLink href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1">
      购买我 - $9.99
    </NuxtLink>
  </ScriptLemonSqueezy>
</template>
```

它通过向组件内的任何 `a` 标签注入 `.lemonsqueezy-button` 类，然后用 `visibility` [元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 加载 Lemon Squeezy 脚本来工作。

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
          购买我 - $9.99
        </UButton>
        <UButton to="https://harlantest.lemonsqueezy.com/buy/76bbfa74-a81a-4111-8449-4f5ad564ed76?embed=1" class="block">
          购买我 - 想付多少就付多少
        </UButton>
      </ScriptLemonSqueezy>
    </div>
    <div>
      <UAlert v-if="!ready" class="mb-5" size="sm" color="blue" variant="soft" title="Lemon Squeezy 未加载" description="当 DOM 元素在视口内时加载。" />
      <UAlert v-else color="green" variant="soft" title="Lemon Squeezy 已加载">
        <template #description>
          <div class="mb-2">
            按钮已激活并会打开模态框，跟踪事件：
          </div>
          <div v-for="(event, index) in events" :key="index" class="text-xs">
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

查看 [外观组件 API](/docs/guides/facade-components#facade-components-api) 了解完整的 props、事件和插槽。

### 事件

***`lemon-squeezy-event`***

Lemon.js 脚本发出的事件会通过此事件转发。载荷是一个包含 `event` 和 `data` 键的对象。

```ts
export type LemonSqueezyEventPayload = { event: 'Checkout.Success', data: Record<string, any> }
  & { event: 'Checkout.ViewCart', data: Record<string, any> }
  & { event: 'GA.ViewCart', data: Record<string, any> }
  & { event: 'PaymentMethodUpdate.Mounted' }
  & { event: 'PaymentMethodUpdate.Closed' }
  & { event: 'PaymentMethodUpdate.Updated' }
  & { event: string }
  ```

## [`useScriptLemonSqueezy()`{lang="ts"}](/scripts/lemon-squeezy){lang="ts"}

[`useScriptLemonSqueezy()`{lang="ts"}](/scripts/lemon-squeezy){lang="ts"} 组合函数让你可以细粒度地控制 Lemon Squeezy SDK。它提供了一种方式来加载 Lemon Squeezy SDK 并通过编程与之交互。

```ts
export function useScriptLemonSqueezy<T extends LemonSqueezyApi>(_options?: LemonSqueezyInput) {}
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南，了解更多关于高级用法的信息。

## 示例

使用 Lemon Squeezy SDK 结合支付链接。

```vue
<script setup lang="ts">
const { proxy } = useScriptLemonSqueezy()
onMounted(() => {
  proxy.Setup()
})
</script>

<template>
  <a href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1" class="lemonsqueezy-button">立即购买</a>
</template>
```
