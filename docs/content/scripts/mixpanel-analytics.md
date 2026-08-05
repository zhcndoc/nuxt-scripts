---
title: Mixpanel
description: 加载 Mixpanel 并跟踪产品事件、身份、用户画像和同意状态。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/mixpanel-analytics.ts
  size: xs
---

[Mixpanel](https://mixpanel.com) 通过漏斗和留存报告分析产品事件。

[`useScriptMixpanelAnalytics()`{lang="ts"}](/scripts/mixpanel-analytics) 初始化 SDK 并公开 `mixpanel` API。

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

Mixpanel 会将[注册的超级属性](https://docs.mixpanel.com/docs/tracking-methods/sdks/javascript#setting-super-properties)添加到后续事件中：

```vue
<script setup lang="ts">
const { proxy } = useScriptMixpanelAnalytics()

proxy.mixpanel.register({
  app_version: '2.0.0',
  platform: 'web',
})
</script>
```

### 注销时重置身份

当已识别的用户退出登录或其会话过期时，调用 `reset()`{lang="ts"}。Mixpanel [建议在注销时进行重置](https://docs.mixpanel.com/docs/tracking-methods/id-management/identifying-users-simplified#client-side-identity-management)，以避免共享同一设备的两个人被合并为同一个身份：

```ts
const { proxy } = useScriptMixpanelAnalytics()

function logout() {
  // 首先结束应用会话。
  proxy.mixpanel.reset()
}
```

## 同意模式

Mixpanel 提供 [`opt_in_tracking` / `opt_out_tracking`](https://docs.mixpanel.com/docs/tracking-methods/sdks/javascript#opt-out-of-tracking)。使用 `defaultConsent` 设置启动时的默认状态，并在运行时调用 `consent.optIn()`{lang="ts"} / `consent.optOut()`{lang="ts"}。

### `defaultConsent`

| 值 | 行为 |
|-------|-----------|
| `'opt-in'` | 以已同意状态启动。 |
| `'opt-out'` | 调用 `mixpanel.init(..., { opt_out_tracking_by_default: true })`{lang="ts"}，使 SDK 以未同意状态启动。 |

::callout{icon="i-heroicons-information-circle"}
`defaultConsent: 'opt-out'` 会在第一个事件之前生效。之后调用 `consent.optOut()`{lang="ts"} 无法撤回 SDK 已捕获的事件。
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
