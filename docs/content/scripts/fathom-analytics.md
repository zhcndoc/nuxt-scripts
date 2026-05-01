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

## 不支持代理

与 Nuxt Scripts 中的大多数分析集成不同，Fathom **不能** 被代理（`proxy: true`）。

Fathom 的机器人检测会使用连接来源的 IP 地址。当 beacon 通过代理转发时，它们会从你服务器的 IP（通常是数据中心）到达 Fathom，而 Fathom 的机器人检测会忽略来自任意服务器的 `X-Forwarded-For`，因此每个访客都会被标记为机器人。

Fathom 之前提供过官方的自定义域名功能（CNAME 指向他们的基础设施），用于第一方托管，但他们已在 [2023 年 5 月将其弃用](https://usefathom.com/changelog/mar2023-firewall-settings)，且没有替代方案。

打包（`bundle: true`）**是**支持的：脚本会从你的源站提供，但 beacon 仍会直接从浏览器发送到 `cdn.usefathom.com`，因此真实的客户端 IP 能正确地到达 Fathom 的机器人检测系统。

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
