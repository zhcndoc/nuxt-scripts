---
title: DeskCrew
description: 为你的 Nuxt 应用添加一个延迟加载的 DeskCrew 支持小组件
links:
  - label: useScriptDeskCrew
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/deskcrew.ts
    size: xs
  - label: "<ScriptDeskCrew>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptDeskCrew.vue
    size: xs
---

[DeskCrew](https://deskcrew.io/) 是一个支持小组件，结合了实时聊天、来自你的知识库的 AI 答案、帮助中心和更新日志。

使用 [`useScriptDeskCrew()`{lang="ts"}](#usescriptdeskcrew){lang="ts"} 直接调用 SDK，或使用 [`<ScriptDeskCrew>`{lang="html"}](#scriptdeskcrew){lang="html"} 创建自定义聊天启动器。

::script-stats
::

::script-docs
::

## [`<ScriptDeskCrew>`{lang="html"}](/scripts/deskcrew){lang="html"}

无头门面会一直阻止 DeskCrew 小组件加载，直到其[元素触发器](/docs/guides/script-triggers#element-event-triggers)触发。它默认监听 `click`，因此从未打开聊天的访客不会下载任何小组件内容。

### 组件 API

完整的 props、事件和插槽请参阅[门面组件 API](/docs/guides/facade-components#facade-components-api)。

#### 使用环境变量

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      deskcrew: { trigger: 'onNuxtReady' },
    }
  },
  runtimeConfig: {
    public: {
      scripts: {
        deskcrew: {
          widgetKey: '', // NUXT_PUBLIC_SCRIPTS_DESKCREW_WIDGET_KEY
          board: '', // NUXT_PUBLIC_SCRIPTS_DESKCREW_BOARD
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_DESKCREW_WIDGET_KEY=<YOUR_PUBLIC_KEY>
NUXT_PUBLIC_SCRIPTS_DESKCREW_BOARD=<YOUR_BOARD_SLUG>
```

### 事件

组件会在小组件挂载其启动器后发出一次 `ready` 事件，如果脚本加载失败则发出 `error` 事件。

### 插槽

`awaitingLoad`、`loading`、`error` 和默认插槽的行为与门面组件文档中的说明一致。

## [`useScriptDeskCrew()`{lang="ts"}](/scripts/deskcrew){lang="ts"}

```ts
export function useScriptDeskCrew<T extends DeskCrewApi>(_options?: DeskCrewInput) {}
```

::script-types
::

### 识别访客

身份标识是由你自己的后端生成的签名令牌，因此它属于运行时调用，而不是 `nuxt.config` 选项。`nuxt.config` 中的所有内容都是部署时常量，如果将某位访客的令牌嵌入构建结果，就会将该身份交给其他每一位访客。

```vue
<script setup lang="ts">
const { proxy } = useScriptDeskCrew({ widgetKey: 'pub_xxxxxxxx' })
const { data } = await useFetch('/api/deskcrew-token')
watchEffect(() => {
  if (data.value?.token)
    proxy.identify({ token: data.value.token })
})
</script>
```

### 其他界面

`embed()`{lang="ts"}、`changelog()`{lang="ts"} 和 `surveys()`{lang="ts"} 都会挂载一个界面。每个页面最多调用一次：第二次调用会记录警告并且不执行任何操作。

::callout
DeskCrew 从其自身的源提供小组件，并根据脚本的 `src` 推导其 API 端点，因此此脚本不支持[捆绑](/docs/guides/bundling)或[第一方模式](/docs/guides/first-party)。它会直接从 `deskcrew.io` 加载。
::
