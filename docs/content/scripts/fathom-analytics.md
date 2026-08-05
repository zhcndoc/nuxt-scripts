---
title: Fathom Analytics
description: 加载 Fathom Analytics 并跟踪页面浏览量、目标和自定义事件。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/fathom-analytics.ts
    size: xs
---

[Fathom Analytics](https://usefathom.com/) 在不收集访客个人数据的情况下跟踪网站流量。

::script-stats  
::

::script-docs  
::

## 不支持代理

与 Nuxt Scripts 中的大多数分析集成不同，Fathom **不能** 被代理（`proxy: true`）。

Fathom 的机器人检测使用发起连接的源 IP 地址。当 beacon 被代理时，它们会从你的服务器 IP（通常是数据中心 IP）到达 Fathom，而 Fathom 的机器人检测会忽略来自任意服务器的 `X-Forwarded-For`，因此每位访客都会被标记为机器人。此行为以及由此产生的 Nuxt Scripts 更改已记录在[代理支持修复](https://github.com/nuxt/scripts/pull/722)中。

Fathom 于 [2023 年 3 月停止提供自定义域名](https://usefathom.com/changelog/mar2023-firewall-settings)。现有的自定义域名仍可继续使用，但新客户无法配置自定义域名。

打包（`bundle: true`）**是**支持的：脚本会从你的源站提供，但 beacon 仍会直接从浏览器发送到 `cdn.usefathom.com`，因此真实的客户端 IP 能正确地到达 Fathom 的机器人检测系统。

## 默认值

- **触发器：`onNuxtReady`** Nuxt 应用准备就绪时加载脚本。

对于无返回值的调用，请使用组合式函数的 `proxy` 对象。需要加载的 `fathom` 对象时，请使用 `onLoaded`。

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

默认触发器会等待 Nuxt 准备就绪：

```vue [app.vue]
<script setup lang="ts">
useScriptFathomAnalytics({
  site: 'YOUR_SITE_ID',
})
</script>
```
