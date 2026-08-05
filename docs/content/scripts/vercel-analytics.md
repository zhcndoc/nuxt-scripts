---
title: Vercel Analytics
description: 加载 Vercel Web Analytics 并发送页面浏览量或自定义事件。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/vercel-analytics.ts
    size: xs
---

[Vercel Web Analytics](https://vercel.com/docs/analytics) 会记录页面浏览量和自定义事件。其[隐私与合规](https://vercel.com/docs/analytics/privacy-policy)指南介绍了数据收集和访客识别模型。

::script-stats
::

::script-docs
::

### 非 Vercel 部署

在 [Vercel](https://vercel.com) 之外进行部署时，请显式提供 DSN。其[分析包参考](https://vercel.com/docs/analytics/package)文档介绍了此情况下的 `dsn` 选项：

```ts
useScriptVercelAnalytics({
  dsn: 'YOUR_DSN',
})
```

### 第一方模式

此注册表条目启用了第一方模式。Nuxt 会在本地打包脚本，并通过你的服务器代理数据接收请求。该代理会将客户端 IP 匿名化为子网，但不会对事件正文或 URL 进行脱敏处理。

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      vercelAnalytics: { trigger: 'client' },
    }
  }
})
```

## 默认设置

- **触发器：客户端** 脚本在 Nuxt hydration 期间加载，以确保 Web Vitals 指标的准确性。

注册表会将此触发器固定为 `client`。调用方传入的 `scriptOptions.trigger` 值会被覆盖。

构建环境会选择文件：开发构建使用 `script.debug.js`，而生产构建使用 `script.js`。`mode` 选项用于设置 Vercel 的 `window.vam` 运行时值；它不会改变文件选择。当前的 `debug` 映射仅会在开发环境中传递 `debug: false`，因此在生产构建中设置 `debug: true` 不会启用调试脚本。

通过代理调用返回 void 的 `track` 和 `pageview` 方法。你也可以等待 `$script`，以访问已加载的对象。

::code-group

```ts [代理]
const { proxy } = useScriptVercelAnalytics()
proxy.track('signup', { plan: 'pro' })
```

```ts [onLoaded]
const { onLoaded } = useScriptVercelAnalytics()
onLoaded(({ track }) => {
  track('signup', { plan: 'pro' })
})
```

::

::script-types
::

## 示例

通过 `app.vue` 加载 Vercel Analytics：

```vue [app.vue]
<script setup lang="ts">
const { proxy } = useScriptVercelAnalytics()

// 追踪自定义事件
proxy.track('signup', { plan: 'pro' })
</script>
```

### 手动追踪

禁用自动追踪，并自行调用 `track` 或 `pageview`：

```vue [app.vue]
<script setup lang="ts">
const { proxy } = useScriptVercelAnalytics({
  disableAutoTrack: true,
})

// 追踪自定义事件
proxy.track('purchase', { product: 'widget', price: 9.99 })

// 手动页面浏览统计
proxy.pageview({ path: '/custom-page' })
</script>
```

### beforeSend

使用 `beforeSend` 在事件发送到 Vercel 之前对其进行筛选或修改。返回 `null` 可取消事件。Vercel 也建议在此处[编辑敏感的 URL 数据](https://vercel.com/docs/analytics/redacting-sensitive-data)。

```vue [app.vue]
<script setup lang="ts">
const { proxy } = useScriptVercelAnalytics({
  beforeSend(event) {
    // 忽略管理页面
    if (event.url.includes('/admin'))
      return null
    return event
  },
})
</script>
```
