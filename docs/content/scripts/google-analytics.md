---
title: 谷歌分析
description: 在 Nuxt 应用中使用 Google Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-analytics.ts
    size: xs
---

[谷歌分析](https://marketingplatform.google.com/about/analytics/) 是适用于 Nuxt 应用的分析解决方案。

它提供了详尽的洞察，帮助你了解网站的性能表现、用户如何与你的内容互动，以及他们如何浏览你的网站。

::script-stats
::

::script-docs
::

### 使用方法

为了与 Google Analytics API 交互，推荐使用脚本的 [proxy](/docs/guides/key-concepts#understanding-proxied-functions)。

```ts
const { proxy } = useScriptGoogleAnalytics()

proxy.gtag('event', 'page_view')
```

proxy 暴露了 `gtag` 和 `dataLayer` 属性，你应当按照 Google Analytics 的最佳实践使用它们。

## 同意模式

Google Analytics 原生支持 [GCMv2 同意状态](https://developers.google.com/tag-platform/security/guides/consent)。使用 `defaultConsent` 设置默认值（会在 `gtag('js', ...)`{lang="ts"} 之前触发 `gtag('consent', 'default', state)`{lang="ts"}），并在运行时调用 `consent.update()`{lang="ts"} 来切换各项类别。对于需要在运行时动态推导默认值（例如在触发前等待地区/CMS 解析完成）的场景，可在客户端调用 `consent.default()`{lang="ts"}。

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/regional-consent" target="_blank"}
在 [StackBlitz](https://stackblitz.com) 上试用实时的 [区域同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/regional-consent)。
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

向 `defaultConsent` 传入一个数组，即可为每个条目分别触发一次 `gtag('consent','default', state)`{lang="ts"}。这与 Google 的 [特定地区同意模式](https://developers.google.com/tag-platform/security/guides/consent?consentmode=advanced#region-specific-behavior) 一致：更具体的地区（例如 `US-CA`）会覆盖更宽泛的地区（`US`）；没有 `region` 的条目则是不带范围的全局回退值。

```vue
<script setup lang="ts">
useScriptGoogleAnalytics({
  id: 'G-XXXXXXXX',
  defaultConsent: [
    {
      // 欧盟 + 英国 + 瑞士——默认拒绝，等待用户选择 500 毫秒
      ad_storage: 'denied',
      ad_user_data: 'denied',
      ad_personalization: 'denied',
      analytics_storage: 'denied',
      region: ['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB', 'IS', 'LI', 'NO', 'CH'],
      wait_for_update: 500,
    },
    {
      // 其他所有地区——默认授予
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

## 客户/消费者 ID 跟踪

对于需要在主追踪之外进行客户特定数据追踪的电商或多租户应用：

```vue [ProductPage.vue]
<script setup lang="ts">
// 带有客户特定追踪的产品页面
const route = useRoute()
const pageData = await $fetch(`/api/product/${route.params.id}`)

// 使用自定义 dataLayer 名称加载 gtag 以进行客户追踪
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

## 自定义维度和用户属性

```ts
const { proxy } = useScriptGoogleAnalytics()

// 用户属性（跨会话持久化）
proxy.gtag('set', 'user_properties', {
  user_tier: 'premium',
  account_type: 'business'
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

## 手动页面浏览追踪（单页应用）

GA4 默认自动追踪页面浏览。若需禁用并手动追踪：

```ts
const { proxy } = useScriptGoogleAnalytics()

// 禁用自动页面浏览
proxy.gtag('config', 'G-XXXXXXXX', { send_page_view: false })

// 路由变化时追踪
const router = useRouter()
router.afterEach((to) => {
  proxy.gtag('event', 'page_view', { page_path: to.fullPath })
})
```

## 代理排队机制

proxy 会将所有 `gtag` 调用排队，直到脚本加载完成。调用是 SSR 安全的，对广告屏蔽器友好，并且保持调用顺序。

```ts
const { proxy, onLoaded } = useScriptGoogleAnalytics()

// 发后即忘（排队直到 GA 加载）
proxy.gtag('event', 'cta_click', { button_id: 'hero-signup' })

// 需要返回值？等待加载完成
onLoaded(({ gtag }) => {
  gtag('get', 'G-XXXXXXXX', 'client_id', id => console.log(id))
})
```

## 常见事件模式

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

通过配置或 URL 参数 `?debug_mode=true` 启用调试模式：

```ts
proxy.gtag('config', 'G-XXXXXXXX', { debug_mode: true })
```

在 GA4 查看事件：**管理 > DebugView**。安装 [GA Debugger 扩展](https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna) 以在控制台中查看日志。

关于同意模式设置，请参阅 [同意指南](/docs/guides/consent)。

::script-types
::
