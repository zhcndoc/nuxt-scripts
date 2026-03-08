---

title: Google Analytics  
description: 在你的 Nuxt 应用中使用 Google Analytics。  
links:  
  - label: 源码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-analytics.ts  
    size: xs  

---

[Google Analytics](https://marketingplatform.google.com/about/analytics/) 是适用于 Nuxt 应用的分析解决方案。

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

::script-types
::

### 客户/消费者 ID 追踪

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

// 火并忘（排队直到 GA 加载）
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

有关同意模式的设置，请参阅 [同意指南](/docs/guides/consent)。
