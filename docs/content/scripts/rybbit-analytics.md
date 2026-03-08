---

title: Rybbit Analytics  
description: 在你的 Nuxt 应用中使用 Rybbit Analytics。  
links:  
  - label: 源代码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/rybbit.ts  
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
  scriptInput: {
    src: 'https://your-rybbit-instance.com/api/script.js'
  },
  siteId: 'YOUR_SITE_ID'
})
```

::script-types  
::

#### 配置选项

- `siteId`（必填）：你的 Rybbit Analytics 站点 ID  
- `autoTrackPageview`：设置为 `false` 可禁用脚本加载时自动跟踪初始页面访问。你需要手动调用页面访问函数进行跟踪。默认值：`true`  
- `trackSpa`：设置为 `false` 可禁用单页应用的自动页面访问跟踪  
- `trackQuery`：设置为 `false` 可禁用 URL 查询字符串的跟踪  
- `trackOutbound`：设置为 `false` 可禁用出站链接点击的自动跟踪。默认值：`true`  
- `trackErrors`：设置为 `true` 启用 JavaScript 错误和未处理 Promise 拒绝的自动跟踪。仅跟踪相同来源的错误，以避免来自第三方脚本的噪音。默认值：`false`  
- `sessionReplay`：设置为 `true` 启用会话回放记录。捕获用户交互、鼠标移动和 DOM 变更，方便调试和用户体验分析。默认值：`false`  
- `webVitals`：设置为 `true` 启用 Web Vitals 性能指标收集（LCP、CLS、INP、FCP、TTFB）。Nuxt 默认禁用 Web Vitals，以减少脚本体积和网络请求。默认值：`false`  
- `skipPatterns`：要忽略的 URL 路径模式数组  
- `maskPatterns`：要为了隐私进行遮蔽的 URL 路径模式数组  
- `debounce`：URL 变化后延迟多少毫秒才跟踪页面访问  
- `apiKey`：开发过程中从 localhost 跟踪时使用的 API 密钥。绕过自托管 Rybbit Analytics 实例的来源验证
