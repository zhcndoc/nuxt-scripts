---
title: Meta Pixel
description: 加载 Meta Pixel，通过 fbq 发送事件，并管理其二元同意状态。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/meta-pixel.ts
  size: xs
---

[Meta Pixel](https://www.facebook.com/business/tools/meta-pixel) 向 Meta Ads 发送转化和受众事件。

[`useScriptMetaPixel()`{lang="ts"}](/scripts/meta-pixel){lang="ts"} 加载像素代码并公开 `fbq` 队列。

::script-stats
::

::script-docs
::

::script-types
::

## 同意模式

Meta Pixel 提供了一个二元同意开关。使用 `defaultConsent` 设置初始状态（会在 `fbq('init', id)`{lang="ts"} 之前触发 `fbq('consent', 'grant'|'revoke')`{lang="ts"}），并在运行时调用 `consent.grant()`{lang="ts"} / `consent.revoke()`{lang="ts"}：

```vue
<script setup lang="ts">
const { consent } = useScriptMetaPixel({
  id: 'YOUR_PIXEL_ID',
  defaultConsent: 'denied',
})

function acceptAds() {
  consent.grant()
}
function rejectAds() {
  consent.revoke()
}
</script>
```

`defaultConsent: 'denied'` 会在像素初始化之前将撤销命令加入队列，但不会延迟 SDK 请求。如果您的同意政策要求在用户选择加入之前不得向 Meta 发起请求，请为脚本本身使用[二元加载门](/docs/guides/consent#binary-load-gate)。
