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
