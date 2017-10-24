---
layout: docs-default
---

> [原文](https://identityserver.github.io/Documentation/docsv2/advanced/signout-session.html)

# 基于 JavaScript 的客户端应用会话管理 (Session management for client-side JavaScript-based applications)

[会话管理](https://openid.net/specs/openid-connect-session-1_0.html) 规范为 OpenID Connect 提供商提供了通知基于 JavaScript 的客户端用户登出的通知机制。

规范中定义的机制需要 JavaScript 应用打开一个连接到 OpenID Connect 提供商 "check\_session\_iframe" （它的值在元数据端点可以获取到）的 `<iframe>` 。这个 `<iframe>` 可以获取到由 OP 管理的 cookie （鉴于它和 OP 同源）并能侦测到用户登录会话的更改（意味着用户已经登出了或者有另一个用于已登录）。

JavaScript 客户端应用可以定期在 `<iframe>` 中使用 `postMessage` 来询问用户的会话是否发生更改。而 `<iframe>` 会回复 `"changed"` 或者 `"unchanged"` 。如果响应是 "changed" ，那么 JavaScript 应用就会知道用户的会话已经结束了（或者通过某种方式更改了），然后再执行必要的清理工作。

**想要在登出通知上应用此技术，参照 [这里](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Clients/JavaScriptImplicitClient) 的JavaScript 应用样例 **
