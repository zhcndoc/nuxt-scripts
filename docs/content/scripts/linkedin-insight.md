---
title: LinkedIn Insight 标签
description: 在你的 Nuxt 应用中使用 LinkedIn Insight Tag 跟踪转化、重定向访问者并了解你的受众。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/linkedin-insight.ts
  size: xs
---

[LinkedIn Insight Tag](https://business.linkedin.com/marketing-solutions/insight-tag) 是一个轻量级的 JavaScript 片段，用于在 LinkedIn Ads 活动中进行转化跟踪、重定向以及受众洞察。

Nuxt Scripts 提供了一个注册表脚本组合式函数 [`useScriptLinkedInInsight()`{lang="ts"}](/scripts/linkedin-insight)，用于将其集成到你的 Nuxt 应用中。

::script-stats
::

::script-docs
::

::script-types
::

## 示例

### 跟踪一次转化

```vue
<script setup lang="ts">
const { proxy } = useScriptLinkedInInsight({
  id: '111143',
})

function trackPurchase() {
  proxy.lintrk('track', { conversion_id: 1111111177 })
}
</script>
```

### 使用 Conversions API 按事件去重

当你也通过 LinkedIn 的服务端 Conversions API 发送转化时，请将相同的 `event_id` 传给两端。LinkedIn 会丢弃服务端的重复事件，并统计 Insight Tag 事件。参见 [LinkedIn 去重](https://learn.microsoft.com/en-us/linkedin/marketing/conversions/deduplication?view=li-lms-2026-01)。

```vue
<script setup lang="ts">
const { proxy } = useScriptLinkedInInsight({
  id: '111143',
})

function trackSignup() {
  const eventId = crypto.randomUUID()
  proxy.lintrk('track', { conversion_id: 1111111177, event_id: eventId })
  // 将相同的 eventId 发送到你的服务端 Conversions API 调用。
}
</script>
```

### 页面加载转化去重

对于自动触发的页面浏览去重，请在注册时设置 `eventId`。这个组合式函数会在 Insight Tag 基础代码运行 *之前* 赋值 `window._linkedin_event_id`，因此页面浏览 URL 会自动包含 `&eventId=…`。

```vue
<script setup lang="ts">
const { proxy } = useScriptLinkedInInsight({
  id: '111143',
  eventId: 'page-load-event-id-123',
})
</script>
```

### 使用 `setUserData` 进行增强匹配

传入纯文本邮箱；Insight Tag 会在设备端对其进行 SHA-256 哈希。参见 [LinkedIn 增强匹配](https://www.linkedin.com/help/lms/answer/a6246095)。

```vue
<script setup lang="ts">
const { proxy } = useScriptLinkedInInsight({
  id: '111143',
})

function onSignupSuccess(email: string) {
  proxy.lintrk('setUserData', { email })
}
</script>
```

Insight Tag 会将哈希后的邮箱存储在 `localStorage["li_hem"]` 中，并在下一次页面浏览时通过向 `https://px.ads.linkedin.com/wa/...`（WebsiteActions 网关）发起单独的 POST 请求来传输它。紧随 `setUserData` 之后的页面浏览的 `/collect` URL 不会携带它；LinkedIn 会在下一次完整页面加载时，通过引导脚本从 `localStorage` 中读回它来获取。

### SPA 虚拟页面浏览

默认情况下，Insight Tag 在脚本加载时只会触发一次页面浏览，因此 SPA 路由变化不会被跟踪。可以通过 `enableAutoSpaTracking` 开启按路由跟踪：

```vue
<script setup lang="ts">
useScriptLinkedInInsight({
  id: '111143',
  enableAutoSpaTracking: true,
})
</script>
```

启用后，该组合式函数会抑制脚本内置的自动页面浏览（通过 `window._wait_for_lintrk = true`），并在 Nuxt 的 `page:finish` 钩子上触发 `lintrk('track')`{lang="ts"}，这样每个路由（包括初始页面）都会恰好生成一次 `/collect` 信标。

### 多个 Partner ID

如果你需要向 `window._linkedin_data_partner_ids` 推送多个 Partner ID，请传入数组。该组合式函数会将第一个 ID 提升为主 `_linkedin_partner_id` 全局变量。

```vue
<script setup lang="ts">
useScriptLinkedInInsight({
  id: ['111143', '111154'],
})
</script>
```
