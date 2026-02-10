---
title: NPM
description: 在你的 Nuxt 应用中从 NPM 加载 IIFE 脚本。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/npm.ts
    size: xs
---

## 背景

使用 NPM 文件时，通常会将它们作为 `package.json` 文件中的 node_module 依赖包含进来。然而，
优化这些脚本的加载较为困难，需要从单独的代码块动态导入模块，
并且仅在需要时加载。它还会拖慢构建速度，因为模块需要被转译。

`useScriptNpm` 注册脚本抽象了这一过程，使你可以通过一行代码加载那些已经导出为立即调用函数（IIFE）的脚本。

在许多情况下，将脚本作为 `package.json` 文件中的依赖可能更合理，但对于不常用或对应用不关键的脚本，
这可以是一个很好的替代方案。

一开始，我们可以将使用此脚本视为 `useHead` 组合式函数的替代方案。以下代码示例展示了该抽象层次。

::code-group

```ts [Registry Script useScriptNpm]
useScriptNpm({
  packageName: 'js-confetti',
  file: 'dist/js-confetti.browser.js',
  version: '0.12.0',
})
```

```ts [useScript]
useScript('https://cdn.jsdelivr.net/npm/js-confetti@0.12.0/dist/js-confetti.browser.js')
```

```ts [useHead]
useHead({
  script: [
    { src: 'https://cdn.jsdelivr.net/npm/js-confetti@latest/dist/js-confetti.browser.js' }
  ]
})
```

::

## useScriptNpm

`useScriptNpm` 组合式函数让你可以精细地控制你的网站上 NPM 脚本何时以及如何加载。

```ts
function useScriptNpm<T extends Record<string | symbol, any>>(_options: NpmInput) {}
```

请参考 [注册脚本指南](/docs/guides/registry-scripts) 了解更多进阶用法。

### NpmOptions

```ts
export const NpmOptions = object({
  packageName: string(),
  file: optional(string()),
  version: optional(string()),
  type: optional(string()),
})
```

### 返回值

要获取所加载脚本的类型，你需要扩展 `useScriptNpm` 函数的类型定义。

```ts
interface SomeApi {
  doSomething: () => void
}
useScriptNpm<SomeApi>({
  packageName: 'some-api'
})
```

## 示例

更多示例请参见 [教程：加载 js-confetti](/docs/getting-started/confetti-tutorial)。