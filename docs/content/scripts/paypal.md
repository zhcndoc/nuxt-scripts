---
title: PayPal
description: 从 Vue 组件加载 PayPal JavaScript SDK v6，并创建结账或消息会话。
links:
  - label: useScriptPayPal
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/paypal.ts
    size: xs
  - label: "<ScriptPayPalButtons>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptPayPalButtons.vue
    size: xs
  - label: "<ScriptPayPalMessages>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptPayPalMessages.vue
    size: xs
---

[PayPal](https://www.paypal.com) 提供在线结账和支付 API。

Nuxt Scripts 通过以下方式集成 [PayPal JavaScript SDK v6](https://developer.paypal.com/sdk/js/reference)：

- `useScriptPayPal` 可组合函数，用于加载 `https://www.paypal.com/web-sdk/v6/core`。
- `ScriptPayPalButtons` 组件，用于初始化 PayPal SDK v6 实例，并通过作用域插槽暴露该实例。
- `ScriptPayPalMessages` 组件，用于创建 `paypal-messages` 会话。

::script-stats  
::  

::script-docs
::

::callout{type="warning"}
`ScriptPayPalButtons` 和 `ScriptPayPalMessages` 上的 `trigger` 属性目前不会影响 SDK 加载或门面状态。需要延迟 SDK 本身的加载时，请配置 `scripts.registry.paypal.trigger`。
::

除非显式设置 `sandbox`，否则该可组合函数在开发环境中使用 PayPal 的沙盒端点，在生产环境中使用线上端点。

## 类型

安装 `@paypal/paypal-js` 以获得完整的 TypeScript 支持。

```bash
pnpm add -D @paypal/paypal-js
```

v6 类型定义可从 `@paypal/paypal-js/sdk-v6` 获取。

### 示例

::code-group

:pay-pal-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
import type { Components, SdkInstance } from '@paypal/paypal-js/sdk-v6'

const clientId = 'YOUR_CLIENT_ID'

function onSdkReady(instance: SdkInstance<Components[]>) {
  console.log('PayPal SDK v6 已准备好', instance)
}

async function startPayment(instance?: SdkInstance<Components[]>) {
  if (!instance)
    return

  const eligibility = await instance.findEligibleMethods()
  if (eligibility.isEligible('paypal')) {
    const session = instance.createPayPalOneTimePaymentSession({
      onApprove: async (data) => {
        await fetch(`/api/capture-order/${data.orderId}`, { method: 'POST' })
      },
      onError: (error) => {
        console.error('付款错误:', error)
      },
    })
    await session.start(
      { presentationMode: 'auto' },
      fetch('/api/create-order', { method: 'POST' }).then(r => r.json()),
    )
  }
}
</script>

<template>
  <div>
    <ScriptPayPalButtons
      :client-id="clientId"
      :components="['paypal-payments']"
      page-type="checkout"
      @ready="onSdkReady"
    >
      <template #default="{ sdkInstance }">
        <button @click="startPayment(sdkInstance)">
          使用 PayPal 付款
        </button>
      </template>
    </ScriptPayPalButtons>
  </div>
</template>
```

::

#### 使用环境变量

使用环境变量配置客户端 ID：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      paypal: { trigger: 'onNuxtReady' },
    }
  },
})
```

该模块会自动注册 PayPal 的公共 `clientId` 运行时配置字段，因此你可以省略手动配置的 `runtimeConfig` 块。

```text [.env]
NUXT_PUBLIC_SCRIPTS_PAYPAL_CLIENT_ID=<YOUR_CLIENT_ID>
```

门面组件目前会将其 `client-id` prop 默认设置为字面值 `test`。该 prop 会覆盖运行时配置，因此请将解析后的 ID 显式传递给组件：

```vue
<script setup lang="ts">
const config = useRuntimeConfig()
const clientId = config.public.scripts.paypal.clientId
</script>

<template>
  <ScriptPayPalButtons :client-id="clientId" />
</template>
```

如果不传递该 prop，生产构建将使用 PayPal 的正式端点和 `test` 客户端 ID。

### 组合式函数

```ts
export function useScriptPayPal<T extends PayPalApi>(_options?: PayPalInput) {}
```

有关触发器、代理和其他脚本选项，请参阅[注册表脚本](/docs/guides/registry-scripts)。

::script-types
::
