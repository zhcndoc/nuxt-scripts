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

## 同意模式

Mixpanel 提供了 [`opt_in_tracking` / `opt_out_tracking`](https://docs.mixpanel.com/docs/privacy/opt-out-of-tracking)。使用 `defaultConsent` 设置启动时默认状态，并在运行时调用 `consent.optIn()`{lang="ts"} / `consent.optOut()`{lang="ts"}。

### `defaultConsent`

| 值 | 行为 |
|-------|-----------|
| `'opt-in'` | 以已同意状态启动。 |
| `'opt-out'` | 调用 `mixpanel.init(..., { opt_out_tracking_by_default: true })`{lang="ts"}，使 SDK 以未同意状态启动。 |

::callout{icon="i-heroicons-information-circle"}
当你需要 SDK 以未同意状态启动时，请使用 `defaultConsent: 'opt-out'`。运行时的 `consent.optOut()`{lang="ts"} 会在初始化后调用 `opt_out_tracking()`{lang="ts"}，其效力弱于启动时标志；在初始化与退出同意调用之间捕获的任何事件仍会被发送。
::

### 示例

```vue
<script setup lang="ts">
const { consent } = useScriptMixpanelAnalytics({
  token: 'YOUR_TOKEN',
  defaultConsent: 'opt-out',
})

function onAccept() {
  consent.optIn()
}
function onRevoke() {
  consent.optOut()
}
</script>
```
