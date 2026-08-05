# 第一方模式 —— 架构参考

## 架构

第一方路由的两种不同机制：

### 路径 A：打包 + 重写 + 拦截（大多数脚本）
1. **构建**：下载脚本，通过 AST 使用从 `proxy-configs.ts` 的 `domains[]` 派生的域名到代理的映射重写硬编码 URL，应用 SDK 特定的 `postProcess` 补丁
2. **客户端**：拦截插件通过 `__nuxtScripts` 包装 `fetch`/`sendBeacon`/`XHR`/`Image.src` —— 任何非同源 URL 都会自动通过 `/_scripts/p/<host><path>` 进行代理
3. **服务端**：在 `/_scripts/p/**` 的 Nitro 处理程序从路径中提取域名，重构上游 URL，并带隐私转换进行代理

### 路径 B：配置注入 + 代理（仅 PostHog）
1. **构建**：PostHog 代理配置上的 `autoInject` 设置 `apiHost` → `/_scripts/p/us.i.posthog.com`
2. **客户端**：SDK 原生使用注入的端点 —— 无需拦截
3. **服务端**：相同的 Nitro 代理处理程序

PostHog 是唯一真正的路径 B 脚本 —— 它使用 npm 模式（`src: false`，没有要下载/重写的脚本）。

### 运行时拦截的工作原理
AST 重写器转换下载的第三方脚本中的 API 调用：
- `fetch(url)` → `__nuxtScripts.fetch(url)`
- `navigator.sendBeacon(url)` → `__nuxtScripts.sendBeacon(url)`
- `new XMLHttpRequest` → `new __nuxtScripts.XMLHttpRequest`
- `new Image` → `new __nuxtScripts.Image`

拦截插件使用一个简单的 `proxyUrl` 函数定义 `__nuxtScripts`：
```js
function proxyUrl(url) {
  const parsed = new URL(url, location.origin)
  if (parsed.origin !== location.origin)
    return `${location.origin + proxyPrefix}/${parsed.host}${parsed.pathname}${parsed.search}`
  return url
}
```

无需域名白名单或规则匹配。只有经过 AST 重写的第三方脚本才会调用 `__nuxtScripts`，因此任何非同源 URL 都可以安全地代理。

服务端处理程序从路径中提取域名（例如 `/_scripts/p/www.google-analytics.com/g/collect` → `https://www.google-analytics.com/g/collect`）并查找每个域名的隐私配置。

### 自动注入（路径 A 的补充）
一些路径 A 脚本也在其代理配置中定义了 `autoInject` 来设置 SDK 端点配置：
- **Plausible**：`endpoint` → `/_scripts/p/plausible.io/api/event`
- **Umami**：`hostUrl` → `/_scripts/p/cloud.umami.is`
- **Rybbit**：`analyticsHost` → `/_scripts/p/app.rybbit.io/api`
- **Databuddy**：`apiUrl` → `/_scripts/p/basket.databuddy.cc`

自动注入遵守每个脚本的 `proxy: false` 退出选项（见下方的"每个脚本的退出选项"）。

### SDK 特定的后处理（ProxyConfig 上的 `postProcess`）
一些 SDK 有特性需要在 AST 重写后进行有针对性的正则表达式补丁。这些在每个脚本的 `ProxyConfig` 上直接定义为 `postProcess` 函数：
- **Rybbit**：SDK 从 `document.currentScript.src.split("/script.js")[0]` 派生 API 主机 —— 当打包到 `/_scripts/assets/<hash>.js` 时会中断。正则表达式用代理路径替换拆分表达式。
- **Fathom**：SDK 检查 `src.indexOf("cdn.usefathom.com") < 0` 以检测自托管模式并覆盖跟踪器 URL。正则表达式使此检查失效。

注意：Google Analytics 以前需要 `postProcess` 正则表达式补丁来处理动态构建的 collect URL。现在不再需要，因为运行时拦截插件会在 `sendBeacon`/`fetch` 调用点捕获所有非同源 URL。

## 路径别名（`proxy.alias`）

默认情况下，代理路径会嵌入第三方主机名的原文（`/_scripts/p/us.i.posthog.com/e/`），这会泄露自托管/内部域名，并且很容易被广告拦截器识别。`scripts.proxy.alias` 会使用别名替换主机名部分：
- `true` — 为每个域名自动生成一个简短的确定性哈希值（`sha256(domain).slice(0,8)`）
- `Record<domain, alias>` — 使用显式别名；未列出的域名保持原文

纯逻辑位于 `proxy-alias.ts`（`aliasForDomain`、`buildDomainAliasMap`、`invertAliasMap`、`aliasProxyValue`）中。该模块会根据所有代理域名（即 `domainPrivacy` 中的域名）构建 `domain → alias` 映射，并将其传递到所有生成代理路径的位置：
- **构建时重写**（`transform.ts`）：`to: ${proxyPrefix}/${alias ?? domain}`
- **自动注入**（`applyAutoInject`）：`aliasProxyValue` 会重写计算所得端点的主机部分
- **运行时拦截**（`intercept.ts`）：嵌入别名映射；`proxyUrl` 将 `parsed.host → alias`
- **Partytown**（`generatePartytownResolveUrl`）：嵌入别名映射；工作线程请求将 `url.host → alias`
- **服务器处理程序**（`proxy-handler.ts`）：反向的 `aliasToDomain` 映射会在允许列表匹配和转发之前，将别名部分解析回真实域名（原文主机名仍可正常解析，因此使用别名不会造成破坏性影响）
- **开发者工具**（`useScript.ts` 网络匹配器）：`aliasToDomain` 会暴露在开发者工具配置中，以便带别名的请求仍能归因到其脚本

通配符域名（`*`）永远不会使用别名——它们没有可供重写的字面路径形式，仅用于运行时允许列表匹配。

## 键映射

代理配置键与注册表键直接匹配 —— 没有间接层。脚本的 `registryKey` 用于从 `proxy-configs.ts` 查找其代理配置。

唯一的例外是 `googleAdsense`，它设置 `proxy: 'googleAnalytics'` 以共享 GA 的代理配置。

## 每个脚本的退出选项

脚本可以在三个级别上退出第一方模式：

### 注册表级别（无 `proxy` 能力）
在 `registry.ts` 中没有 `proxy` 能力的脚本永远不会被代理。用于核心功能需要指纹识别的脚本：
- **Stripe**、**PayPal**：欺诈检测需要真实的客户端 IP 和浏览器指纹
- **Google reCAPTCHA**：机器人检测需要真实的指纹
- **Google Sign-In**：认证完整性需要直接连接

这些脚本还设置了 `scriptBundling: false` 以防止 AST 重写。

### 配置级别（注册表配置中的 `proxy: false`）
用户可以在 `nuxt.config.ts` 中按脚本退出：

```
scripts.registry.plausibleAnalytics = { domain: 'mysite.com', proxy: false }
```

使用扁平配置语法：
```
scripts.registry.plausibleAnalytics = { domain: 'mysite.com', proxy: false }
```

这会跳过该脚本的域名注册、自动注入和 AST 重写。对于有 `autoInject` 的脚本（Plausible、PostHog、Umami、Rybbit、Databuddy）很重要，因为 `autoInject` 在转换之前的模块设置时运行。

### 组合式级别（`scriptOptions: { proxy: false }`）
```ts
useScriptPlausibleAnalytics({
  scriptOptions: { proxy: false }
})
```

这只影响 AST 重写（转换插件会跳过捆绑脚本的代理重写）。它**不会**撤销 `autoInject` 配置更改，因为这些在转换之前的模块设置时运行。对于有 `autoInject` 的脚本，请改用配置级退出选项。

## 添加新脚本

1. 在 `proxy-configs.ts` 中添加带有脚本域名和隐私预设的 `domains[]` 条目
2. 在 `registry.ts` 中添加带有匹配 `registryKey` 的注册表条目
3. 完成 —— 转换插件从域名派生重写规则为 `{ from: domain, to: proxyPrefix/domain }`

对于 npm 模式脚本（无需下载），定义 `autoInject` 来配置 SDK 的端点字段。

对于需要指纹识别的脚本（支付、验证码、认证），省略 `proxy` 能力并在注册表条目中设置 `scriptBundling: false`。

## 隐私预设

`proxy-configs.ts` 中的四个预设涵盖了所有启用代理的脚本：

| 预设 | 标志 | 使用者 |
|---|---|---|
| `PRIVACY_NONE` | 全部为 false | （当前未分配给任何脚本） |
| `PRIVACY_FULL` | 全部为 true | Meta、TikTok、X、Snap、Reddit、LinkedIn |
| `PRIVACY_HEATMAP` | ip、language、hardware | GA、Clarity、Hotjar |
| `PRIVACY_IP_ONLY` | 仅 ip | PostHog、Plausible、Umami、Rybbit、Databuddy、Ahrefs、Fathom、CF Web Analytics、Vercel、Matomo、Carbon Ads、Lemon Squeezy、Intercom、Gravatar、YouTube、Vimeo、Calendly |

注意：GTM、Segment、Crisp、Mixpanel、Bing UET 和 SpeedCurve 没有代理能力，因此不会应用隐私转换。

## 脚本支持

| 配置键 | 注册表脚本 | 隐私 | 机制 |
|---|---|---|---|
| `googleAnalytics` | googleAnalytics, **googleAdsense** | `PRIVACY_HEATMAP` | Path A |
| `metaPixel` | metaPixel | `PRIVACY_FULL` | Path A |
| `tiktokPixel` | tiktokPixel | `PRIVACY_FULL` | Path A |
| `xPixel` | xPixel | `PRIVACY_FULL` | Path A |
| `snapchatPixel` | snapchatPixel | `PRIVACY_FULL` | Path A |
| `redditPixel` | redditPixel | `PRIVACY_FULL` | Path A |
| `linkedinInsight` | linkedinInsight | `PRIVACY_FULL` | Path A |
| `ahrefsAnalytics` | ahrefsAnalytics | `PRIVACY_IP_ONLY` | Path A |
| `clarity` | clarity | `PRIVACY_HEATMAP` | Path A |
| `hotjar` | hotjar | `PRIVACY_HEATMAP` | Path A |
| `posthog` | posthog | `PRIVACY_IP_ONLY` | **Path B**（仅 npm）+ autoInject |
| `plausibleAnalytics` | plausibleAnalytics | `PRIVACY_IP_ONLY` | Path A + autoInject |
| `umamiAnalytics` | umamiAnalytics | `PRIVACY_IP_ONLY` | Path A + autoInject |
| `rybbitAnalytics` | rybbitAnalytics | `PRIVACY_IP_ONLY` | Path A + autoInject + postProcess |
| `databuddyAnalytics` | databuddyAnalytics | `PRIVACY_IP_ONLY` | Path A + autoInject |
| `fathomAnalytics` | fathomAnalytics | `PRIVACY_IP_ONLY` | Path A + postProcess |
| `cloudflareWebAnalytics` | cloudflareWebAnalytics | `PRIVACY_IP_ONLY` | Path A |
| `vercelAnalytics` | vercelAnalytics | `PRIVACY_IP_ONLY` | Path A |
| `matomoAnalytics` | matomoAnalytics | `PRIVACY_IP_ONLY` | Path A |
| `carbonAds` | carbonAds | `PRIVACY_IP_ONLY` | Path A |
| `lemonSqueezy` | lemonSqueezy | `PRIVACY_IP_ONLY` | Path A |
| `youtubePlayer` | youtubePlayer | `PRIVACY_IP_ONLY` | Path A |
| `vimeoPlayer` | vimeoPlayer | `PRIVACY_IP_ONLY` | Path A |
| `intercom` | intercom | `PRIVACY_IP_ONLY` | Path A |
| `gravatar` | gravatar | `PRIVACY_IP_ONLY` | Path A |
| `calendly` | calendly | `PRIVACY_IP_ONLY` | Path A |
| `googleTagManager` | googleTagManager | n/a | 仅打包 |
| `segment` | segment | n/a | 仅打包 |
| `crisp` | crisp | n/a | 仅打包 |
| `speedcurve` | speedcurve | n/a | 无代理（ID 参数化的 CDN URL） |

### 从第一方模式排除（`proxy: false`）

| 脚本 | 原因 |
|---|---|
| `stripe` | 欺诈检测需要真实的指纹 |
| `paypal` | 欺诈检测需要真实的指纹 |
| `googleRecaptcha` | 机器人检测需要真实的指纹 |
| `googleSignIn` | 认证完整性需要直接连接 |

## 设计说明

### 基于域名的代理路由
每个代理配置声明 `domains[]` —— 脚本与之通信的第三方域名列表。转换插件在构建时派生重写规则为 `{ from: domain, to: proxyPrefix/domain }`。服务端处理程序从代理路径中提取域名并转发到上游。

### 运行时拦截：无需规则
拦截插件代理任何非同源 URL。不需要域名白名单或规则匹配，因为 `__nuxtScripts` 包装器只会注入到经过 AST 重写的第三方脚本中。常规应用代码使用原生 `fetch`/`sendBeacon` 且不受影响。

服务端处理程序直接从代理路径（`/_scripts/p/<host>/<path>`）中提取目标域名并按域名查找隐私配置。无法识别的域名默认为完全匿名化（故障关闭）。

### 两阶段设置，配置一次性构建
`module.ts` 调用两个函数：
1. `setupFirstParty(config, resolvePath)` —— 无条件注册代理处理程序（处理程序在运行时拒绝未知域名）。返回 `FirstPartyConfig`。
2. 在 `modules:done` 中：通过 `resolveCapabilities()` 解析每个配置脚本的能力，然后调用 `finalizeFirstParty({...})`，它从注册表构建代理配置、收集域名隐私映射、应用自动注入、注册拦截插件并填充 runtimeConfig。遵守每个条目的 `proxy: false` 退出选项。

转换插件通过选项接收预构建的 `proxyConfigs` 映射 —— 按脚本直接查找，无需重新构建。

注册表条目规范化（`true`/`'mock'`/object/array → `[input, scriptOptions?]` 元组）由 `normalizeRegistryConfig()` (`src/normalize.ts`) 在模块设置时一次性处理。所有下游消费者（环境默认值、模板插件、自动注入、partytown）接收单一格式。

### 无可变闭包模式
拦截插件使用静态 `proxyPrefix` 字符串注册 —— 没有通过引用捕获的可变数组。插件生成是其输入的纯函数。

### `firstParty: true` 默认
默认为 `true`。对于 `nuxt generate` 和静态预设，会触发带有可操作指导的警告。

### 隐私默认值
GA 默认值（`PRIVACY_HEATMAP`）：匿名化 ip/language/hardware，透传 userAgent/screen/timezone。Browser/OS/Device 报告需要 UA 字符串；`hardware` 标志涵盖跨站指纹识别向量（canvas/WebGL/插件/字体/高熵客户端提示），GA 不需要这些用于标准报告。

`hardware: true` 还会剥离高熵客户端提示（`sec-ch-ua-arch`、`sec-ch-ua-model`），GA4 正将它们用于设备报告。

### 关键文件
- `src/normalize.ts` —— 将注册表配置条目规范化为 `[input, scriptOptions?]` 元组形式
- `src/first-party/proxy-configs.ts` —— 所有带 `domains[]` 和隐私预设的代理配置
- `src/first-party/setup.ts` —— 编排（`setupFirstParty`、`finalizeFirstParty`、`applyAutoInject`）
- `src/first-party/intercept-plugin.ts` —— 客户端 `__nuxtScripts` 包装器（用于非同源 URL 的 proxyUrl）
- `src/first-party/types.ts` —— FirstPartyOptions、ProxyConfig、ProxyAutoInject
- `src/registry.ts` —— 脚本元数据（标签、导入、打包、`proxy: false` 退出选项）；`registryKey` = 代理配置查找键
- `src/registry-logos.ts` —— 从注册表提取的 SVG 标志，用于更小的差异
- `src/runtime/server/proxy-handler.ts` —— 带域名提取和隐私转换的服务端代理
- `src/runtime/server/utils/privacy.ts` —— 隐私解析、IP 匿名化、UA 规范化、负载剥离
- `src/plugins/rewrite-ast.ts` —— AST URL 重写 + canvas 指纹识别中和（通用）；`postProcess` 中的 SDK 特定补丁
- `src/plugins/transform.ts` —— 构建时脚本下载 + URL 重写，透传 `postProcess`
- `src/module.ts` —— ModuleOptions、默认值
- `docs/content/docs/1.guides/2.first-party.md` —— 主文档页面

### 配置选项
- `proxy: false | { prefix?, privacy? }` —— 模块级选项（当任何脚本具有代理能力时自动推断）
- `proxy.prefix` —— 代理端点路径前缀（默认：`/_scripts/p`）
- `assets.prefix` —— 捆绑脚本资源路径（默认：`/_scripts/assets`）
- 每个脚本的 `proxy: false` —— 在扁平配置或 scriptOptions 中用于退出单个脚本
- 注册表级 `proxy: false` —— 在 registry.ts 中用于绝不应被代理的脚本（指纹识别要求）的能力。
