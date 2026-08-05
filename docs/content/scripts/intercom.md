---
title: Intercom
description: 通过类型化命令队列或点击触发的自定义启动器加载 Intercom。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/intercom.ts
    size: xs
  - label: "<ScriptIntercom>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptIntercom.vue
    size: xs
---

[Intercom](https://www.intercom.com/) 为客户对话提供应用内消息工具。

使用 [`useScriptIntercom()`{lang="ts"}](#usescriptintercom){lang="ts"} 直接调用 API，或使用 [`<ScriptIntercom>`{lang="html"}](#scriptintercom){lang="html"} 创建自定义消息启动器。

::script-stats
::

::script-docs
::

## [`<ScriptIntercom>`{lang="html"}](/scripts/intercom){lang="html"}

无头外观组件会等待其[元素触发器](/docs/guides/script-triggers#element-event-triggers)触发后再加载 Intercom。默认监听 `click` 事件。

### 演示

::code-group

:intercom-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
const isLoaded = ref(false)
</script>

<template>
  <div>
    <ScriptIntercom app-id="akg5rmxb" alignment="left" :horizontal-padding="50" class="intercom" @ready="isLoaded = true">
      <div style="display: flex; align-items: center; justify-content: center; width: 48px; height: 48px;">
        <svg width="24" height="24" xmlns="http://www.w3.org/2000/svg" fill="white" viewBox="0 0 28 32"><path d="M28 32s-4.714-1.855-8.527-3.34H3.437C1.54 28.66 0 27.026 0 25.013V3.644C0 1.633 1.54 0 3.437 0h21.125c1.898 0 3.437 1.632 3.437 3.645v18.404H28V32zm-4.139-11.982a.88.88 0 00-1.292-.105c-.03.026-3.015 2.681-8.57 2.681-5.486 0-8.517-2.636-8.571-2.684a.88.88 0 00-1.29.107 1.01 1.01 0 00-.219.708.992.992 0 00.318.664c.142.128 3.537 3.15 9.762 3.15 6.226 0 9.621-3.022 9.763-3.15a.992.992 0 00.317-.664 1.01 1.01 0 00-.218-.707z" /></svg>
      </div>
    </ScriptIntercom>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击以加载" description="点击右侧按钮将加载 Intercom 脚本" />
      <UAlert v-else color="green" variant="soft" title="Intercom 已加载" description="Intercom 外观组件不再显示。" />
    </div>
  </div>
</template>

<style>
.intercom {
  display: block;
  position: relative; /* 可改为 fixed */
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

完整的 props、事件及插槽，请查看 [外观组件 API](/docs/guides/facade-components#facade-components-api)。

::warning
组件的 `api-base` prop 当前会传递 `app_base`，但 Intercom 需要 [`api_base`](https://developers.intercom.com/installing-intercom/web/installation)。在该映射发生变化之前，此 prop 无法选择欧盟或澳大利亚的数据主机。需要使用区域端点时，请直接使用组合式函数：

```ts
useScriptIntercom({
  app_id: 'YOUR_APP_ID',
  api_base: 'https://api-iam.eu.intercom.io',
})
```
::

#### 使用环境变量

启用注册表条目后，模块会创建其公共运行时配置字段。无需添加单独的 `runtimeConfig` 块，即可设置应用 ID：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      intercom: { trigger: 'onNuxtReady' },
    }
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_INTERCOM_APP_ID=<你的_APP_ID>
```

### 事件

[`<ScriptIntercom>`{lang="html"}](/scripts/intercom){lang="html"} 组件在挂载消息工具后触发 `ready` 事件；如果脚本加载失败，则触发 `error` 事件。

```ts
const emits = defineEmits<{
  ready: [intercom: ReturnType<typeof useScriptIntercom>]
  error: []
}>()
```

```vue
<script setup lang="ts">
function onReady(intercom) {
  console.log('Intercom 已准备好', intercom)
}
</script>

<template>
  <ScriptIntercom app-id="YOUR_APP_ID" @ready="onReady" />
</template>
```

### Intercom API

组件会暴露其 `intercom` 实例，该实例是 `useScriptIntercom()`{lang="ts"} 的返回值。通过其代理调用 [Intercom JavaScript API](https://developers.intercom.com/installing-intercom/web/methods)：

```vue
<script setup lang="ts">
const intercomEl = ref()

function showMessenger() {
  intercomEl.value?.intercom.proxy.Intercom('show')
}
</script>

<template>
  <ScriptIntercom ref="intercomEl" app-id="YOUR_APP_ID" trigger="immediate" />
  <button @click="showMessenger">
    Open chat
  </button>
</template>
```

### 插槽

使用插槽构建启动器及其加载状态。

**default**

默认插槽会在外观组件可见时显示内容。

**awaitingLoad**

当 Intercom 等待其配置的触发器时，此插槽会显示内容。

```vue
<template>
  <ScriptIntercom app-id="YOUR_APP_ID">
    <template #awaitingLoad>
      <div style="width: 54px; height: 54px; border-radius: 54px; cursor: pointer; background-color: #1972F5;">
        聊天！
      </div>
    </template>
  </ScriptIntercom>
</template>
```

**loading**

该插槽在 Intercom 加载时显示内容。

`ScriptLoadingIndicator` 提供可见状态和状态标签：

```vue
<template>
  <ScriptIntercom app-id="YOUR_APP_ID">
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptIntercom>
</template>
```

组件声明了 `error` 插槽，但在 `isReady` 为 `false` 时，其加载分支当前始终优先，包括加载失败之后。请使用触发的 `error` 事件在组件外部渲染备用内容，直到该分支顺序得到修复。


## [`useScriptIntercom()`{lang="ts"}](/scripts/intercom){lang="ts"}

当你需要命令队列而不需要启动器组件时，请使用 [`useScriptIntercom()`{lang="ts"}](/scripts/intercom){lang="ts"}。

```ts
const { proxy } = useScriptIntercom({
  app_id: '你的_APP_ID'
})

// 示例
proxy.Intercom('show')
proxy.Intercom('update', { name: '张三' })
```

当已识别的用户退出登录后，在其他人使用同一浏览器之前调用 `shutdown`。Intercom 建议这样做，以清除之前用户的 Messenger 会话：

```ts
proxy.Intercom('shutdown')
```

有关触发器和加载选项，请参阅[注册脚本](/docs/guides/registry-scripts)。

::script-types
::

## 示例

通过按钮打开 Intercom：

::code-group

```vue [IntercomButton.vue]
<script setup lang="ts">
const { proxy } = useScriptIntercom({
  app_id: 'YOUR_APP_ID',
})

function showIntercom() {
  proxy.Intercom('show')
}
</script>

<template>
  <div>
    <button @click="showIntercom">
      联系我们聊天
    </button>
  </div>
</template>
```

::
