---
title: Ahrefs Web Analytics
description: 在你的 Nuxt 应用中使用 Ahrefs Web Analytics，通过隐私优先、无 Cookie 的分析脚本来跟踪页面浏览量和自定义事件。
links:
  - label: 来源
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/ahrefs-analytics.ts
    size: xs
---

[Ahrefs Web Analytics](https://ahrefs.com/web-analytics) 是来自 [Ahrefs](https://ahrefs.com) 的一项隐私优先、无 Cookie 的 Web 分析服务，可在不与第三方共享访客数据的情况下跟踪页面浏览量和自定义事件。

::script-stats
::

::script-docs
::

该组合式函数具有以下默认值：
- **触发时机：客户端** 当 Nuxt 正在挂载时，脚本将加载。

你可以直接将 `AhrefsAnalytics` 对象作为代理访问，或者等待 `$script` Promise 以访问该对象。对于任何无返回值函数，建议使用代理。

::code-group

```ts [Proxy]
const { proxy } = useScriptAhrefsAnalytics({
  key: 'your-project-key',
})
function trackSignup() {
  proxy.AhrefsAnalytics.sendEvent('signup', {
    props: { plan: 'pro' },
  })
}
```

```ts [onLoaded]
const { onLoaded } = useScriptAhrefsAnalytics({
  key: 'your-project-key',
})
onLoaded(({ AhrefsAnalytics }) => {
  AhrefsAnalytics.sendEvent('signup', {
    props: { plan: 'pro' },
  })
})
```

::

## SPA 导航

Ahrefs Analytics 原生跟踪单页应用的导航：加载后的 `analytics.js` 会补丁 `history.pushState` 并监听 `popstate`，每当 URL 发生变化时都会触发一次新的页面浏览。对于 Nuxt 路由变化，无需额外配置。

::script-types
::

## 示例

在 Nuxt 准备就绪时，通过 `app.vue` 加载 Ahrefs Web Analytics。

```vue [app.vue]
<script setup lang="ts">
useScriptAhrefsAnalytics({
  key: 'your-project-key',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```
