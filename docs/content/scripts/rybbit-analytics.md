---
title: Rybbit Analytics
description: 在你的 Nuxt 应用中使用 Rybbit Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/rybbit-analytics.ts
    size: xs
---

[Rybbit Analytics](https://www.rybbit.io/) 是一个注重隐私的分析解决方案，用于跟踪你网站上的用户活动，同时不会影响用户隐私。

::script-stats  
::

::script-docs  
::

### 自托管的 Rybbit Analytics

如果你使用的是自托管版本的 Rybbit Analytics，可以提供自定义的脚本源：

```ts
useScriptRybbitAnalytics({
  analyticsHost: 'https://your-rybbit-instance.com',
  siteId: 'YOUR_SITE_ID'
})
```

::script-types  
::
