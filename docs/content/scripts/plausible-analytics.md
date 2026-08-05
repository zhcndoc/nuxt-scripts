---
title: Plausible Analytics
description: 加载 Plausible 的特定站点跟踪器，包括自托管端点。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/plausible-analytics.ts
    size: xs
---

[Plausible Analytics](https://plausible.io/) 是一个注重隐私的 Web 分析平台。此注册表条目支持其当前的特定站点脚本。

::script-stats  
::  

::script-docs  
::  

### 自托管版 Plausible

如果您使用的是自托管版本的 Plausible，请同时提供脚本 URL 和事件端点。仅更改脚本 URL 不会更改传递给 Plausible 的端点。

```ts
useScriptPlausibleAnalytics({
  scriptId: 'YOUR_SCRIPT_ID',
  endpoint: 'https://my-self-hosted-plausible.io/api/event',
  scriptInput: {
    src: 'https://my-self-hosted-plausible.io/js/script.js'
  }
})
```

对于 Plausible Cloud，请在站点设置的 **Site Installation** 下找到 `scriptId`。Plausible 的[脚本更新指南](https://plausible.io/docs/script-update-guide)解释了特定站点 URL 和 `plausible.init()`{lang="ts"} 选项。

### 提取脚本 ID

当前的 Plausible 安装标签如下所示：

```html
<script async src="https://plausible.io/js/pa-gYyxvZhkMzdzXBAtSeSNz.js"></script>
```

您的 `scriptId` 是 `pa-` 之后、`.js` 之前的部分：

```ts
scriptId: 'gYyxvZhkMzdzXBAtSeSNz'
//         ^^^^^^^^^^^^^^^^^^^^^^^
//         从 pa-{scriptId}.js 中提取
```

::script-types
::
