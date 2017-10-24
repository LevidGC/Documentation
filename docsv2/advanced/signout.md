---
layout: docs-default
---

> [原文](https://identityserver.github.io/Documentation/docsv2/advanced/signout.html)

# 登出支持 (Signout Support)

当用户登出 IdentityServer 的时候，应该尽可能通知用户登录的应用用户现在已经登出了。

IdentityServer 支持两种形式的登出通知。这些风格对应的是两种（三个）不同的 OpenID Connect 会话管理规范：[会话管理](https://openid.net/specs/openid-connect-session-1_0.html) 和 [基于 HTTP 的登出](https://openid.net/specs/openid-connect-logout-1_0.html) 规范。一个对应于基于 JavaScript 的客户端应用，另一个对应的是服务器端 Web 应用（ASP.NET 等等）。

更多 IdentityServer 相关的支持参见这里：

* [基于 JavaScript 的客户端应用](signout-session.html)
* [服务器端 Web 应用](signout-http.html)
