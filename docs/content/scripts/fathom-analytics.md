---
title: Fathom Analytics
description: 在你的 Nuxt 应用中使用 Fathom Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/fathom-analytics.ts
    size: xs
---

[Fathom Analytics](https://usefathom.com/) 是一个非常优秀的隐私分析解决方案，适用于你的 Nuxt 应用。它不会收集访客的个人数据，但能提供关于访客如何使用你的网站的详细洞察。

::script-stats  
::

::script-docs  
::

## 默认值

- **触发时机**：脚本将在 Nuxt 完成水合（hydrated）时加载。

你可以直接将 `fathom` 对象作为代理（proxy）访问，或者等待 `$script` promise 来访问该对象。建议
对任何无返回值的函数使用代理。

::code-group

```ts [Proxy]
const { proxy } = useScriptFathomAnalytics()
function trackMyGoal() {
  proxy.trackGoal('MY_GOAL_ID', 100)
}
```

```ts [onLoaded]
const { onLoaded } = useScriptFathomAnalytics()
onLoaded(({ trackGoal }) => {
  trackGoal('MY_GOAL_ID', 100)
})
```

::

::script-types
::

## 示例

当 Nuxt 准备就绪时，通过 `app.vue` 加载 Fathom Analytics。

```vue [app.vue]
<script setup lang="ts">
useScriptFathomAnalytics({
  site: 'YOUR_SITE_ID',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```
