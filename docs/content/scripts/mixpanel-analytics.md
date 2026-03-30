---
title: Mixpanel
description: 在你的 Nuxt 应用中使用 Mixpanel。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/mixpanel-analytics.ts
  size: xs
---

[Mixpanel](https://mixpanel.com) 是一个产品分析平台，通过事件追踪、漏斗分析和留存分析来帮助你了解用户如何与应用交互。

Nuxt Scripts 提供了一个注册表脚本组合式函数 [`useScriptMixpanelAnalytics()`{lang="ts"}](/scripts/mixpanel-analytics)，用于在你的 Nuxt 应用中轻松集成 Mixpanel。

::script-stats
::

::script-docs
::

::script-types
::

## 示例

### 追踪事件

```vue
<script setup lang="ts">
const { proxy } = useScriptMixpanelAnalytics()

function trackSignup() {
  proxy.mixpanel.track('Sign Up', {
    plan: 'premium',
    source: 'landing_page',
  })
}
</script>
```

### 识别用户

```vue
<script setup lang="ts">
const { proxy } = useScriptMixpanelAnalytics()

function login(userId: string) {
  proxy.mixpanel.identify(userId)
  proxy.mixpanel.people.set({
    $name: 'Jane Doe',
    $email: 'jane@example.com',
    plan: 'premium',
  })
}
</script>
```

### 注册超级属性

Mixpanel 会将超级属性随每个后续事件一起发送：

```vue
<script setup lang="ts">
const { proxy } = useScriptMixpanelAnalytics()

proxy.mixpanel.register({
  app_version: '2.0.0',
  platform: 'web',
})
</script>
```
