---
title: Clarity
description: 在你的 Nuxt 应用中使用 Clarity。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/clarity.ts
    size: xs
---

[Clarity](https://clarity.microsoft.com/) 是微软推出的屏幕录制和热力图工具，帮助你了解用户如何与网站交互。

::script-stats
::

::script-docs
::

::script-types
::

## 同意模式

Clarity 支持 cookie 同意开关（布尔值）或高级同意向量（记录）。使用 `defaultConsent` 设置初始值，并在运行时调用 `consent.set()`{lang="ts"}：

```vue
<script setup lang="ts">
const { consent } = useScriptClarity({
  id: 'YOUR_PROJECT_ID',
  defaultConsent: false, // 在用户选择加入之前禁用 cookie
})

function acceptAnalytics() {
  consent.set(true)
}
</script>
```

`consent.set()`{lang="ts"} 也接受 Clarity 的高级同意向量，以便对 cookie 类别进行更细粒度的控制：

```ts
const { consent } = useScriptClarity({
  id: 'YOUR_PROJECT_ID',
  defaultConsent: {
    ad_storage: 'denied',
    analytics_storage: 'granted',
  },
})

consent.set({
  ad_storage: 'granted',
  analytics_storage: 'granted',
})
```

有关详细信息，请参见 [Clarity cookie 同意](https://learn.microsoft.com/en-us/clarity/setup-and-installation/cookie-consent)。
