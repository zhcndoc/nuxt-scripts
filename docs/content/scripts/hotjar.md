---
title: Hotjar
description: 加载 Hotjar 并将事件、用户属性和虚拟页面浏览进行排队。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/hotjar.ts
  size: xs
---

[Hotjar](https://help.hotjar.com/hc/en-us/articles/36820019634961-What-is-Hotjar) 集成了会话录制、热图、调查和用户反馈工具。

::script-stats
::

::script-docs
::

## 事件和用户属性

通过 `proxy.hj` 发出的调用会排队，直到脚本准备就绪。想要根据特定操作筛选录制内容或热图时，请发送事件：

```ts
const { proxy } = useScriptHotjar()

function recordSignup() {
  proxy.hj('event', 'signup_completed')
}
```

Hotjar 的 [Events API 参考](https://help.hotjar.com/hc/en-us/articles/36819965075473-Events-API-Reference)列出了事件名称限制和筛选限制。事件不接受属性。

请改用 Identify API 来传递用户属性：

```ts
const { proxy } = useScriptHotjar()

proxy.hj('identify', user.id, {
  plan: user.plan,
})
```

发送属性前，请查看 Hotjar 的 [Identify API 隐私和数据指南](https://help.hotjar.com/hc/en-us/articles/36820006120721-Identify-API-Reference)。相比电子邮件地址，建议优先使用稳定的内部 ID，并且只发送所需的字段。

::script-types
::
