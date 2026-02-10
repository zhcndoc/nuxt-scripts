---
title: Google Adsense
description: 在您的 Nuxt 应用中显示 Google Adsense 广告。
links:
  - label: useScriptGoogleAdsense
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-adsense.ts
    size: xs
  - label: "<ScriptGoogleAdsense>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptGoogleAdsense.vue
    size: xs
---

[Google AdSense](https://www.google.com/adsense/start/) 允许您通过展示与内容相关的 Google 广告来实现网站的盈利。

Nuxt Scripts 提供：

- `useScriptGoogleAdsense`：用于动态管理 Google AdSense 的组合函数。
- `<ScriptGoogleAdsense>`：一个无头组件，用于将广告直接嵌入您的 Nuxt 应用。

## 全局配置

您可以在 `nuxt.config.ts` 中全局配置 Google AdSense，这样脚本将在所有页面自动加载。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleAdsense: {
        client: "ca-pub-<your-id>", // 您的 Google AdSense 发布者 ID
        autoAds: true, // 启用自动广告
      },
    },
  },
});
```

## 如何找到 `<your-id>`（发布者 ID）

您的 **Google AdSense 发布者 ID**（也称为 `ca-pub-XXXXXXX`）可以在您的 **Google AdSense 账户**中找到：

1. 登录您的 **Google AdSense** 账户。
2. 进入 **账户 > 设置**（点击您的头像图标 > “账户信息”）。
3. 在 **账户信息** 下找到 **发布者 ID**。
4. 将上述配置中的 `<your-id>` 替换为您的实际 ID。

::callout{icon="i-heroicons-light-bulb" to="https://adsense.google.com/start/" target="_blank"}
您也可以在 **Google AdSense 控制台**中管理 **自动广告设置**，以控制*广告类型、广告位置及营收优化*。
::

## 站点所有权验证

### 自动插入 Meta 标签

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
  });
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

### 使用 `ads.txt` 进行验证

Google 建议添加 `ads.txt` 文件以确保**广告收入资格**。

#### 步骤：

1. 新建文件：`public/ads.txt`
2. 添加以下内容：
   ```plaintext
   google.com, pub-<your-id>, DIRECT, f08c47fec0942fa0
   ```
3. 将 `<your-id>` 替换为您的 **AdSense 发布者 ID**。

::callout{icon="i-heroicons-light-bulb"}
**为什么使用 `ads.txt`？** 它有助于**防止广告欺诈**，确保**只有您的网站**可以展示您的广告。
::

## 启用自动广告

自动广告允许 Google **自动**投放广告，实现**更优的优化**。

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
  });
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

## 使用 `ScriptGoogleAdsense` 组件

它提供了一种简单方式，将广告嵌入您的 Nuxt 应用。

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

| 属性                         | 说明                                                               |
| ---------------------------- | ------------------------------------------------------------------ |
| `data-ad-client`             | 您的 **Google Adsense 发布者 ID**（`ca-pub-XXXXXXXXXX`）。          |
| `data-ad-slot`               | 您的 **广告位 ID**（可在 AdSense 控制台获得）。                     |
| `data-ad-format`             | 广告格式类型（`auto`，`rectangle`，`horizontal`，`vertical`，`fluid`，`autorelaxed`）。 |
| `data-ad-layout`             | 布局（`in-article`，`in-feed`，`fixed`）。                          |
| `data-full-width-responsive` | **设置为 `true`** 可使广告响应式。                                 |

#### 使用 `data-ad-layout` 的示例

若要为广告指定布局（如 “in-article”），可使用 `data-ad-layout` 属性：

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

## 如何处理广告屏蔽程序？

如果用户启用了广告屏蔽程序，您可以显示**备用内容**。

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

## 使用 `useScriptGoogleAdsense` 组合函数

`useScriptGoogleAdsense` 组合函数允许您对 AdSense 脚本进行**细粒度控制**。

```ts
export function useScriptGoogleAdsense<T extends GoogleAdsenseApi>(
  _options?: GoogleAdsenseInput
) {}
```

请参阅 [Registry Scripts 指南](/docs/guides/registry-scripts) 获取高级用法。

## GoogleAdsenseApi 接口

该接口定义了 Google Adsense API 结构，增强 TypeScript 支持。

```ts
export interface GoogleAdsenseApi {
  adsbygoogle: any[] & { loaded: boolean };
}
```

## GoogleAdsenseInput

您可以使用以下结构定义传给 `useScriptGoogleAdsense` 组合函数的输入选项：

```ts
export const GoogleAdsenseOptions = object({
  /**
   * Google Adsense 发布者 ID。
   */
  client: optional(string()),
  /**
   * 启用或禁用自动广告。
   */
  autoAds: optional(boolean()),
});
```

::callout{icon="i-heroicons-light-bulb" to="https://support.google.com/adsense" target="_blank"}
需要更多帮助？请查看官方 **Google AdSense 指南**
::