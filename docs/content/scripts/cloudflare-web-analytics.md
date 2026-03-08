---

title: Cloudflare 网站分析  
description: 在你的 Nuxt 应用中使用 Cloudflare 网站分析。  
links:  
  - label: 源码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/cloudflare-web-analytics.ts  
    size: xs  

---

[Cloudflare 网站分析](https://developers.cloudflare.com/analytics/web-analytics/) 与 Nuxt 配合使用是一个极佳的隐私分析解决方案。它为你的网站提供免费的、以隐私为中心的分析。该方案不会收集访客的个人数据，却能提供详尽的见解，展示访客体验中的网页性能表现。

::script-stats  
::

::script-docs  
::

该组合式函数默认包含以下设置：  
- **触发时机：客户端**  脚本将在 Nuxt 进行水合（hydration）时加载，以保持网页关键性能指标的准确性。

::script-types  
::

## 在 app.vue 中加载

通过 `app.vue` 在 Nuxt 准备就绪时加载 Cloudflare 网站分析。

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

Cloudflare 网站分析组合式函数会向全局作用域注入一个 `window.__cfBeacon` 对象。如果需要访问它，你可以通过等待脚本加载完成后获取。

```ts
const { onLoaded } = useScriptCloudflareWebAnalytics()
onLoaded(({ cfBeacon }) => {
  console.log(cfBeacon)
})
```
