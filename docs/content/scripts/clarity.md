---
title: Clarity
description: 加载 Microsoft Clarity 并管理其会话录制同意状态。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/clarity.ts
    size: xs
---

[Microsoft Clarity](https://learn.microsoft.com/en-us/clarity/setup-and-installation/about-clarity) 会记录您网站上的会话并生成热图。

::script-stats
::

::script-docs
::

::script-types
::

## 同意模式

`defaultConsent` 选项和 `consent.set()`{lang="ts"} 封装了 Clarity 已弃用的 [Consent API V1](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-consent-api-v1)。Microsoft 使用布尔值记录该 API（或者不传值以授予同意）。尽管当前 Nuxt Scripts 架构也接受 `Record<string, string>`{lang="html"}，但 Clarity 并未为 V1 记录对象负载。

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

对于新的集成，请通过脚本代理调用 Consent API V2，并使用 Clarity 区分大小写的类别名称：

```ts
const { proxy } = useScriptClarity({
  id: 'YOUR_PROJECT_ID',
})

proxy.clarity('consentv2', {
  ad_Storage: 'denied',
  analytics_Storage: 'denied',
})

// 用户授予同意后更新。
proxy.clarity('consentv2', {
  ad_Storage: 'granted',
  analytics_Storage: 'granted',
})
```

`defaultConsent` 和 `consent.set()`{lang="ts"} 不会发出 `consentv2` 命令。有关当前 API，请参阅 [Clarity Consent API V2](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-consent-api-v2)。
