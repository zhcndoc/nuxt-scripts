---

title: PayPal  
description: 在你的 Nuxt 应用中使用 PayPal。  
links:  
  - label: useScriptPayPal  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/paypal.ts  
    size: xs  
  - label: "<ScriptPayPalButtons>"  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptPayPalButtons.vue  
    size: xs  
  - label: "<ScriptPayPalMessages>"  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptPayPalMessages.vue  
    size: xs  

---

[PayPal](https://www.paypal.com) 是一个流行的支付网关，允许你在线接受付款。

Nuxt Scripts 提供了 PayPal SDK v6 的集成：  
- `useScriptPayPal` 组合式函数，用于加载脚本 `https://www.paypal.com/web-sdk/v6/core`。  
- `ScriptPayPalButtons` 组件，初始化 PayPal SDK v6 实例并通过作用域插槽暴露它。  
- `ScriptPayPalMessages` 组件，允许你在网站上使用 [PayPal Messages](https://developer.paypal.com/sdk/js/reference/)。  

::script-stats  
::  

::script-types  
::  

## 类型

要使用包含完整 TypeScript 支持的 PayPal，你需要安装 `@paypal/paypal-js` 依赖。

```bash
pnpm add -D @paypal/paypal-js
```

v6 类型定义可从 `@paypal/paypal-js/sdk-v6` 获取。

### 示例

::code-group

:pay-pal-demo{label="输出"}

```vue [Input]
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
        console.log('付款已批准:', data.orderId)
      },
      onError: (error) => {
        console.error('付款错误:', error)
      },
    })
    await session.start(
      { presentationMode: 'auto' },
      fetch('/api/create-order').then(r => r.json()),
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

#### 使用环境变量配置

如果你更喜欢使用环境变量来配置你的 client ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      paypal: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        paypal: {
          clientId: '', // NUXT_PUBLIC_SCRIPTS_PAYPAL_CLIENT_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_PAYPAL_CLIENT_ID=<YOUR_CLIENT_ID>
```
