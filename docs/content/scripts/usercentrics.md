---
title: Usercentrics
description: 加载 Usercentrics CMP v3，并根据 UC_UI_CMP_EVENT 事件驱动 useScript 的同意触发器。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/usercentrics.ts
    size: xs
---

[Usercentrics](https://usercentrics.com) 是一个用于记录同意选择和控制第三方服务的同意管理平台（CMP）。

[`useScriptUsercentrics()`{lang="ts"}](/scripts/usercentrics) 加载 CMP v3（“Web CMP”）脚本，为 `window.__ucCmp` API 提供类型定义，并公开触发其他注册表脚本所需的 `UC_UI_CMP_EVENT` 变更。

::script-stats
::

::script-docs
::

该组合式函数使用以下默认设置：

- **触发器：`onNuxtReady`。** 脚本在 Nuxt 水合后加载，使用模块级默认设置。
- **捆绑和代理：关闭。** CMP 直接从 Usercentrics 加载，不受同意状态限制。

需要访问已加载对象时，可以通过代理访问 `ucCmp`，或等待 `$script`。

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

`onConsentChange` 会返回一个 teardown 函数，可与 `onScopeDispose` 搭配使用。其回调接收原始的 `UC_UI_CMP_EVENT` 详情，例如 `{ type: 'ACCEPT_ALL' | 'DENY_ALL' | 'SAVE', ... }`。

## 打开同意界面

当你需要等待初始化时，请在 CMP 的 `UC_CMP_API_READY` 事件之前调用 `consent.whenReady()`{lang="ts"}。当前的辅助函数只监听下一个事件；它不会检测已经准备就绪的 CMP，因此后续调用可能会一直处于等待状态。其他 `consent` 辅助函数会在 `window.__ucCmp` 存在时调用它，否则不执行任何操作。

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

如果您的 Usercentrics 规则集使用了**自动阻止**，请将 `autoblocker: true` 设置为在加载器之前注入自动阻止模块：

```ts
useScriptUsercentrics({
  rulesetId: 'your-ruleset-id',
  autoblocker: true,
})
```

Usercentrics 要求自动阻止模块在其他服务脚本之前运行；请参阅其[直接实施指南](https://support.usercentrics.com/hc/en-us/articles/19446626144540-Direct-implementation-and-markup-guide)。由于该组合式函数在客户端运行时会添加此选项，因此请在依赖它执行同意管理之前，验证其在渲染应用中的加载顺序。

::script-types
::

## Partytown

不要在 Partytown 下运行 Usercentrics。其 CMP 会渲染 DOM 覆盖层，并且其 `__ucCmp` 方法未配置为转发到 Worker。
