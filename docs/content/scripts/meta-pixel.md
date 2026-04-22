---
title: Meta Pixel
description: 在你的 Nuxt 应用中使用 Meta Pixel。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/meta-pixel.ts
  size: xs
---

[Meta Pixel](https://www.facebook.com/business/tools/meta-pixel) 让你能够衡量、优化并为你的 Facebook 广告活动建立受众。

Nuxt Scripts 提供了注册脚本组合函数 [`useScriptMetaPixel()`{lang="ts"}](/scripts/meta-pixel){lang="ts"}，方便你在 Nuxt 应用中集成 Meta Pixel。

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

详情请参阅 [Meta 的同意文档](https://www.facebook.com/business/help/1151321516677370)。
