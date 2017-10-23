---
layout: docs-default
---

# 认证选项 (Authentication Options)

**本部分示例参见 [这里](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/CustomUserService)**

 `AuthenticationOptions` 是 `IdentityServerOptions` 的一个属性，用于自定义登录和登出的视图和行为。

* `EnableLocalLogin`
    * 指示 IdentityServer 是否允许使用本地账户来验证用户。禁用这个设置将不会在登录页面展示用户名/密码表单。这也会禁用掉资源所有者密码流。默认为 `true` 。
* `EnableLoginHint`
    * 指示 `login_hint` 参数是否用于预填充 username 字段。默认为 `true` 。
* `LoginPageLinks`
    * `LoginPageLink` 对象列表。这允许在登录视图上提供用户自定义链接，这些链接到的页面可能需要在登录之前访问（比如注册页面，或者密码重置页面）。
    * `LoginPageLink` 包含：
        * `Type`: 链接类型的标识符。
        * `Text`: 链接上显示的文字。
        * `Href`: 链接的 URL 。
    * `LoginPageLink` 表示的自定义页面应该由宿主应用提供。一旦它完成了任务，就应该通过将用户重定向到登录视图恢复登录流。
    * 当一个用户使用了一个 `LoginPageLink` 时，`signin` 查询字符串参数将会传递到这个页面。在用户想要恢复登录界面时需要将这个查询字符串参数作为 `siginin` 回传到登录界面。登录视图可以通过 "~/login" 路径定位到，它相对于 IdentityServer 的应用基地址。
* `RememberLastUsername`
    * Indicates whether IdentityServer will remember the last username entered on the login page. Defaults to `false`.
* `IdentityProviders`
    * Allows configuring additional identity providers - see [here](identityProviders.html).
* `CookieOptions`
    * `CookieOptions` object that configures how cookies are managed by IdentityServer. 
    * `CookieOptions` has these properties:
        * `Prefix`: Allows setting a prefix on cookies to avoid potential conflicts with other cookies with the same names. By default no prefix is used.
        * `ExpireTimeSpan`: The expiration duration of the authentication cookie. Defaults to `10` hours.
        * `IsPersistent`: Indicates whether the authentication cookie is marked as persistent. Defaults to `false`.
        * `SlidingExpiration`: Indicates if the authentication cookie is sliding, which means it auto renews as the user is active. Defaults to `false`.
        * `Path`: Sets the cookie path. Defaults to the base path of IdentityServer in the hosting application.
        * `AllowRememberMe`: Indicates whether the "remember me" option is presented to users on the login page. If selected this option will issue a persistent authentication cookie. Defaults to `true`.
          * If this setting is in use then the user's decision (either yes or no) will override the `IsPersistent` setting. In other words, if both `IsPersistent` and `AllowRememberMe` is enabled and the user decides to not remember their login, then no persistent cookie will be issued.
        * `RememberMeDuration`: Duration of the persistent cookie issued by the "remember me" option on the login page. Defaults to `30` days.
        * `SecureMode`: Gets or sets the mode for issuing the secure flag on the cookies issued. Defaults to SameAsRequest.
* `RequireSignOutPrompt` (added in v2.4)
    * Gets or sets a value indicating whether IdentityServer will always show a confirmation page for sign-out. Defaults to `false`.
* `EnableSignOutPrompt`
    * Indicates whether IdentityServer will show a confirmation page for sign-out. When a client initiates a sign-out, by default IdentityServer will ask the user for confirmation. This is a mitigation technique against "logout spam". Defaults to `true`.
* `EnablePostSignOutAutoRedirect`
    * Gets or sets a value indicating whether IdentityServer automatically redirects back to a validated `post_logout_redirect_uri` passed to the signout endpoint. Defaults to `false`.
* `PostSignOutAutoRedirectDelay`
    * Gets or sets the delay (in seconds) before redirecting to a `post_logout_redirect_uri`. Defaults to `0`.
* `SignInMessageThreshold`
    * Gets or sets the limit after which old signin messages (cookies) are purged. Defaults to `5`.
* `InvalidSignInRedirectUrl`
    * Gets or sets the invalid sign in redirect URL. If the user arrives at the login page without a valid sign-in request, then they will be redirected to this URL. The URL must be absolute or can relative URLs (starting with "~/").
