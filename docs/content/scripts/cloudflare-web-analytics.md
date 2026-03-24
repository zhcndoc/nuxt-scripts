---
title: Cloudflare 网站分析
description: 在你的 Nuxt 应用中使用 Cloudflare 网站分析。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/cloudflare-web-analytics.ts
    size: xs
---

[Cloudflare Web Analytics](https://developers.cloudflare.com/analytics/web-analytics/) 是适用于 Nuxt 的绝佳隐私保护分析解决方案。它为你的网站提供免费的、以隐私为中心的分析服务。它不会从访客那里收集任何个人身份信息，却能提供详细的洞察，展示页面在访客体验中的表现。

::script-stats  
::

::script-docs  
::

该组合式函数具有以下默认设置：  
- **客户端加载**  脚本将在 Nuxt 完成水合（hydration）时加载，以确保你的网页关键性能指标（Web Vitals）保持准确。

## 在 app.vue 中加载

通过 `app.vue` 在 Nuxt 就绪时加载 Cloudflare 网站分析。

```vue [app.vue]
<script setup lang="ts">
useScriptCloudflareWebAnalytics({
  token: '12ee46bf598b45c2868bbc07a3073f58',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```

Cloudflare 网站分析组合式函数会向全局作用域注入一个 `window.__cfBeacon` 对象。如果你需要访问它，可以等待脚本加载完成后获取。

```ts
const { onLoaded } = useScriptCloudflareWebAnalytics()
onLoaded(({ cfBeacon }) => {
  console.log(cfBeacon)
})
```

::script-types
::
