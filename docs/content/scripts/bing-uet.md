---
title: Bing UET
description: 加载 Microsoft Advertising UET，发送转化事件，并更新其广告存储同意信号。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/bing-uet.ts
  size: xs
---

[Microsoft Advertising UET](https://about.ads.microsoft.com/en/tools/performance/conversion-tracking) 可跟踪转化，并向 Microsoft Advertising 提供受众数据。

[`useScriptBingUet()`{lang="ts"}](/scripts/bing-uet) 加载 UET 标签并公开其事件队列。

::script-stats
::

::script-docs
::

::script-types
::

## 示例

### 跟踪转化

Microsoft 建议使用较新的事件语法，该语法用 [`revenue_value` 和 `currency`](https://learn.microsoft.com/en-us/advertising/msa-help/hlp_ba_conc_uetv2syntaxupdate_2) 替代 `gv` 和 `gc` 等简短参数。

```vue
<script setup lang="ts">
const { proxy } = useScriptBingUet()

function trackPurchase() {
  proxy.uetq.push('event', 'purchase', {
    revenue_value: 49.99,
    currency: 'USD',
    transaction_id: 'ORDER-123',
  })
}
</script>
```

### 自定义事件

```vue
<script setup lang="ts">
const { proxy } = useScriptBingUet()

function trackSignup() {
  proxy.uetq.push('event', 'sign_up', {
    event_category: 'engagement',
    event_label: 'newsletter',
  })
}
</script>
```

### 同意模式

Bing UET 支持[高级同意模式](https://learn.microsoft.com/en-us/advertising/msa-help/hlp_ba_conc_uet_consent)。仅支持 `ad_storage`；使用 `defaultConsent` 设置初始状态，并在运行时通过 `consent.update()`{lang="ts"} 更新：

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

Microsoft 要求使用同意模式的网站在每个页面上发送 `granted` 或 `denied`。目前，其强制执行范围涵盖欧洲经济区、英国和瑞士。

`onBeforeUetStart` 仍可用于任何其他加载前设置。
