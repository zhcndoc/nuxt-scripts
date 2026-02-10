---
title: Stripe
description: 在你的 Nuxt 应用中使用 Stripe。
links:
  - label: useScriptStripe
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/stripe.ts
    size: xs
  - label: "<ScriptStripePricingTable>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptStripePricingTable.vue
    size: xs
---

[Stripe](https://stripe.com) 是一个流行的支付网关，允许你在线接受付款。

Nuxt Scripts 提供了两个 Stripe 功能：
- `useScriptStripe` 组合函数，会加载脚本 `https://js.stripe.com/v3/`。
- `ScriptStripePricingTable` 组件，允许你使用 `https://js.stripe.com/v3/pricing-table.js` 在你的网站上嵌入一个 [Stripe 价格表](https://docs.stripe.com/payments/checkout/pricing-table)。

## 类型

为了在使用 Stripe 时获得完整的 TypeScript 支持，你需要安装 `@stripe/stripe-js` 依赖。

```bash
pnpm add -D @stripe/stripe-js
```

## 全局加载

Stripe 建议在你的应用中全局加载其脚本，以增强防欺诈检测。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      stripe: true,
    }
  }
})
```

```ts [仅生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        stripe: true,
      }
    }
  }
})
```

::

## ScriptStripePricingTable

`ScriptStripePricingTable` 组件允许你以优化的方式在网站上嵌入一个 [Stripe 价格表](https://docs.stripe.com/payments/checkout/pricing-table)。

为了提升性能，它会在价格表元素可见时加载，利用了 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。

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

`ScriptStripePricingTable` 组件接受以下属性：

- `trigger`：触发加载 Stripe 的事件。默认为 `mouseover`。更多信息请参见 [元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。
- `pricing-table-id`：你在 Stripe 控制台创建的价格表 ID。
- `publishable-key`：你的 Stripe 可发布密钥。
- `client-reference-id`：客户的唯一标识符。
- `customer-email`：客户的电子邮件。
- `customer-session-client-secret`：客户会话的客户端密钥。

## useScriptStripe

`useScriptStripe` 组合函数让你可以精细控制 Stripe SDK。它提供了一种加载 Stripe SDK 并以编程方式与之交互的方法。

```ts
export function useScriptStripe<T extends StripeApi>(_options?: StripeInput) {}
```

请参考 [注册表脚本](/docs/guides/registry-scripts) 指南了解更多高级用法。

### 选项

```ts
export const StripeOptions = object({
  advancedFraudSignals: optional(boolean()),
})
```

### StripeApi

```ts
export interface StripeApi {
  Stripe: stripe.StripeStatic
}
```

## 示例

加载 Stripe SDK 并使用它创建支付元素。

```vue
<script setup lang="ts">
const paymentEl = ref(null)
const { onLoaded } = useScriptStripe()
onMounted(() => {
  onLoaded(({ Stripe }) => {
    const stripe = Stripe('YOUR_STRIPE_KEY')
    const elements = stripe.elements()
    const paymentElement = elements.create('payment', { /* 传入选项 */ })
    paymentElement.mount(paymentEl.value)
  })
})
</script>

<template>
  <div ref="paymentEl" />
</template>
```