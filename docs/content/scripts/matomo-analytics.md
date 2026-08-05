---
title: Matomo 分析
description: 使用内置的同意控制来跟踪 Matomo 页面浏览和事件。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/matomo-analytics.ts
    size: xs
---

[Matomo Analytics](https://matomo.org/) 可通过 Matomo Cloud 或自托管实例跟踪页面浏览和事件。

::script-stats  
::  

::script-docs  
::  

`watch` 选项默认为 `true`。该组合式函数会立即注册其 `useScriptEventPage` 监听器，与脚本触发器无关。请在初始的 `page:finish` 钩子之前注册组合式函数，以跟踪该路由及之后的导航；在此之后注册则只能捕获后续导航。请显式传入 `siteId`：当前运行时在省略该选项时不会应用其预期的 `1` 备用值。

```ts
useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
  siteId: 2,
  // watch: true, // 默认启用；自动跟踪页面。
})
```

将命令推送到 `_paq`，以设置自定义维度或手动发送事件：

```ts
const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
  siteId: 2,
})

// 设置自定义维度
proxy._paq.push(['setCustomDimension', 1, 'value'])
// 发送页面事件
proxy._paq.push(['trackPageView'])
```

完整选项列表请参阅[配置架构](#config-schema)。

## 自定义页面跟踪

提供 `watch: false` 以禁用内置页面监听器，然后在需要时自行将初始路由加入队列：

```ts
const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID',
  siteId: 2,
  watch: false, // 禁用自动跟踪
})

// watch: false 会禁用内置页面监听器。
proxy._paq.push(['trackPageView'])

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

### 使用自托管的 Matomo

对于自托管的 Matomo，将 `matomoUrl` 设置为自定义跟踪地址。如果使用自定义跟踪端点，也请设置 `trackerUrl`。

```ts
useScriptMatomoAnalytics({
  // 例如，https://your-url.com/matomo.js 和 /matomo.php 都存在。
  matomoUrl: 'https://your-url.com',
  siteId: 2,
})
```

## 同意模式

Matomo 内置了一个由 `requireConsent` 控制的 [跟踪同意 API](https://developer.matomo.org/guides/tracking-consent)。在注册时设置 `defaultConsent` 以启用该控制，然后在运行时调用 `consent.give()`{lang="ts"} / `consent.forget()`{lang="ts"}。

### `defaultConsent`

| 值 | 行为 |
|-------|-----------|
| `'required'` | 推送 `['requireConsent']`。在用户选择同意之前，Matomo 不会进行任何跟踪。 |
| `'given'` | 先推送 `['requireConsent']`，然后推送 `['setConsentGiven']`。立即开始跟踪。 |
| `'not-required'` | Matomo 的默认行为（不进行同意控制）。 |

::callout{icon="i-heroicons-exclamation-triangle" color="warning"}
除非在注册时设置了 `defaultConsent: 'required'` 或 `'given'`，否则 `consent.give()`{lang="ts"} 和 `consent.forget()`{lang="ts"} **不会执行任何操作**。如果尚未推送 `requireConsent`，Matomo 会忽略 `setConsentGiven` 和 `forgetConsentGiven`。如果忘记设置，开发环境中会发出警告。
::

### 示例

```vue
<script setup lang="ts">
const { consent } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID',
  siteId: 2,
  defaultConsent: 'required',
})

function onAccept() {
  consent.give()
}
function onRevoke() {
  consent.forget()
}
</script>
```

### 使用白标 Matomo

对于白标 Matomo 部署，请设置 `trackerUrl` 和 `scriptInput.src` 以自定义跟踪。

```ts
useScriptMatomoAnalytics({
  siteId: 2,
  trackerUrl: 'https://c.staging.cookie3.co/lake',
  scriptInput: {
    src: 'https://cdn.cookie3.co/scripts/latest/cookie3.analytics.min.js',
  },
})
```

::script-types
::
