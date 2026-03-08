---

title: Plausible Analytics  
description: 在您的 Nuxt 应用中使用 Plausible Analytics。  
links:  
  - label: 源代码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/plausible-analytics.ts  
    size: xs  

---

[Plausible Analytics](https://plausible.io/) 是一个注重隐私的 Nuxt 应用分析解决方案，允许您在不侵犯用户隐私的前提下跟踪网站流量。

::script-stats  
::  

::script-docs  
::  

### 自托管版 Plausible

如果您使用自托管版 Plausible，请为脚本提供一个明确的 src 地址，以便浏览器将 API 事件发送到正确的端点。

```ts
useScriptPlausibleAnalytics({
  scriptInput: {
    src: 'https://my-self-hosted-plausible.io/js/script.js'
  }
})
```

::script-types  
::  

**注意：** 您可以在 Plausible 仪表盘的站点设置中 **Site Installation** 区域找到 `scriptId`。

**提取您的 Script ID：**

Plausible 会为您提供如下的脚本标签：

```html
<script async src="https://plausible.io/js/pa-gYyxvZhkMzdzXBAtSeSNz.js"></script>
```

您的 `scriptId` 是 `pa-` 之后、`.js` 之前的部分：

```ts
scriptId: 'gYyxvZhkMzdzXBAtSeSNz'
//         ^^^^^^^^^^^^^^^^^^^^^^^
//         从 pa-{scriptId}.js 中提取
```
