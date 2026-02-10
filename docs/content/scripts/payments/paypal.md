---
title: PayPal
description: 在您的 Nuxt 应用中使用 PayPal。
links:
  - label: useScriptPayPal
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/paypal.ts
    size: xs
  - label: "<ScriptPayPalButtons>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptPayPalButtons.vue
    size: xs
  - label: "<ScriptPayPalMarks>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptPayPalMarks.vue
    size: xs
  - label: "<ScriptPayPalMessages>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptPayPalMessages.vue
    size: xs
---

[PayPal](https://www.paypal.com) 是一个流行的支付网关，允许您在线接受付款。

Nuxt Scripts 提供了多种 PayPal 功能：
- `useScriptPayPal` 组合函数，用于加载脚本 `https://www.paypal.com/sdk/js`。
- `ScriptPayPalButtons` 组件，允许您在网站中嵌入 [PayPal 按钮](https://developer.paypal.com/sdk/js/reference/#buttons)。
- `ScriptPayPalMarks` 组件，允许您在网站中嵌入 [PayPal 标志](https://developer.paypal.com/sdk/js/reference/#marks)。
- `ScriptPayPalMessages` 组件，允许您在网站中嵌入 [PayPal 消息](https://developer.paypal.com/studio/checkout/pay-later/us/customize/reference)。

## 类型

要使用带有完整 TypeScript 支持的 PayPal，您需要安装 `@paypal/paypal-js` 依赖。

```bash
pnpm add -D @paypal/paypal-js
```
### 演示

::code-group

:pay-pal-demo{label="输出"}

```vue [Input]
<template>
  <div>
    <ScriptPayPalButtons
      class="border border-gray-200 dark:border-gray-800 rounded-lg"
      :button-options="buttonOptions"
      :disabled="disabled"
    />
    <label>
      禁用
      <input v-model="disabled" type="checkbox">
    </label>
    <ScriptPayPalMarks />
    <ScriptPayPalMessages :messages-options="{ style: { color: 'white-no-border', layout: 'flex' } }" />
  </div>
</template>

<script setup lang="ts">
  import { computed, ref } from 'vue'
  import type { PayPalButtonsComponentOptions } from '@paypal/paypal-js'

  const buttonOptions = computed(() => ({
    style: {
      layout: 'vertical',
      color: 'blue',
      shape: 'rect',
      label: 'paypal',
    },
    message: { amount: '10.00' },
  } satisfies PayPalButtonsComponentOptions))

  const disabled = ref(false)
</script>
```

::

#### 使用环境变量

如果您更喜欢使用环境变量来配置客户端 ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      paypal: true,
    }
  },
  // 需要提供运行时配置以访问环境变量
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