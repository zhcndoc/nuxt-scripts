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

[Intercom](https://www.intercom.com/) 是一个客户消息交流平台，帮助你建立更好的客户关系。

Nuxt Scripts 提供了一个 [`useScriptIntercom()`{lang="ts"}](#usescriptintercom){lang="ts"} 组合函数和一个无头的外观组件 [`<ScriptIntercom>`{lang="html"}](#scriptintercom){lang="html"} 组件用于与 Intercom 交互。

::script-stats
::

::script-types
::

## [`<ScriptIntercom>`{lang="html"}](/scripts/intercom){lang="html"}

[`<ScriptIntercom>`{lang="html"}](/scripts/intercom){lang="html"} 组件是一个无头的外观组件，封装了 [`useScriptIntercom()`{lang="ts"}](#usescriptintercom){lang="ts"} 组合函数，提供了一种简单并且性能优化的方式，将 Intercom 加载到你的 Nuxt 应用中。

它通过使用[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)进行了性能优化，仅在特定元素事件发生时才加载 Intercom。

默认情况下，它会在 `click` DOM 事件发生时加载。

### 示例演示

::code-group

:intercom-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
const isLoaded = ref(false)
</script>

<template>
  <div>
    <ScriptIntercom app-id="akg5rmxb" api-base="https://api-iam.intercom.io" alignment="left" :horizontal-padding="50" class="intercom" @ready="isLoaded = true">
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

#### 使用环境变量

如果你想通过环境变量配置你的 app ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      intercom: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
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
NUXT_PUBLIC_SCRIPTS_INTERCOM_APP_ID=<你的_APP_ID>
```

### 事件

[`<ScriptIntercom>`{lang="html"}](/scripts/intercom){lang="html"} 组件在 Intercom 加载时触发一个 `ready` 事件。

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

该组件暴露了一个 `intercom` 实例，你可以通过它访问基本的 Intercom API。

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

该组件默认提供最简洁的界面，仅保证功能性和可访问性。你可以通过多个插槽自定义界面。

**default**

默认插槽显示始终可见的内容。

**awaitingLoad**

该插槽在 Intercom 未加载时显示内容。

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

该插槽在 Intercom 加载时显示内容。

提示：建议默认使用 `ScriptLoadingIndicator`，提升无障碍和用户体验。

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

## [`useScriptIntercom()`{lang="ts"}](/scripts/intercom){lang="ts"}

[`useScriptIntercom()`{lang="ts"}](/scripts/intercom){lang="ts"} 组合函数让你对 Intercom 在网站上的加载时机和方式拥有细粒度控制。

```ts
const { proxy } = useScriptIntercom({
  app_id: '你的_APP_ID'
})

// 示例
proxy.Intercom('show')
proxy.Intercom('update', { name: '张三' })
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南了解更多高级用法。

## 示例

仅在生产环境中使用 Intercom。

::code-group

```vue [IntercomButton.vue]
<script setup lang="ts">
const { proxy } = useScriptIntercom()

// 在开发环境和 SSR 中无操作，
// 仅在生产环境客户端生效
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
