---
title: NPM
description: 在你的 Nuxt 应用中从 NPM 加载 IIFE 脚本。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/npm.ts
    size: xs
---

::script-stats
::

::script-docs
::

## 背景

通常，你会安装一个 [npm](https://www.npmjs.com/) 软件包，并将其与应用一起打包。仅在需要时加载它则需要更多工作：你需要一个[动态 `import()`{lang="ts"}](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import)、一个单独的代码块，以及构建期间所需的任何转译。

[`useScriptNpm()`{lang="ts"}](/scripts/npm){lang="ts"} 注册脚本抽象了这一过程，让你只需一行代码即可加载立即调用函数表达式（IIFE）构建版本。

请将经常使用或关键的软件包保留在 `package.json` 中。通过 CDN 加载最适合偶尔使用的非关键 IIFE。

下面的三个示例通过注册脚本、`useScript` 和 `useHead` 加载同一个文件。

::code-group

```ts [注册脚本 useScriptNpm]
useScriptNpm({
  packageName: 'js-confetti',
  file: 'dist/js-confetti.browser.js',
  version: '0.12.0',
  provider: 'jsdelivr',
})
```

```ts [useScript]
useScript('https://cdn.jsdelivr.net/npm/js-confetti@0.12.0/dist/js-confetti.browser.js')
```

```ts [useHead]
useHead({
  script: [
    { src: 'https://cdn.jsdelivr.net/npm/js-confetti@0.12.0/dist/js-confetti.browser.js' }
  ]
})
```

::

## [`useScriptNpm()`{lang="ts"}](/scripts/npm){lang="ts"}

[`useScriptNpm()`{lang="ts"}](/scripts/npm){lang="ts"} 组合式函数默认使用 unpkg。将 `provider` 设置为 `'jsdelivr'` 或 `'cdnjs'`，即可使用其他受支持的 URL 格式。

```ts
function useScriptNpm<T extends Record<string | symbol, any>>(_options: NpmInput) {}
```

有关触发器、代理和其他脚本选项，请参阅[注册表脚本](/docs/guides/registry-scripts)。

### 映射已加载的全局变量

泛型类型描述了代理，但不会发现浏览器全局变量。使用 `scriptOptions.use` 显式映射已加载的库：

```ts
interface SomeApi {
  doSomething: () => void
}

const { proxy } = useScriptNpm<SomeApi>({
  packageName: 'some-api',
  scriptOptions: {
    use: () => (window as Window & { SomeApi: SomeApi }).SomeApi,
  },
})

proxy.doSomething()
```

如果没有 `scriptOptions.use`，IIFE 仍会加载，但组合式函数不会通过 `proxy` 或 `onLoaded` 暴露其全局 API。

::script-types
::

## 示例

更多示例请参见 [教程：加载 js-confetti](/docs/getting-started/confetti-tutorial)。
