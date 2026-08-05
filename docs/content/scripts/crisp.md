---
title: Crisp
description: 为你的 Nuxt 应用添加一个延迟加载的 Crisp 聊天启动器。
links:
  - label: useScriptCrisp
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/crisp.ts
    size: xs
  - label: "<ScriptCrisp>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptCrisp.vue
    size: xs
---

[Crisp](https://crisp.chat/) 集成了实时聊天、电子邮件和其他客户消息渠道。

使用 [`useScriptCrisp()`{lang="ts"}](#usescriptcrisp){lang="ts"} 直接调用 SDK，或使用 [`<ScriptCrisp>`{lang="html"}](#scriptcrisp){lang="html"} 创建自定义聊天启动器。

::script-stats
::

::script-docs
::

## [`<ScriptCrisp>`{lang="html"}](/scripts/crisp){lang="html"}

无头外观组件会一直延迟加载 Crisp SDK，直到其[元素触发器](/docs/guides/script-triggers#element-event-triggers)触发。默认监听 `click` 事件。

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
      <ScriptCrisp id="b1021910-7ace-425a-9ef5-07f49e5ce417" class="crisp" @ready="isLoaded = true">
        <template #awaitingLoad>
          <div class="crisp-icon" />
        </template>
        <template #loading>
          <ScriptLoadingIndicator color="black" />
        </template>
      </ScriptCrisp>
    </div>
    <div class="text-center">
      <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击加载" description="点击右侧按钮将加载 Crisp 脚本。" />
      <UAlert v-else color="green" variant="soft" title="Crisp 已加载" description="Crisp 外观组件将不再显示。" />
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
  box-shadow: 0 4px 10px 0 rgba(0, 0, 0, 0.05);
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
  background-image: url(data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjMwIiB3aWR0aD0iMzUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPjxkZWZzPjxmaWx0ZXIgaWQ9ImEiIGhlaWdodD0iMTM4LjclIiB3aWR0aD0iMTMxLjQiIHg9Ii0xNS43JSIgeT0iLTE1LjElIj48ZmVNb3JwaG9sb2d5IGluPSJTb3VyY2VBbHBoYSIgb3BlcmF0b3I9ImRpbGF0ZSIgcmFkaXVzPSIxIiByZXN1bHQ9InNoYWRvd1NwcmVhZE91dGVyMSIvPjxmZU9mZnNldCBkeT0iMSIgaW49InNoYWRvd1NwcmVhZE91dGVyMSIgcmVzdWx0PSJzaGFkb3dPZmZzZXRPdXRlcjEiLz48ZmVHb3Vzc2lhbkJsdXIgaW49InNoYWRvd09mZnNldE91dGVyMSIgcmVzdWx0PSJzaGFkb3dCbHVyT3V0ZXIxIiBzdGREZXY9IjEiLz48ZmVDb21wb3NpdGUgaW49InNoYWRvd0JsdXJPaXRlcjEiIGluMj0iU291cmNlQWxwaGEiIG9wZXJhdG9yPSJvdXQiIHJlc3VsdD0ic2hhZG93Qmx1ck91dGVyMSIvPjxmaUNvbG9yTWF0cml4IGluPSJzaGFkb3dCbHVyT3V0ZXIxIiB2YWx1ZXM9IjAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwLjA3IDAiLz48L2ZpbHRlcj48cGF0aCBpZD0iYiIgZD0iTTE0LjIzIDIwLjQ2bC05LjY1IDEuMUwzIDUuMTIgMzAuMDcgMmwxLjU4IDE2LjQ2LTkuMzcgMS4wNy0zLjUgNS43Mi00LjU1LTQuOHoiLz48L2RlZnM+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48dXNlIGZpbGw9IiMwMDAiIGZpbHRlcj0idXJsKCNhKSIgeGxpbms6aHJlZj0iI2IiLz48dXNlIGZpbGw9IiNmZmYiIHN0cm9rZT0iI2ZmZiIgc3Ryb2tlLXdpZHRoPSIyIiB4bGluazpocmVmPSIjYiIvPjwvZz48L3N2Zz4=)!important
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

你可以使用环境变量配置 Crisp 网站 ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      crisp: { trigger: 'onNuxtReady' },
    }
  },
  // 公共运行时配置会接收环境变量。
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

[`<ScriptCrisp>`{lang="html"}](/scripts/crisp){lang="html"} 组件在挂载聊天框后触发 `ready` 事件；如果脚本加载失败，则触发 `error` 事件。

::callout{color="amber"}
当前模板会在进入 `error` 分支前检查 `status === 'loading' || !isReady`，因此命名插槽 `#error` 无法访问。在修复该顺序问题之前，请监听 `@error`，并在组件外部渲染备用 UI。
::

```ts
const emits = defineEmits<{
  ready: [crisp: ReturnType<typeof useScriptCrisp>]
  error: []
}>()
```

```vue
<script setup lang="ts">
function onReady(crisp) {
  console.log('Crisp 已准备好', crisp)
}
</script>

<template>
  <ScriptCrisp id="YOUR_ID" @ready="onReady" />
</template>
```

### 插槽

**awaitingLoad**

当 Crisp 等待其配置的触发器时，此插槽会显示内容。

```vue
<template>
  <ScriptCrisp id="YOUR_ID">
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

`ScriptLoadingIndicator` 提供该模块标准且符合无障碍要求的加载状态。

```vue
<template>
  <ScriptCrisp id="YOUR_ID">
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptCrisp>
</template>
```

## [`useScriptCrisp()`{lang="ts"}](/scripts/crisp){lang="ts"}

当你需要在不使用启动器组件的情况下调用 Crisp 时，请使用 [`useScriptCrisp()`{lang="ts"}](/scripts/crisp){lang="ts"}。

```ts
export function useScriptCrisp<T extends CrispApi>(_options?: CrispInput) {}
```

有关触发器和加载选项，请参阅[脚本注册表](/docs/guides/registry-scripts)；有关可用命令，请参阅 [Crisp API 文档](https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/dollar-crisp/)。

::script-types
::

## 示例

通过代理调用 Crisp：

```vue
<script setup lang="ts">
const { proxy } = useScriptCrisp({
  id: 'YOUR_ID'
})
proxy.set('user:nickname', 'Harlan')
proxy.do('chat:open')
</script>
```
