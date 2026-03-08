---

title: Vercel 分析
description: 在你的 Nuxt 应用中使用 Vercel 分析。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/vercel-analytics.ts
    size: xs

---

[Vercel 分析](https://vercel.com/docs/analytics) 为你的 Nuxt 应用提供轻量级且注重隐私的网页分析。在部署于 [Vercel](https://vercel.com) 时，它能够零配置地追踪页面浏览和自定义事件。

::script-stats
::

::script-docs
::

### 非 Vercel 部署

当在 Vercel 之外部署时，需要显式提供你的 DSN：

```ts
useScriptVercelAnalytics({
  dsn: 'YOUR_DSN',
})
```

### 第一方模式

当你启用 `scripts.firstParty`，Nuxt 会将分析脚本打包到本地，并通过你的服务器代理数据收集请求。这可以防止广告拦截器阻止分析，且避免将敏感数据发送到第三方请求。

```ts
export default defineNuxtConfig({
  scripts: {
    firstParty: true,
    registry: {
      vercelAnalytics: true,
    }
  }
})
```

## 默认设置

- **触发时机：客户端** 脚本将在 Nuxt 进行 hydration 时加载，以保持网页关键性能指标的准确性。

::script-types
::

你可以直接通过代理访问 `track` 和 `pageview` 方法，或者等待 `$script` Promise 来获取对象。建议对于无返回值的函数使用代理方式。

::code-group

```ts [Proxy]
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

## 示例

当 Nuxt 准备好时通过 `app.vue` 加载 Vercel 分析。

```vue [app.vue]
<script setup lang="ts">
const { proxy } = useScriptVercelAnalytics({
  scriptOptions: {
    trigger: 'onNuxtReady',
  },
})

// 追踪自定义事件
proxy.track('signup', { plan: 'pro' })
</script>
```

### 手动追踪

如果你想完全控制追踪内容，可以禁用自动追踪，并手动调用 `track` / `pageview`。

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

使用 `beforeSend` 在事件发送到 Vercel 前过滤或修改事件。返回 `null` 可取消事件。

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
