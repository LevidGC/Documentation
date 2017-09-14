---
layout: docs-default
---

# 登出端点 (Logout Endpoint)

重定向到登出端点将会清除掉认证会话和 cookie 。

你可以将以下的可选参数传递给此端点：

* `id_token_hint`
    * 认证期间客户端获取到的 `id_token` 。提供 post logout 重定向 URL 可以允许绕过登出确认屏幕。
* `post_logout_redirect_uri`
    * 登出之后 IdentityServer 可以重定向的 URL （默认情况会展示一个链接）。这个 URL 必须在客户端注册的允许的 post logout URL 列表中。

```
/connect/endsession?id_token_hint=...&post_logout_redirect_uri=https://myapp.com
```

参见 [AuthenticationOptions](../configuration/authenticationOptions.html) 获取更多关于登出端点和登出页面的配置。