---
title: Carbon 广告
description: 使用 Vue 组件在您的 Nuxt 应用中展示 Carbon 广告。
links:
  - label: "<ScriptCarbonAds>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptCarbonAds.vue
    size: xs
---

[Carbon Ads](https://www.carbonads.net/) 通过标准嵌入脚本投放广告。

使用无头 [`<ScriptCarbonAds>`{lang="html"}](/scripts/carbon-ads){lang="html"} 组件，在 Nuxt 页面中放置一个广告。

::script-stats
::

::script-docs
::

## [`<ScriptCarbonAds>`{lang="html"}](/scripts/carbon-ads){lang="html"}

与其他 Nuxt Scripts 组件不同，`<ScriptCarbonAds>`{lang="html"} 不使用 [`useScript()`{lang="ts"}](/docs/api/use-script){lang="ts"}，而是将 Carbon 脚本插入其自身的 `div` 中。

组件挂载时加载。如果广告应等待特定交互，请传入一个[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。

Carbon 的[展示政策](https://www.carbonads.net/placement-policy)要求每个页面只加载一次广告代码，并禁止修改或自行托管该脚本。请为当前页面渲染一个组件。该组件会直接请求 Carbon 的 CDN 脚本，但不会强制限制只能使用一个组件。

```vue
<template>
  <ScriptCarbonAds
    serve="..."
    placement="..."
    format="..."
  />
</template>
```

### 处理广告拦截器

当 Carbon Ads 被拦截时，使用 `error` 插槽：

```vue
<template>
  <ScriptCarbonAds
    serve="..."
    placement="..."
    format="..."
  >
    <template #error>
      <!-- 备用广告 -->
      请关闭广告拦截器支持我们。
    </template>
  </ScriptCarbonAds>
</template>
```

### 添加用户界面样式

该组件不会继承任何样式。以下示例使用了 nuxt.com 中的样式：

```vue
<template>
  <ScriptCarbonAds
    class="Carbon border border-gray-200 dark:border-gray-800 rounded-lg bg-white dark:bg-white/5"
    serve="..."
    placement="..."
    format="..."
  />
</template>

<style lang="postcss">
/* 感谢 nuxt.com */
.dark .Carbon {
  min-height: 220px;
  .carbon-text {
    @apply text-gray-400;

    &:hover {
      @apply text-gray-200;
    }
  }
}

.light .Carbon {
  .carbon-text {
    @apply text-gray-600;

    &:hover {
      @apply text-gray-800;
    }
  }
}

.Carbon {
  @apply p-3 flex flex-col max-w-full;

  @screen sm {
    @apply max-w-xs;
  }

  @screen lg {
    @apply mt-0;
  }

  #carbonads span {
    @apply flex flex-col justify-between;

    .carbon-wrap {
      @apply flex flex-col;

      flex: 1;

      @media (min-width: 320px) {
        @apply flex-row;
      }

      @screen lg {
        @apply flex-col;
      }

      .carbon-img {
        @apply flex items-start justify-center mb-4;

        @media (min-width: 320px) {
          @apply mb-0;
        }

        @screen lg {
          @apply mb-4;
        }
      }

      .carbon-text {
        @apply flex-1 text-sm w-full m-0 text-left block;

        &:hover {
          @apply no-underline;
        }

        @media (min-width: 320px) {
          @apply ml-4;
        }

        @screen lg {
          @apply ml-0;
        }
      }
    }
  }

  img {
    @apply w-full;
  }

  & .carbon-poweredby {
    @apply ml-2 text-xs text-right text-gray-400 block pt-2;

    &:hover {
      @apply no-underline text-gray-500;
    }
  }
}
</style>
```

### 组件 API

完整的 props、事件和插槽请参阅 [门面组件 API](/docs/guides/facade-components#facade-components-api)。

完整的 props、事件和插槽请参阅[门面组件 API](/docs/guides/facade-components#facade-components-api)。

组件的 `ready` 事件会接收注入的 `HTMLScriptElement`。

::script-types
::
