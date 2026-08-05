---
title: Cloudflare Web Analytics
description: 在水合期间加载 Cloudflare Web Analytics，并启用其内置的 SPA 测量功能。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/cloudflare-web-analytics.ts
    size: xs
---

[Cloudflare Web Analytics](https://developers.cloudflare.com/web-analytics/about/) 报告流量和 Web Vitals。其信标 [不使用 Cookie 或本地存储](https://developers.cloudflare.com/web-analytics/data-metrics/core-web-vitals/#information-collected)，并且该公司表示，该产品不会收集或使用访问者的个人数据。

::script-stats  
::

::script-docs  
::

默认：

- **触发器：客户端** 脚本会在 Nuxt 水合期间加载，以保持 Web Vitals 指标的准确性。

此注册表将触发器固定为 `client`。调用方传入的 `scriptOptions.trigger` 值会被覆盖，因此当前无法通过同意或元素触发器延迟此集成。

::callout{type="warning"}
当前运行时始终启用 SPA 跟踪，尽管 [Cloudflare 将 `spa: false` 记录为禁用 SPA 分析的开关](https://developers.cloudflare.com/web-analytics/get-started/web-analytics-spa/#disable-spa-analytics)。目前向此注册表传递 `spa: false` 不会产生任何效果。
::

## 在 app.vue 中加载

在 `app.vue` 中注册 Beacon：

```vue [app.vue]
<script setup lang="ts">
useScriptCloudflareWebAnalytics({
  token: '12ee46bf598b45c2868bbc07a3073f58',
})
</script>
```

分析脚本会创建一个 `window.__cfBeacon` 对象。在脚本加载后访问它：

```ts
const { onLoaded } = useScriptCloudflareWebAnalytics({
  token: '12ee46bf598b45c2868bbc07a3073f58',
})
onLoaded(({ __cfBeacon }) => {
  console.log(__cfBeacon)
})
```

::script-types
::
