---
title: Databuddy 分析
description: 加载 Databuddy 的浏览器 SDK，并配置其分析和性能跟踪功能。
links:
  - label: 源代码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/databuddy-analytics.ts
    size: xs
---

[Databuddy](https://www.databuddy.cc/docs/getting-started) 通过浏览器 SDK 提供网站分析、错误跟踪和性能测量功能。

::script-stats
::

::script-docs
::

## CDN 或自行托管

默认情况下，注册表会从 `https://cdn.databuddy.cc/databuddy.js` 注入[文档中的浏览器 SDK](https://www.databuddy.cc/docs/sdk/vanilla-js)。如果你自行托管脚本，请传入 `scriptUrl` 以覆盖 `src`。

```ts
useScriptDatabuddyAnalytics({
  scriptUrl: 'https://my-host/databuddy.js',
  clientId: 'YOUR_CLIENT_ID',
})
```

::callout{color="amber"}
第一方打包会重写脚本 URL，因此当前集成还会创建 `window.databuddyConfig`。该回退配置包含 `clientId`，以及 `apiUrl`、`disabled`、`trackScreenViews`、`trackPerformance` 和 `trackSessions` 的真值。其他选项仍仅作为 `data-*` 属性保留，并且这三个跟踪字段中显式设置为 `false` 的值不会复制到回退配置中。如果打包后的 SDK 未应用你所需的设置，请禁用打包。
::

::script-types
::
