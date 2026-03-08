---

title: TikTok 像素  
description: 在你的 Nuxt 应用中使用 TikTok 像素。  
links:  
- label: 源代码  
  icon: i-simple-icons-github  
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/tiktok-pixel.ts  
  size: xs  

---

[TikTok 像素](https://ads.tiktok.com/help/article/tiktok-pixel) 让你能够测量、优化并为你的 TikTok 广告活动构建受众。

Nuxt Scripts 提供了一个注册脚本组合式函数 [`useScriptTikTokPixel()`{lang="ts"}](/scripts/tiktok-pixel){lang="ts"}，方便你在 Nuxt 应用中集成 TikTok 像素。

::script-stats
::

::script-docs
::

::script-types
::

## 识别用户

你可以识别用户以进行高级匹配：

```ts
const { proxy } = useScriptTikTokPixel()

proxy.ttq('identify', {
  email: 'user@example.com',
  phone_number: '+1234567890'
})
```

## 禁用自动页面浏览追踪

默认情况下，TikTok 像素会自动追踪页面浏览。若要禁用：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      tiktokPixel: {
        id: 'YOUR_PIXEL_ID',
        trackPageView: false
      }
    }
  }
})
```
