---
title: Google Analytics
description: 在你的 Nuxt 应用中使用 Google Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-analytics.ts
    size: xs
---

[Google Analytics](https://marketingplatform.google.com/about/analytics/) 是 Nuxt 应用的分析解决方案。

它提供了详细的洞察，让你了解网站的性能、用户如何与内容互动以及用户如何导航你的网站。

在 Nuxt 应用中全局加载 Google Analytics 最简单的方式是通过 Nuxt 配置。或者你也可以直接使用 [useScriptGoogleAnalytics](#useScriptGoogleAnalytics) 组合函数。

### 全局加载

如果你不打算发送自定义事件，可以使用 [环境变量覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 来在开发环境禁用脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleAnalytics: {
        id: 'YOUR_ID',
      }
    }
  }
})
```

```ts [仅生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        googleAnalytics: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```

```ts [环境变量]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleAnalytics: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        googleAnalytics: {
          // .env 文件
          // NUXT_PUBLIC_SCRIPTS_GOOGLE_ANALYTICS_ID=<your-id>
          id: '',
        },
      },
    },
  },
})
```

::

## useScriptGoogleAnalytics

`useScriptGoogleAnalytics` 组合函数让你可以细粒度控制 Google Analytics 在网站上的加载时机和方式。

::code-group

```ts [默认]
const googleAnalytics = useScriptGoogleAnalytics({
  id: 'YOUR_ID'
})
```

```ts [环境变量]
// 1. 设置 .env 中的 NUXT_PUBLIC_SCRIPTS_GOOGLE_ANALYTICS_ID=<your-id>
// 2. 将 runtimeConfig.public.scripts.googleAnalytics.id 设为空字符串或回退值
const googleAnalytics = useScriptGoogleAnalytics()
```

::

请参阅[注册脚本](/docs/guides/registry-scripts)指南，了解更多高级用法。

### 使用

与 Google Analytics API 交互，推荐使用脚本的[代理](/docs/guides/key-concepts#understanding-proxied-functions)。

```ts
const { proxy } = useScriptGoogleAnalytics()

proxy.gtag('event', 'page_view')
```

代理对象暴露了 `gtag` 和 `dataLayer` 属性，应遵循 Google Analytics 的最佳实践使用它们。

### GoogleAnalyticsApi

```ts
export interface GTag {
  // 使用时间戳初始化 gtag.js
  (command: 'js', value: Date): void

  // 配置一个 GA4 属性
  (command: 'config', targetId: string, configParams?: ConfigParams): void

  // 从 gtag 获取值
  (command: 'get', targetId: string, fieldName: string, callback?: (field: any) => void): void

  // 向 GA4 发送事件
  (command: 'event', eventName: DefaultEventName, eventParams?: EventParameters): void

  // 为所有后续事件设置默认参数
  (command: 'set', params: GtagCustomParams): void

  // 更新同意状态
  (command: 'consent', consentArg: 'default' | 'update', consentParams: ConsentOptions): void
}

interface GoogleAnalyticsApi {
  dataLayer: Record<string, any>[]
  gtag: GTag
}
```

### 配置 Schema

首次设置脚本时必须提供选项。

```ts
export const GoogleAnalyticsOptions = object({
  /**
   * Google Analytics ID。可选 - 允许加载 gtag.js 但不做初始配置。
   */
  id: optional(string()),
  /**
   * 你希望关联的数据层（datalayer）的名称
   */
  l: optional(string())
})
```

### 客户/用户 ID 追踪

对于需要同时追踪客户特定分析的电子商务或多租户应用：

```vue [ProductPage.vue]
<script setup lang="ts">
// 带有客户特定追踪的产品页面
const route = useRoute()
const pageData = await $fetch(`/api/product/${route.params.id}`)

// 以自定义 dataLayer 名加载 gtag 以进行客户追踪
const { proxy: customerGtag, load } = useScriptGoogleAnalytics({
  key: 'gtag-customer',
  l: 'customerDataLayer', // 自定义 dataLayer 名称
})

// 当可用时配置客户的追踪 ID
const consumerGtagId = computed(() => pageData?.gtag)

if (consumerGtagId.value) {
  // 配置客户的 GA4 属性
  customerGtag.gtag('config', consumerGtagId.value)

  // 发送客户特定事件
  customerGtag.gtag('event', 'product_view', {
    item_id: pageData.id,
    customer_id: pageData.customerId,
    value: pageData.price
  })
}
</script>
```

## 示例

仅在生产环境使用 Google Analytics，同时用 `gtag` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptGoogleAnalytics()

// 开发环境及 SSR 中为无操作
// 仅生产环境和客户端生效
proxy.gtag('event', 'conversion-test')
function sendConversion() {
  proxy.gtag('event', 'conversion')
}
</script>

<template>
  <div>
    <button @click="sendConversion">
      发送转化
    </button>
  </div>
</template>
```

::

## 自定义维度和用户属性

```ts
const { proxy } = useScriptGoogleAnalytics()

// 用户属性（跨会话持久化）
proxy.gtag('set', 'user_properties', {
  user_tier: 'premium',
  account_type: 'business'
})

// 带自定义维度的事件（在 GA4 管理 > 自定义定义中注册）
proxy.gtag('event', 'purchase', {
  transaction_id: 'T12345',
  value: 99.99,
  payment_method: 'credit_card',  // 自定义维度
  discount_code: 'SAVE10'         // 自定义维度
})

// 以后所有事件的默认参数
proxy.gtag('set', { country: 'US', currency: 'USD' })
```

## 手动页面浏览追踪（SPA）

GA4 默认自动追踪页面浏览。若想禁用并手动追踪：

```ts
const { proxy } = useScriptGoogleAnalytics()

// 禁用自动页面浏览
proxy.gtag('config', 'G-XXXXXXXX', { send_page_view: false })

// 路由切换时手动追踪
const router = useRouter()
router.afterEach((to) => {
  proxy.gtag('event', 'page_view', { page_path: to.fullPath })
})
```

## 代理排队

代理会在脚本加载前排队所有的 `gtag` 调用。调用可在服务端渲染时安全使用，抗广告拦截且保持顺序。

```ts
const { proxy, onLoaded } = useScriptGoogleAnalytics()

// 立即调用（排队等待 GA 加载）
proxy.gtag('event', 'cta_click', { button_id: 'hero-signup' })

// 需要返回值时等待加载完成
onLoaded(({ gtag }) => {
  gtag('get', 'G-XXXXXXXX', 'client_id', (id) => console.log(id))
})
```

## 常用事件模式

```ts
const { proxy } = useScriptGoogleAnalytics()

// 电子商务
proxy.gtag('event', 'purchase', {
  transaction_id: 'T_12345',
  value: 59.98,
  currency: 'USD',
  items: [{ item_id: 'SKU_12345', item_name: 'Widget', price: 29.99, quantity: 2 }]
})

// 交互事件
proxy.gtag('event', 'login', { method: 'Google' })
proxy.gtag('event', 'search', { search_term: 'nuxt scripts' })

// 自定义事件
proxy.gtag('event', 'feature_used', { feature_name: 'dark_mode' })
```

## 调试

通过配置或 URL 参数 `?debug_mode=true` 启用调试模式：

```ts
proxy.gtag('config', 'G-XXXXXXXX', { debug_mode: true })
```

在 GA4 查看事件：**管理员 > 调试视图**。安装 [GA 调试器插件](https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna) 以便控制台日志。

有关同意模式设置，请参阅[同意指南](/docs/guides/consent)。
