---
title: Bing UET
description: 在您的 Nuxt 应用中使用 Microsoft Advertising Universal Event Tracking。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/bing-uet.ts
  size: xs
---

[Microsoft Advertising UET](https://about.ads.microsoft.com/en-us/solutions/tools/universal-event-tracking)（通用事件跟踪）让您能够跟踪转化、构建再营销列表并优化您的 Microsoft Advertising 广告系列。

Nuxt Scripts 提供了一个注册表脚本组合式函数 [`useScriptBingUet()`{lang="ts"}](/scripts/bing-uet)，以便在您的 Nuxt 应用中轻松集成 Bing UET。

::script-stats
::

::script-docs
::

::script-types
::

## 示例

### 跟踪转化

```vue
<script setup lang="ts">
const { proxy } = useScriptBingUet()

function trackPurchase() {
  proxy.uetq.push({
    ec: 'purchase',
    ev: 49.99,
    gc: 'USD',
  })
}
</script>
```

### 自定义事件

```vue
<script setup lang="ts">
const { proxy } = useScriptBingUet()

function trackSignup() {
  proxy.uetq.push({
    ec: 'sign_up',
    el: 'newsletter',
    ea: 'engagement',
  })
}
</script>
```

### Consent Mode

Bing UET 支持 [高级同意模式](https://help.ads.microsoft.com/#apex/ads/en/60119/1-500)。仅会遵循 `ad_storage`；使用 `defaultConsent` 设置初始状态，并通过运行时的 `consent.update()`{lang="ts"} 进行更新：

```vue
<script setup lang="ts">
const { consent } = useScriptBingUet({
  defaultConsent: { ad_storage: 'denied' },
})

function grantConsent() {
  consent.update({ ad_storage: 'granted' })
}
</script>
```

`onBeforeUetStart` 仍可用于任何其他预加载设置。
