---
title: Carbon Ads 碳广告
description: 在您的 Nuxt 应用中使用 Vue 组件展示碳广告。
links:
  - label: "<ScriptCarbonAds>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptCarbonAds.vue
    size: xs
---

[Carbon Ads](https://www.carbonads.net/) 是一种广告服务，提供对网站展示广告的性能友好型方案。

Nuxt Scripts 提供了一个无头（headless）`ScriptCarbonAds` 组件，用于在您的 Nuxt 应用中嵌入碳广告。

## ScriptCarbonAds

`ScriptCarbonAds` 组件的工作方式与其他 Nuxt Scripts 组件不同，它不依赖于 `useScript`，而是在组件挂载时简单地将一个 script 标签插入组件的 div 中。

默认情况下，组件使用 CarbonAds 的最佳实践，即在挂载时立即加载。如果您希望在特定事件触发时加载广告，可以使用[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。

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

当 CarbonAds 被拦截时，您可以使用这些插槽来添加备用内容。

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

该组件无头渲染，意味着没有内置样式。如果您想自定义广告外观，可以使用来自 nuxt.com 的示例。

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

完整的 props、事件和插槽请参阅[门面组件 API](/docs/guides/facade-components#facade-components-api)。

注意：Carbon Ads 脚本**不**扩展 `useScript` 组合函数。访问该脚本会返回 `HTMLScriptElement`。

### Props

`ScriptCarbonAds` 组件接受以下属性：

- `serve`：Carbon Ads 提供的服务 URL。
- `placement`：Carbon Ads 提供的广告位 ID。
- `format`：Carbon Ads 提供的格式。