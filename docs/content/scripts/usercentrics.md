---
title: Usercentrics
description: 加载 Usercentrics CMP v3，并根据 UC_UI_CMP_EVENT 事件驱动 useScript 的同意触发器。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/usercentrics.ts
    size: xs
---

[Usercentrics](https://usercentrics.com) 是一个同意管理平台（CMP），用于在 GDPR、CCPA 以及 IAB TCF v2 框架下收集、存储并传递终端用户对第三方脚本的同意。

Nuxt Scripts 提供了 [`useScriptUsercentrics()`{lang="ts"}](/scripts/usercentrics)，这样你就可以启动 CMP v3（“Web CMP”）加载器，暴露对 `window.__ucCmp` 程序化 API 的类型化访问，并将其他注册脚本的同意触发器直接连接到 Usercentrics 的 `UC_UI_CMP_EVENT` 浏览器事件。

::script-stats
::

::script-docs
::

这个组合式函数包含以下默认值：
- **触发器：客户端** 当 Nuxt 正在进行 hydration 时，脚本将加载。
- **Bundle / 代理：关闭** CMP 本身就是同意界面，因此它必须直接访问供应商源站。同时，它也不受同意门控限制。

你可以直接通过代理访问 `ucCmp` 对象，或者等待 `$script` promise。对于任何返回 void / Promise 的调用，建议使用代理。

::code-group

```ts [Proxy]
const { proxy } = useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
})
function showSettings() {
  proxy.ucCmp.showSecondLayer()
}
```

```ts [onLoaded]
const { onLoaded } = useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
})
onLoaded(({ ucCmp }) => {
  ucCmp.showFirstLayer()
})
```

::

## 从 Usercentrics 驱动同意触发器

将 `consent.onConsentChange(...)`{lang="ts"} 与 [`useScriptTriggerConsent`](/docs/api/use-script-trigger-consent) 配对使用，即可在用户通过 Usercentrics 横幅选择同意的瞬间加载任何第三方脚本。

```vue
<script setup lang="ts">
import { ref } from 'vue'

const { consent } = useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
})

const analyticsGranted = ref(false)

if (import.meta.client) {
  consent.onConsentChange(async (detail) => {
    if (detail.type === 'ACCEPT_ALL' || detail.type === 'SAVE') {
      const details = await window.__ucCmp!.getConsentDetails()
      analyticsGranted.value = !!details?.services?.['your-template-id']?.consent?.status
    }
    else if (detail.type === 'DENY_ALL') {
      analyticsGranted.value = false
    }
  })
}

useScriptGoogleAnalytics({
  id: 'G-XXXXXXX',
  scriptOptions: {
    trigger: useScriptTriggerConsent({ consent: analyticsGranted }),
  },
})
</script>

<template>
  <button @click="consent.showSecondLayer()">
    隐私设置
  </button>
</template>
```

`onConsentChange` 会返回一个清理函数，因此你可以在 `onScopeDispose` 中取消订阅。回调会接收原始的 `UC_UI_CMP_EVENT` 详情（例如 `{ type: 'ACCEPT_ALL' | 'DENY_ALL' | 'SAVE', ... }`）。

## 打开同意界面

在 CMP API 准备好之前，`__ucCmp` 的方法都不会生效。使用 `consent.whenReady()`{lang="ts"} 来等待它，或者直接调用 `consent` 上的辅助方法（在 CMP 启动期间它们不会生效）：

```vue
<script setup lang="ts">
const { consent } = useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
})

async function logConsent() {
  const cmp = await consent.whenReady()
  console.log(await cmp.getConsentDetails())
}
</script>

<template>
  <button @click="consent.showFirstLayer()">
    显示横幅
  </button>
  <button @click="consent.acceptAll()">
    接受全部
  </button>
  <button @click="consent.denyAll()">
    拒绝全部
  </button>
</template>
```

## 自动阻止

如果你的 Usercentrics ruleset 使用的是 **自动阻止**（而不是手动阻止），请设置 `autoblocker: true`，以便在加载器之前注入 autoblocker 模块：

```ts
useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
  autoblocker: true,
})
```

::script-types
::

## 示例

在 Nuxt 就绪时，通过 `app.vue` 加载 Usercentrics。

```vue [app.vue]
<script setup lang="ts">
useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```

## Partytown

不要在 Partytown 下运行 Usercentrics。`__ucCmp` API 的方法调用非常密集，且不适合跨 worker 边界转发；同时，CMP 需要主线程的 DOM 访问来渲染其 UI 覆层。
