---
title: Lemon Squeezy
description: 加载 Lemon.js，或将结账链接转换为延迟加载的覆盖层触发器。
links:
  - label: useScriptLemonSqueezy
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/lemon-squeezy.ts
    size: xs
  - label: "<ScriptLemonSqueezy>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptLemonSqueezy.vue
    size: xs
---

[Lemon Squeezy](https://www.lemonsqueezy.com/) 是一个面向数字产品和订阅服务的名义商户平台。

使用 [`useScriptLemonSqueezy()`{lang="ts"}](#usescriptlemonsqueezy){lang="ts"} 调用 Lemon.js。 [`<ScriptLemonSqueezy>`{lang="html"}](#scriptlemonsqueezy){lang="html"} 会将结账链接转换为覆盖层触发器。

::script-stats
::

::script-docs
::

## [`<ScriptLemonSqueezy>`{lang="html"}](/scripts/lemon-squeezy){lang="html"}

这个无头[外观组件](/docs/guides/facade-components)会在挂载时扫描其中存在的链接。当组件根元素进入视口时，它会加载 [Lemon.js](https://docs.lemonsqueezy.com/guides/developer-guide/lemonjs)。

```vue
<template>
  <ScriptLemonSqueezy>
    <NuxtLink href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1">
      以 9.99 美元购买
    </NuxtLink>
  </ScriptLemonSqueezy>
</template>
```

挂载时，它会为组件内的链接添加 `.lemonsqueezy-button` 类。之后插入的链接不会被扫描。在动态添加结账链接后，请自行添加该类并调用 `Refresh()`{lang="ts"}，或者重新挂载组件。

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
          以 9.99 美元购买
        </UButton>
        <UButton to="https://harlantest.lemonsqueezy.com/buy/76bbfa74-a81a-4111-8449-4f5ad564ed76?embed=1" class="block">
          购买：自定义价格
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

组件会通过此事件转发 [Lemon.js 覆盖层事件](https://docs.lemonsqueezy.com/help/lemonjs/handling-events)。载荷包含一个 `event` 名称，以及对于 `Checkout.Success` 等事件而言的一个 `data` 对象。

::warning
组件在运行时会发出正确的对象，但当前的 `LemonSqueezyEventPayload` TypeScript 定义将互斥的事件名称进行了交叉合并，可能会将载荷缩减为 `never`。在类型发生更改之前，请在处理函数边界处接受 `unknown`，然后将其缩小到你所使用的字段。
::

## [`useScriptLemonSqueezy()`{lang="ts"}](/scripts/lemon-squeezy){lang="ts"}

当你需要 Lemon.js，但不使用结账链接组件时，请使用 [`useScriptLemonSqueezy()`{lang="ts"}](/scripts/lemon-squeezy){lang="ts"}。

```ts
export function useScriptLemonSqueezy<T extends LemonSqueezyApi>(_options?: LemonSqueezyInput) {}
```

有关触发器和加载选项，请参阅[脚本注册表](/docs/guides/registry-scripts)。

如果你不使用该组合式函数中的组件，并在 Lemon.js 加载后添加结账链接，请调用 [`Refresh()`{lang="ts"}](https://docs.lemonsqueezy.com/help/lemonjs/methods)，以便 SDK 将叠加层监听器附加到新链接：

```ts
proxy.Refresh()
```

::script-types
::

## 示例

为支付链接初始化 Lemon.js：

```vue
<script setup lang="ts">
const { proxy } = useScriptLemonSqueezy()
proxy.Setup({
  eventHandler(event) {
    console.log(event)
  },
})
</script>

<template>
  <a href="https://harlantest.lemonsqueezy.com/buy/52a40427-36d2-4450-a514-ae80d9e1a333?embed=1" class="lemonsqueezy-button">立即购买</a>
</template>
```
