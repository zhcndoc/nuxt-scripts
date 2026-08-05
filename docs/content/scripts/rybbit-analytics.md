---
title: Rybbit Analytics
description: 加载托管或自托管的 Rybbit Analytics，并调用其跟踪 API。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/rybbit-analytics.ts
    size: xs
---

[Rybbit Analytics](https://rybbit.com/) 是一个开源网站分析平台。其[跟踪脚本文档](https://rybbit.com/docs/script)介绍了托管和自托管脚本的 URL。

::script-stats  
::

::script-docs  
::

### 自托管的 Rybbit Analytics

Rybbit 官方的自托管代码片段会将 `/api/script.js` 中的 `app.rybbit.io` 替换掉。当前的组合式函数会将 `/script.js` 追加到 `analyticsHost`，因此请在该值中包含 `/api`：

```ts
useScriptRybbitAnalytics({
  analyticsHost: 'https://your-rybbit-instance.com/api',
  siteId: 'YOUR_SITE_ID'
})
```

目前仅传入源地址会请求 `https://your-rybbit-instance.com/script.js`，这与 Rybbit 文档中说明的路径不同。

::callout{color="amber"}
Rybbit 现在会在站点控制面板中配置自动页面浏览、SPA 导航、出站链接、错误、会话回放和 Web Vitals。当前的封装仍会为这些字段生成旧版的 `data-*` 属性，而当前的跟踪器文档并未列出这些属性。此外，当 `debounce` 为 `0` 时，它还会省略 `data-debounce`，尽管 Rybbit 文档说明将其设为零是禁用防抖的方法。
::

::script-types
::
