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

[Crisp](https://crisp.chat/) 是一个客户消息平台，让你通过聊天、邮件等方式与客户沟通。

Nuxt Scripts 提供了一个组合式函数 [useScriptCrisp](#usescriptcrisp) 和一个无头外观组件 [ScriptCrisp](#scriptcrisp) 来与 Crisp 交互。

## ScriptCrisp

`ScriptCrisp` 组件是一个无头外观组件，封装了 [useScriptCrisp](#usescriptcrisp) 组合式函数，提供了一种简单且性能优化的方式，在你的 Nuxt 应用中加载 Crisp。

它通过利用[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)实现性能优化，只在特定元素事件发生时加载 Crisp。

默认情况下，它会在 `click` DOM 事件时加载。

### 演示

::code-group

:crisp-demo{label="输出"}

```vue [Input]
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
    <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="点击加载" description="点击右侧按钮将加载 crisp 脚本" />
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
  position: relative; /* 改为 fixed */
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
  background-image: url(data:image/svg+xml;base64,PHN2ZyBoZWlnaHQ9IjMwIiB3aWR0aD0iMzUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiPjxkZWZzPjxmaWx0ZXIgaWQ9ImEiIGhlaWdodD0iMTM4LjclIiB3aWR0aD0iMTMxLjQiIHg9Ii0xNS43JSIgeT0iLTE1LjElIj48ZmVNb3JwaG9sb2d5IGluPSJTb3VyY2VBbHBoYSIgb3BlcmF0b3I9ImRpbGF0ZSIgcmFkaXVzPSIxIiByZXN1bHQ9InNoYWRvd1NwcmVhZE91dGVyMSIvPjxmZU9mZnNldCBkeT0iMSIgaW49InNoYWRvd1NwcmVhZE91dGVyMSIgcmVzdWx0PSJzaGFkb3dPZmZzZXRPdXRlcjEiLz48ZmVHYXVzc2lhbkJsdXIgaW49InNoYWRvd09mZnNldE91dGVyMSIgcmVzdWx0PSJzaGFkb3dCbHVyT3V0ZXIxIiBzdGREZXZpYXRpb249IjEiLz48ZmVDb21wb3NpdGUgaW49InNoYWRvd0JsdXJPdXRlcjEiIGluMj0iU291cmNlQWxwaGEiIG9wZXJhdG9yPSJvdXQiIHJlc3VsdD0ic2hhZG93Qmx1ck91dGVyMSIvPjxmZUNvbG9yTWF0cml4IGluPSJzaGFkb3dCbHVyT3V0ZXIxIiB2YWx1ZXM9IjAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwIDAgMCAwLjA3IDAiLz48L2ZpbHRlcj48cGF0aCBpZD0iYiIgZD0iTTE0LjIzIDIwLjQ2bC05LjY1IDEuMUwzIDUuMTIgMzAuMDcgMmwxLjU4IDE2LjQ2LTkuMzcgMS4wNy0zLjUgNS43Mi00LjU1LTQuOHoiLz48L2RlZnM+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48dXNlIGZpbGw9IiMwMDAiIGZpbHRlcj0idXJsKCNhKSIgeGxpbms6aHJlZj0iI2IiLz48dXNlIGZpbGw9IiNmZmYiIHN0cm9rZT0iI2ZmZiIgc3Ryb2tlLXdpZHRoPSIyIiB4bGluazpocmVmPSIjYiIvPjwvZz48L3N2Zz4=)!important
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

完整的属性、事件和插槽信息，请参见[外观组件 API](/docs/guides/facade-components#facade-components-api)。

### Props

- `trigger`：加载 crisp 的触发事件。默认是 `click`。详见[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。
- `id`：Crisp ID。
- `runtimeConfig`：额外配置选项。用于配置语言环境。等同于 CRISP_RUNTIME_CONFIG。
- `tokenId`：关联会话，相当于使用 CRISP_TOKEN_ID 变量。等同于 CRISP_TOKEN_ID。
- `cookieDomain`：限制 crisp cookie 设置的域。等同于 CRISP_COOKIE_DOMAIN。
- `cookieExpiry`：cookie 过期时间（秒）。等同于 CRISP_COOKIE_EXPIRATION。

完整细节请参见[配置架构](#config-schema)。

#### 使用环境变量配置

如果你希望通过环境变量配置 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      crisp: true,
    }
  },
  // 需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        crisp: {
          id: '', // 对应 NUXT_PUBLIC_SCRIPTS_CRISP_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_CRISP_ID=<YOUR_ID>
```

### 事件

`ScriptCrisp` 组件在 crisp 加载完成时触发一个 `ready` 事件。

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

该插槽用于在 crisp 加载时显示内容。

```vue
<template>
  <ScriptCrisp>
    <template #awaitingLoad>
    <div style="width: 54px; height: 54px; border-radius: 54px; cursor: pointer; background-color: #1972F5;">
      聊天!
    </div>
    </template>
  </ScriptCrisp>
</template>
```

**loading**

该插槽用于在 crisp 正在加载时显示内容。

提示：出于无障碍和用户体验考虑，通常建议使用 `ScriptLoadingIndicator`。

```vue
<template>
  <ScriptCrisp>
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        正在加载...
      </div>
    </template>
  </ScriptCrisp>
</template>
```

## useScriptCrisp

`useScriptCrisp` 组合式函数让你可以细粒度控制 Crisp SDK。它提供了程序加载 Crisp SDK 并进行交互的方式。

```ts
export function useScriptCrisp<T extends CrispApi>(_options?: CrispInput) {}
```

请参阅[注册脚本](/docs/guides/registry-scripts)指南了解更多高级用法。

### 配置架构

```ts
export const CrispOptions = object({
  /**
   * Crisp ID。
   */
  id: string(),
  /**
   * 额外配置选项。用于配置语言环境。
   * 同 CRISP_RUNTIME_CONFIG。
   * @see https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/language-customization/
   */
  runtimeConfig: optional(object({
    locale: optional(string()),
  })),
  /**
   * 关联会话，相当于使用 CRISP_TOKEN_ID 变量。
   * 同 CRISP_TOKEN_ID。
   * @see https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/session-continuity/
   */
  tokenId: optional(string()),
  /**
   * 限制 crisp cookie 设置的域。
   * 同 CRISP_COOKIE_DOMAIN。
   * @see https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/cookie-policies/
   */
  cookieDomain: optional(string()),
  /**
   * cookie 过期时间（秒）。
   * 同 CRISP_COOKIE_EXPIRATION。
   * @see https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/cookie-policies/#change-cookie-expiration-date
   */
  cookieExpiry: optional(number()),
})
```

### CrispApi

```ts
export interface CrispApi {
  push: (...args: any[]) => void
  is: (name: 'chat:opened' | 'chat:closed' | 'chat:visible' | 'chat:hidden' | 'chat:small' | 'chat:large' | 'session:ongoing' | 'website:available' | 'overlay:opened' | 'overlay:closed' | string) => boolean
  set: (name: 'message:text' | 'session:data' | 'session:segments' | 'session:event' | 'user:email' | 'user:phone' | 'user:nickname' | 'user:avatar' | 'user:company' | string, value: any) => void
  get: (name: 'chat:unread:count' | 'message:text' | 'session:identifier' | 'session:data' | 'user:email' | 'user:phone' | 'user:nickname' | 'user:avatar' | 'user:company' | string) => any
  do: (name: 'chat:open' | 'chat:close' | 'chat:toggle' | 'chat:show' | 'chat:hide' | 'helpdesk:search' | 'helpdesk:article:open' | 'helpdesk:query' | 'overlay:open' | 'overlay:close' | 'message:send' | 'message:show' | 'message:read' | 'message:thread:start' | 'message:thread:end' | 'session:reset' | 'trigger:run' | string, arg2?: any) => any
  on: (name: 'session:loaded' | 'chat:initiated' | 'chat:opened' | 'chat:closed' | 'message:sent' | 'message:received' | 'message:compose:sent' | 'message:compose:received' | 'user:email:changed' | 'user:phone:changed' | 'user:nickname:changed' | 'user:avatar:changed' | 'website:availability:changed' | 'helpdesk:queried' | string, callback: (...args: any[]) => any) => void
  off: (name: 'session:loaded' | 'chat:initiated' | 'chat:opened' | 'chat:closed' | 'message:sent' | 'message:received' | 'message:compose:sent' | 'message:compose:received' | 'user:email:changed' | 'user:phone:changed' | 'user:nickname:changed' | 'user:avatar:changed' | 'website:availability:changed' | 'helpdesk:queried' | string, callback: (...args: any[]) => any) => void
  config: (options: any) => void
  help: () => void
  [key: string]: any
}
```

更多信息，请参阅 [Crisp API 文档](https://docs.crisp.chat/guides/chatbox-sdks/web-sdk/dollar-crisp/)。

## 示例

加载 Crisp SDK 并通过编程方式进行交互。

```vue
<script setup lang="ts">
const crisp = useScriptCrisp({
  id: 'YOUR_ID'
})
crisp.set('user:nickname', 'Harlan')
crisp.do('chat:open')
</script>
```