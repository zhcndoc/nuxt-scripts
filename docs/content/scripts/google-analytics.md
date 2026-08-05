---
title: Google Analytics
description: 使用类型化的同意控制发送 Google Analytics 页面浏览和事件。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-analytics.ts
    size: xs
---

[Google Analytics](https://marketingplatform.google.com/about/analytics/) 用于记录页面浏览和事件，以分析流量和受众。

::script-stats
::

::script-docs
::

### 使用方法

通过脚本 [代理](/docs/guides/key-concepts#understanding-proxied-functions) 调用 [`gtag`](https://developers.google.com/tag-platform/gtagjs/reference)：

```ts
const { proxy } = useScriptGoogleAnalytics({ id: 'G-XXXXXXXX' })

proxy.gtag('event', 'page_view')
```

代理还会公开 `dataLayer`。

下面省略 `id` 的示例假定已通过 `scripts.registry.googleAnalytics` 提供该参数。

## 同意模式

Google Analytics 接受 [GCMv2 同意状态](https://developers.google.com/tag-platform/security/guides/consent)。`defaultConsent` 会在 `gtag('js', ...)`{lang="ts"} 之前触发；对于之后的选择，请使用 `consent.update()`{lang="ts"}。在组合式函数返回后调用 `consent.default()`{lang="ts"} 会将新的默认值排入队列，但无法复现这种初始化前的顺序。

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/regional-consent" target="_blank"}
在 [StackBlitz](https://stackblitz.com) 上打开[按地区设置同意的示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/regional-consent)。
::

```vue
<script setup lang="ts">
const { consent } = useScriptGoogleAnalytics({
  id: 'G-XXXXXXXX',
  defaultConsent: {
    ad_storage: 'denied',
    ad_user_data: 'denied',
    ad_personalization: 'denied',
    analytics_storage: 'denied',
  },
})

function acceptAll() {
  consent.update({
    ad_storage: 'granted',
    ad_user_data: 'granted',
    ad_personalization: 'granted',
    analytics_storage: 'granted',
  })
}

function savePreferences(choices: { analytics: boolean, marketing: boolean }) {
  consent.update({
    analytics_storage: choices.analytics ? 'granted' : 'denied',
    ad_storage: choices.marketing ? 'granted' : 'denied',
    ad_user_data: choices.marketing ? 'granted' : 'denied',
    ad_personalization: choices.marketing ? 'granted' : 'denied',
  })
}
</script>
```

`consent.update()`{lang="ts"} 和 `consent.default()`{lang="ts"} 都接受任何 `Partial<ConsentState>`{lang="ts"}；缺失的类别会保持当前值不变。两种方法都会根据标准的 GCMv2 模式验证输入，并在出现未知键或非 `granted`/`denied` 值时通过 `consola` 发出警告。对于除同意默认值之外、在 `gtag('js')`{lang="ts"} 之前进行的设置，`onBeforeGtagStart` 仍然可用作通用的逃生通道。

### 按地区的默认值

向 `defaultConsent` 传入一个数组，即可为每个条目分别触发一次 `gtag('consent','default', state)`{lang="ts"}。这与 Google 的[特定地区同意模式](https://developers.google.com/tag-platform/security/guides/consent?consentmode=advanced#region-specific-behavior)一致：更具体的地区（例如 `US-CA`）会覆盖更宽泛的地区（`US`）；没有 `region` 的条目则是不带范围的全局回退值。

```vue
<script setup lang="ts">
useScriptGoogleAnalytics({
  id: 'G-XXXXXXXX',
  defaultConsent: [
    {
      // 欧洲经济区 + 英国 + 瑞士：初始设为拒绝，并等待 500 毫秒以获取选择。
      ad_storage: 'denied',
      ad_user_data: 'denied',
      ad_personalization: 'denied',
      analytics_storage: 'denied',
      region: ['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB', 'IS', 'LI', 'NO', 'CH'],
      wait_for_update: 500,
    },
    {
      // 其他所有地区：默认设为同意。
      ad_storage: 'granted',
      ad_user_data: 'granted',
      ad_personalization: 'granted',
      analytics_storage: 'granted',
    },
  ],
})
</script>
```

模块会按输入顺序原样转发每个条目。区域作用域默认值与无作用域默认值之间的优先级由运行时的 gtag 负责处理，而不是由顺序决定。

## 客户专属 GA 属性

对于市场平台或多租户应用，第二个标签可以将事件发送到客户的 GA 属性：

```vue [ProductPage.vue]
<script setup lang="ts">
// 带有客户特定追踪的产品页面
const route = useRoute()
const pageData = await $fetch(`/api/product/${route.params.id}`)

const consumerGtagId = pageData.gtag

// 使用自定义数据层名称加载 gtag，以进行客户追踪。
const { proxy: customerGtag } = useScriptGoogleAnalytics({
  key: 'gtag-customer',
  id: consumerGtagId,
  l: 'customerDataLayer',
})

customerGtag.gtag('event', 'product_view', {
  item_id: pageData.id,
  customer_id: pageData.customerId,
  value: pageData.price,
})
</script>
```

## 自定义维度和用户属性

Google 在[此处](https://developers.google.com/tag-platform/gtagjs/reference#parameter_scope)记录了 [`config`、`set` 和事件参数](https://developers.google.com/tag-platform/gtagjs/reference#parameter_scope)的作用域。对于与单个 GA 属性相关联的值，请使用 `config`；对于全局默认值，请使用 `set`。

```ts
const { proxy } = useScriptGoogleAnalytics()

// 此 GA4 属性的用户属性
proxy.gtag('config', 'G-XXXXXXXX', {
  user_properties: {
    user_tier: 'premium',
    account_type: 'business',
  },
})

// 事件中的自定义维度（需在 GA4 管理后台 > 自定义定义中注册）
proxy.gtag('event', 'purchase', {
  transaction_id: 'T12345',
  value: 99.99,
  payment_method: 'credit_card', // 自定义维度
  discount_code: 'SAVE10' // 自定义维度
})

// 所有未来事件的默认参数
proxy.gtag('set', { country: 'US', currency: 'USD' })
```

## 跟踪 SPA 页面浏览量

该注册表会配置 Google 标签并发送初始页面浏览量。`useScriptEventPage` 会监听 Nuxt 的 `page:finish` 钩子，其中包括初始客户端渲染期间触发的钩子（前提是你足够早地注册它）。在 `app.vue` 这类长生命周期组件中，应忽略该初始回调，但不要在稍后注册时误丢弃第一次导航。按照 Google 的 [SPA 测量指南](https://developers.google.com/analytics/devguides/collection/ga4/single-page-applications)中的说明，将之前的 URL 保留为 `page_referrer`：

```ts
const { proxy } = useScriptGoogleAnalytics()
const initialPath = useRoute().fullPath
let initialPageSeen = false
let previousLocation: string | undefined

useScriptEventPage(({ title, path }) => {
  const pageLocation = new URL(path, window.location.origin).href

  // page:finish 可能会在初始客户端渲染时触发。
  if (!initialPageSeen && path === initialPath) {
    initialPageSeen = true
    previousLocation = pageLocation
    return
  }

  previousLocation ||= new URL(initialPath, window.location.origin).href
  proxy.gtag('event', 'page_view', {
    page_title: title,
    page_location: pageLocation,
    page_referrer: previousLocation,
  })
  previousLocation = pageLocation
})
```

如果你的 Web 数据流的增强型衡量功能已经[跟踪浏览器历史记录变化](https://developers.google.com/analytics/devguides/collection/ga4/views)，请不要添加手动页面浏览量，否则会记录重复事件。

## 代理队列

代理会在脚本加载完成前将 `gtag` 调用排队，并保留其调用顺序。

```ts
const { proxy, onLoaded } = useScriptGoogleAnalytics()

// 发后即忘（排队直到 GA 加载）
proxy.gtag('event', 'cta_click', { button_id: 'hero-signup' })

// 需要返回值？等待加载完成
onLoaded(({ gtag }) => {
  gtag('get', 'G-XXXXXXXX', 'client_id', id => console.log(id))
})
```

## 常见事件

当某个操作有对应的事件时，请使用 Google 的[推荐事件](https://developers.google.com/analytics/devguides/collection/ga4/events)；对于应用特定的行为，仍然可以使用自定义名称。

```ts
const { proxy } = useScriptGoogleAnalytics()

// 电商
proxy.gtag('event', 'purchase', {
  transaction_id: 'T_12345',
  value: 59.98,
  currency: 'USD',
  items: [{ item_id: 'SKU_12345', item_name: 'Widget', price: 29.99, quantity: 2 }]
})

// 互动
proxy.gtag('event', 'login', { method: 'Google' })
proxy.gtag('event', 'search', { search_term: 'nuxt scripts' })

// 自定义
proxy.gtag('event', 'feature_used', { feature_name: 'dark_mode' })
```

## 调试

为标签配置启用调试模式：

```ts
proxy.gtag('config', 'G-XXXXXXXX', { debug_mode: true })
```

使用 [DebugView、Tag Assistant 或浏览器的网络面板](https://developers.google.com/analytics/devguides/collection/ga4/troubleshoot) 验证标签。在网络面板中，查找对 `google-analytics.com/g/collect` 或 `analytics.google.com/g/collect` 的成功请求。

关于同意模式设置，请参阅 [同意指南](/docs/guides/consent)。

::script-types
::
