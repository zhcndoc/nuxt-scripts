---
title: Google AdSense
description: 在您的 Nuxt 应用中展示 Google AdSense 广告。
links:
  - label: useScriptGoogleAdsense
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-adsense.ts
    size: xs
  - label: "<ScriptGoogleAdsense>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptGoogleAdsense.vue
    size: xs
---

[Google AdSense](https://www.google.com/adsense/start/) 会在您的网站上投放 Google 广告。

选择与展示位置相匹配的 API：

- [`useScriptGoogleAdsense()`{lang="ts"}](/scripts/google-adsense){lang="ts"} 加载 `adsbygoogle` 队列。
- `<ScriptGoogleAdsense>`{lang="html"} 渲染一个广告单元。

## 全局配置

::script-docs
::

## 在哪里查找 `<your-id>`{lang="html"}（发布商 ID）

在 **帐号 > 设置 > 帐户信息** 下查找您的 [Google AdSense 发布商 ID](https://support.google.com/adsense/answer/2923881?hl=en)。它在您的帐号中显示为 `pub-…`，在广告代码中显示为 `ca-pub-…`。将下方的 `<your-id>`{lang="html"} 替换为该值。

::callout{icon="i-heroicons-light-bulb" to="https://adsense.google.com/start/" target="_blank"}
从 AdSense 信息中心的 **自动广告** 设置中管理广告类型和展示位置。
::

## 网站所有权验证

### 自动插入 meta 标签

如果提供了 `client`，页面将**自动**插入一个用于 Google 验证您站点所有权的 **meta 标签**。

::tabs  
  ::div  
  ---  
  label: 示例  
  icon: i-heroicons-code-bracket-square  
  ---  
  ```ts [nuxt.config.ts]
  export default defineNuxtConfig({
    scripts: {
      registry: {
        googleAdsense: {
          client: "ca-pub-<your-id>", // AdSense 发布者 ID
        },
      },
    },
  })
  ```  
  ::  
  ::div  
  ---  
  label: 输出  
  icon: i-heroicons-magnifying-glass-circle  
  ---  
  ```html
  <meta name="google-adsense-account" content="ca-pub-<your-id>" />
  ```  
  ::  
::  

### 添加 `ads.txt`

Google 建议添加 `ads.txt` 文件，以识别获授权出售您广告资源的广告系统，并减少仿冒广告资源。

#### 步骤

1. 新建文件：`public/ads.txt`  
2. 添加以下内容：  
   ```plaintext
   google.com, pub-<your-id>, DIRECT, f08c47fec0942fa0
   ```  
3. 将 `<your-id>` 替换为您的 **AdSense 发布者 ID**。  

::callout{icon="i-heroicons-light-bulb" to="https://support.google.com/adsense/answer/12171612" target="_blank"}
`ads.txt` 文件不能替代 AdSense 的网站审核。发布该文件后，请在您的 AdSense 控制台中检查文件状态。
::

## 启用自动广告

[自动广告](https://support.google.com/adsense/answer/9261805?hl=en)会根据网页的布局、内容和现有广告，由 Google 选择广告展示位置。您仍然可以在 AdSense 中控制广告格式，以及排除某些网页或区域。

::tabs  
  ::div  
  ---  
  label: 示例  
  icon: i-heroicons-code-bracket-square  
  ---  
  ```ts [nuxt.config.ts]
  export default defineNuxtConfig({
    scripts: {
      registry: {
        googleAdsense: {
          client: "ca-pub-<your-id>", // AdSense 发布者 ID
          autoAds: true, // 启用自动广告
        },
      },
    },
  })
  ```  
  ::  
  ::div  
  ---  
  label: 输出  
  icon: i-heroicons-magnifying-glass-circle  
  ---  
  ```html
  <script>
  (adsbygoogle = window.adsbygoogle || []).push({
    google_ad_client: "ca-pub-<your-id>",
    enable_page_level_ads: true,
  });
  </script>
  ```  
  ::  
::  

## 使用 [`<ScriptGoogleAdsense>`{lang="html"}](/scripts/google-adsense){lang="html"} 组件

`<ScriptGoogleAdsense>`{lang="html"} 会渲染一个广告单元：

```vue
<template>
  <ScriptGoogleAdsense
    data-ad-client="ca-pub-<your-id>"
    data-ad-slot="1234567890"
    data-ad-format="auto"
  />
</template>
```

### 组件属性

| 属性                         | 描述                                                               |
| ---------------------------- | ------------------------------------------------------------------ |
| `data-ad-client`             | 您的 **Google AdSense 发布商 ID**（`ca-pub-XXXXXXXXXX`）。          |
| `data-ad-slot`               | 您的 **广告位 ID**（可在 AdSense 信息中心获取）。                   |
| `data-ad-format`             | 广告格式类型（`auto`、`rectangle`、`horizontal`、`vertical`、`fluid`、`autorelaxed`）。 |
| `data-ad-layout`             | 布局（`in-article`、`in-feed`、`fixed`）。                          |
| `data-full-width-responsive` | **设置为 `true`** 以使广告具有响应式效果。                         |

#### 使用 `data-ad-layout` 的示例

为 `in-article` 等布局设置 `data-ad-layout`：

```vue
<template>
  <ScriptGoogleAdsense
    data-ad-client="ca-pub-<your-id>"
    data-ad-slot="1234567890"
    data-ad-format="fluid"
    data-ad-layout="in-article"
  />
</template>
```

## 处理广告拦截器

对于广告拦截器阻止脚本的访客，请使用 `error` 插槽：

```vue
<template>
  <ScriptGoogleAdsense data-ad-client="ca-pub-..." data-ad-slot="...">
    <template #error>
      <!-- 备用内容 -->
      <p>请支持我们，关闭您的广告屏蔽程序。</p>
    </template>
  </ScriptGoogleAdsense>
</template>
```

## 使用 [`useScriptGoogleAdsense()`{lang="ts"}](/scripts/google-adsense){lang="ts"} 组合式函数

当你需要 `adsbygoogle` 队列而不需要广告单元组件时，请使用 [`useScriptGoogleAdsense()`{lang="ts"}](/scripts/google-adsense){lang="ts"}。

```ts
export function useScriptGoogleAdsense<T extends GoogleAdsenseApi>(
  _options?: GoogleAdsenseInput
) {}
```

请参阅[注册脚本指南](/docs/guides/registry-scripts)，了解触发和加载选项。

::callout{icon="i-heroicons-light-bulb" to="https://support.google.com/adsense" target="_blank"}
请参阅官方的 **Google AdSense 指南**。
::

::script-types
::
