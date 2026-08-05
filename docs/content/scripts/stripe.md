---
title: Stripe
description: 以编程方式加载 Stripe.js，或嵌入延迟加载的 Stripe 定价表。
links:
  - label: useScriptStripe
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/stripe.ts
    size: xs
  - label: "<ScriptStripePricingTable>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptStripePricingTable.vue
    size: xs
---

[Stripe](https://stripe.com) 提供支付 API 和托管式结账组件。

Nuxt Scripts 提供两项 Stripe 功能：

- [`useScriptStripe()`{lang="ts"}](/scripts/stripe){lang="ts"} 组合式函数，默认加载脚本 `https://js.stripe.com/basil/stripe.js`。使用 `version` 选项可以[固定 Stripe.js SDK 版本](https://docs.stripe.com/sdks/stripejs-versioning)（例如 `acacia`、`clover`、`dahlia`）。
- `ScriptStripePricingTable` 组件，用于通过 `https://js.stripe.com/v3/pricing-table.js` 嵌入 [Stripe 定价表](https://docs.stripe.com/payments/checkout/pricing-table)。

## 类型

::script-docs
::

## 类型

安装 `@stripe/stripe-js` 以获得完整的 TypeScript 支持。

```bash
pnpm add -D @stripe/stripe-js
```

## 全局加载

Stripe 建议在每个页面上加载 Stripe.js，以便其[高级欺诈检测](https://docs.stripe.com/disputes/prevention/advanced-fraud-detection)功能能够在结账前观察浏览行为。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      stripe: { trigger: 'onNuxtReady' },
    }
  }
})
```

```ts [仅生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        stripe: { trigger: 'onNuxtReady' },
      }
    }
  }
})
```

::

## ScriptStripePricingTable

`ScriptStripePricingTable` 嵌入 [Stripe 价格表](https://docs.stripe.com/payments/checkout/pricing-table)，并将其脚本的加载延迟到组件可见时。

[元素事件触发器](/docs/guides/script-triggers#element-event-triggers) 会将价格表脚本的加载延迟到组件变为可见时。

::callout  
你需要先创建你自己的 [价格表](https://dashboard.stripe.com/pricing-tables) 才能继续使用。  
::

### 演示

::code-group

:stripe-demo{label="输出"}

```vue [输入]
<template>
  <ScriptStripePricingTable
    pricing-table-id="prctbl_1PD0MMEclFNgdHcR8t0Jop2H"
    publishable-key="pk_live_51OhXSKEclFNgdHcRNi5xBjBClxsA0alYgt6NzBwUZ880pLG88rYSCYPQqpzM3TedzNYu5g2AynKiPI5QVLYSorLJ002iD4VZIB"
  />
</template>
```

::

### 组件 API

完整的属性、事件和插槽请参见 [Facade 组件接口](/docs/guides/facade-components#facade-components-api)。

### 属性

当你需要加载 Stripe.js 并以编程方式调用它时，请使用 [`useScriptStripe()`{lang="ts"}](/scripts/stripe){lang="ts"}。

```ts
export function useScriptStripe<T extends StripeApi>(_options?: StripeInput) {}
```

有关触发器和其他脚本选项，请参见 [注册表脚本](/docs/guides/registry-scripts)。

::script-types
::

## 示例

这会为一笔金额为 10.99 美元的 USD 支付挂载 Payment Element。你仍然需要在服务器上完成支付；Stripe 的[延迟意图指南](https://docs.stripe.com/payments/accept-a-payment-deferred)介绍了完整流程。

```vue
<script setup lang="ts">
const paymentEl = ref<HTMLElement | null>(null)
const { onLoaded } = useScriptStripe()
onMounted(() => {
  onLoaded(({ Stripe }) => {
    const stripe = Stripe('YOUR_STRIPE_KEY')
    const elements = stripe.elements({
      mode: 'payment',
      amount: 1099,
      currency: 'usd',
    })
    const paymentElement = elements.create('payment')
    if (paymentEl.value)
      paymentElement.mount(paymentEl.value)
  })
})
</script>

<template>
  <div ref="paymentEl" />
</template>
```
