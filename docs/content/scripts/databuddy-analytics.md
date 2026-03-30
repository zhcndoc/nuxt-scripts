---
title: Databuddy Analytics
description: 在你的 Nuxt 应用中使用 Databuddy Analytics。
links:
  - label: 源代码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/databuddy-analytics.ts
    size: xs
---

[Databuddy](https://www.databuddy.cc/) 是一个注重隐私的分析平台，专注于性能和最小化数据收集。

::script-stats
::

::script-docs
::

### CDN / 自行托管

默认情况下，注册表注入 `https://cdn.databuddy.cc/databuddy.js`。如果你自行托管脚本，可在选项中传入 `scriptUrl` 来覆盖该 `src`。

```ts
useScriptDatabuddyAnalytics({
  scriptUrl: 'https://my-host/databuddy.js',
  clientId: 'YOUR_CLIENT_ID'
})
```

::script-types
::
