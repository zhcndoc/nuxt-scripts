---

title: Crisp
description: 在你的 Nuxt 应用中展示性能优化的 Crisp。
links:
  - label: useScriptCrisp
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/crisp.ts
    size: xs
  - label: "<ScriptCrisp>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptCrisp.vue
    size: xs

---

[Crisp](https://crisp.chat/) 是一个客户消息平台，让你可以通过聊天、电子邮件等方式与客户沟通。

Nuxt Scripts 提供了一个 [`useScriptCrisp()`{lang="ts"}](#usescriptcrisp){lang="ts"} 组合函数和一个无头的外观组件 [`<ScriptCrisp>`{lang="html"}](#scriptcrisp){lang="html"} 用于与 Crisp 交互。

::script-stats
::

::script-types
::

## [`<ScriptCrisp>`{lang="html"}](/scripts/crisp){lang="html"}

[`<ScriptCrisp>`{lang="html"}](/scripts/crisp){lang="html"} 组件是一个无头的外观组件，封装了 [`useScriptCrisp()`{lang="ts"}](#usescriptcrisp){lang="ts"} 组合函数，提供了一种简单且性能优化的方式，在你的 Nuxt 应用中加载 Crisp。

它通过使用[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)进行性能优化，仅在特定元素事件发生时加载 Crisp。

默认情况下，它将在 `click` DOM 事件时加载。

### 示例

::code-group

:crisp-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
const isLoaded = ref(false)
</script>

<template>
  <div class="not-prose">
    <div class="flex items-center justify-center p-5">
      <ScriptCrisp id="b1021910-7ace-425a-9ef5-07f49e5ce417" class="crisp">
        <template #awaitingLoad>
          <div class="crisp-icon" />
        </template>
        <template #loading>
          <ScriptLoadingIndicator color="black" />
        </template>
      </ScriptCrisp>
    </div>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击加载" description="点击右侧按钮将加载 Crisp 脚本" />
      <UAlert v-else color="green" variant="soft" title="Crisp 已加载" description="Crisp 外观组件不再显示。" />
    </div>
  </div>
</template>

<style>
.crisp {
  width: 54px;
  height: 54px;
  border-radius: 54px;
  cursor: pointer;
  background-color: #1972F5;
  position: relative; /* 可以改为 fixed */
  bottom: 20px;
  right: 24px;
  z-index: 100000;
  box-shadow: 0 4px 10px 0 rgba(0,0,0!important,.05) !important;
}
.crisp-icon {
  position: absolute;
  top: 16px;
  left: 11px;
  width: 32px;
  height: 26px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  background-image: url(data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjMwIiB3aWR0aD0iMzUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPjxkZWZzPjxmaWx0ZXIgaWQ9ImEiIGhlaWdodD0iMTM4LjclIiB3aWR0aD0iMTMxLjQiIHg9Ii0xNS43JSIgeT0iLTE1LjElIj48ZmVNb3JwaG9sb2d5IGluPSJTb3VyY2VBbHBoYSIgb3BlcmF0b3I9ImRpbGF0ZSIgcmFkaXVzPSIxIiByZXN1bHQ9InNoYWRvd1NwcmVhZE91dGVyMSIvPjxmZU9mZnNldCBkeT0iMSIgaW49InNoYWRvd1NwcmVhZE91dGVyMSIgcmVzdWx0PSJzaGFkb3dPZmZzZXRPdXRlcjEiLz48ZmVHb3Vzc2lhbkJsdXIgaW49InNoYWRvd09mZnNldE91dGVyMSIgcmVzdWx0PSJzaGFkb3dCbHVyT3V0ZXIxIiBzdGREZXY9IjEiLz48ZmVDb21wb3NpdGUgaW49InNoYWRvd0JsdXJPdXRlcjEiIGluMj0iU291cmNlQWxwaGEiIG9wZXJhdG9yPSJvdXQiIHJlc3VsdD0ic2hhZG93Qmx1ck91dGVyMSIvPjxmaUNvbG9yTWF0cml4IGluPSJzaGFkb3dCbHVyT3V0ZXIxIiB2YWx1ZXM9IjAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwLjA3IDAiLz48L2ZpbHRlcj48cGF0aCBpZD0iYiIgZD0iTTE0LjIzIDIwLjQ2bC05LjY1IDEuMUwzIDUuMTIgMzAuMDcgMmwxLjU4IDE2LjQ2LTkuMzcgMS4wNy0zLjUgNS43Mi00LjU1LTQuOHoiLz48L2RlZnM+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48dXNlIGZpbGw9IiMwMDAiIGZpbHRlcj0idXJsKCNhKSIgeGxpbms6aHJlZj0iI2IiLz48dXNlIGZpbGw9IiNmZmYiIHN0cm9rZT0iI2ZmZiIgc3Ryb2tlLXdpZHRoPSIyIiB4bGluazpocmVmPSIjYiIvPjwvZz48L3N2Zz4=)!important
}
@media (max-height: 600px) {
  .crisp {
    bottom: 14px;
    right: 14px;
  }
}
</style>
```

::

### 组件 API

有关完整的属性、事件和插槽，请参阅[外观组件 API](/docs/guides/facade-components#facade-components-api)。

#### 使用环境变量

如果你更喜欢使用环境变量配置你的 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      crisp: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        crisp: {
          id: '', // NUXT_PUBLIC_SCRIPTS_CRISP_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_CRISP_ID=<你的_ID>
```

### 事件

[`<ScriptCrisp>`{lang="html"}](/scripts/crisp){lang="html"} 组件在 Crisp 加载后会触发一个 `ready` 事件。

```ts
const emits = defineEmits<{
  ready: [crisp: Crisp]
}>()
```

```vue
<script setup lang="ts">
function onReady(crisp) {
  console.log('Crisp 已准备好', crisp)
}
</script>

<template>
  <ScriptCrisp @ready="onReady" />
</template>
```

### 插槽

**awaitingLoad**

该插槽在 Crisp 加载时显示内容。

```vue
<template>
  <ScriptCrisp>
    <template #awaitingLoad>
      <div style="width: 54px; height: 54px; border-radius: 54px; cursor: pointer; background-color: #1972F5;">
        聊天！
      </div>
    </template>
  </ScriptCrisp>
</template>
```

**loading**

该插槽在 Crisp 加载时显示内容。

提示：为了无障碍和用户体验，默认情况下应该使用 `ScriptLoadingIndicator`。

```vue
<template>
  <ScriptCrisp>
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptCrisp>
</template>
```

## [`useScriptCrisp()`{lang="ts"}](/scripts/crisp){lang="ts"}

[`useScriptCrisp()`{lang="ts"}](/scripts/crisp){lang="ts"} 组合函数让你可以细粒度控制 Crisp SDK。它提供了一种方式来加载 Crisp SDK 并以编程方式与之交互。

```ts
export function useScriptCrisp<T extends CrispApi>(_options?: CrispInput) {}
```

请参阅[脚本注册指南](/docs/guides/registry-scripts)以了解更多高级用法。

更多信息请参考 [Crisp API 文档](https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/dollar-crisp/)。

## 示例

加载 Crisp SDK 并以编程方式与其交互。

```vue
<script setup lang="ts">
const crisp = useScriptCrisp({
  id: 'YOUR_ID'
})
crisp.set('user:nickname', 'Harlan')
crisp.do('chat:open')
</script>
```
