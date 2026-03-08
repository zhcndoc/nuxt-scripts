---

title: Matomo 分析  
description: 在你的 Nuxt 应用中使用 Matomo 分析。  
links:  
  - label: 源码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/matomo-analytics.ts  
    size: xs  

---

[Matomo 分析](https://matomo.org/) 是一个非常适合 Nuxt 应用的分析解决方案。

它提供了对你的网站性能、用户如何与内容互动及如何浏览你网站的详细洞察。

::script-stats  
::  

::script-docs  
::  

::script-types  
::  

默认情况下，Nuxt 使用 `siteId` 为 `1`，并且通过 `watch` 选项自动启用页面跟踪。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
  siteId: 2,
  // watch: true, // 默认启用 - 自动页面跟踪！
})
```

如果你想对跟踪有更多控制，例如设置自定义维度，可以使用 `proxy` 对象发送事件。

```ts
const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
})

// 设置自定义维度
proxy._paq.push(['setCustomDimension', 1, 'value'])
// 发送页面事件
proxy._paq.push(['trackPageView'])
```

请参阅[配置架构](#config-schema)了解所有可用选项。

## 自定义页面跟踪

默认情况下，Nuxt 会自动跟踪所有页面；如需禁用自动跟踪，请设置 `watch: false`。

```ts
import { useScriptEventPage } from '#nuxt-scripts'

const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID',
  watch: false, // 禁用自动跟踪
})

// 带有额外逻辑的自定义页面跟踪
useScriptEventPage((payload) => {
  // 根据路由设置自定义维度
  if (payload.path.startsWith('/products')) {
    proxy._paq.push(['setCustomDimension', 1, '产品页'])
  }

  // 标准的 Matomo 跟踪调用（与内置 watch 行为相同）
  proxy._paq.push(['setDocumentTitle', payload.title])
  proxy._paq.push(['setCustomUrl', payload.path])
  proxy._paq.push(['trackPageView'])

  // 跟踪额外的自定义事件
  proxy._paq.push(['trackEvent', '导航', '页面浏览', payload.path])
})
```

### 使用自托管 Matomo

对于自托管 Matomo，设置 `matomoUrl` 以自定义跟踪，如果你自定义了 `trackerUrl`，可能需要设置它。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  // 例如 https://your-url.com/tracker.js 和 https://your-url.com/matomo.php 同时存在
  matomoUrl: 'https://your-url.com',
})
```

### 使用 Matomo 白标版

对于 Matomo 白标版，设置 `trackerUrl` 和 `scriptInput.src` 以自定义跟踪。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  trackerUrl: 'https://c.staging.cookie3.co/lake',
  scriptInput: {
    src: 'https://cdn.cookie3.co/scripts/latest/cookie3.analytics.min.js',
  },
})
```
