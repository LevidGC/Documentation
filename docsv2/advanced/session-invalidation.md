---
layout: docs-default
---

> [原文](https://identityserver.github.io/Documentation/docsv2/advanced/session-invalidation.html)

# 验证会话作废 (Authentication Session Invalidation) （v2.4 新增）

IdentityServer3 定义了 `IAuthenticationSessionValidator` 接口用于作废一个存在的登录会话。本质上来说，这可用于忽略一个登录的用户的认证 cookie （通常是由于外部的事件，比如用户在登录后更改了密码）。用户将会被认为是匿名的，一般意味着他们必须重新认证来继续使用 IdentityServer 。

## IAuthenticationSessionValidator

接口定义了一个方法：

* `IsAuthenticationSessionValidAsync`
 * 只要登录的用户向 IdentityServer 提供验证 cookie 就会调用此方法。返回 `true` 表示此验证 cookie 是有效的，反之为 `false` 。
 * `ClaimsPrincipal` 表示认证过的用户，作为参数传递给此方法。
