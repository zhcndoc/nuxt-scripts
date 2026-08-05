---
title: Ahrefs Web Analytics
description: 在 Nuxt 中使用 Ahrefs Web Analytics 跟踪页面浏览量和自定义事件。
links:
  - label: 来源
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/ahrefs-analytics.ts
    size: xs
---

[Ahrefs Web Analytics](https://help.ahrefs.com/en/articles/10247870-about-ahrefs-web-analytics) 无需使用 Cookie 或持久化标识符即可跟踪页面浏览量和自定义事件。它会在推导出粗略位置数据后丢弃原始 IP 地址。

::script-stats
::

::script-docs
::

默认：

- **触发器：`onNuxtReady`** 脚本会在 Nuxt 应用准备就绪时加载。

对于无返回值调用，请使用代理。需要加载完成的 `AhrefsAnalytics` 对象时，请等待 `$script`。[Ahrefs 跟踪事件指南](https://help.ahrefs.com/en/articles/11381932-tracked-events-in-ahrefs-web-analytics)介绍了 JavaScript API 和事件属性结构。

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

Ahrefs Analytics 原生跟踪单页应用导航：加载的 [`analytics.js`](https://analytics.ahrefs.com/analytics.js) 会修改 `history.pushState` 并监听 `popstate`，在 URL 发生变化时触发新的页面浏览。Nuxt 路由变化无需额外配置。

::script-types
::
