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
    * 指示 IdentityServer 是否记住最近在登录页面输入的 username 。默认为 `false` 。
* `IdentityProviders`
    * 允许配置额外的身份提供商——参见 [这里](identityProviders.html).
* `CookieOptions`
    * `CookieOptions` 配置 IdentityServer 如何管理 cookie 。
    * `CookieOptions` 有这些属性：
        * `Prefix`: 允许在 cookie 的名称前设置前缀来避免潜在的名称冲突。默认没有使用前缀。
        * `ExpireTimeSpan`: 认证 cookie 的过期持续时间。默认为 `10` 小时。
        * `IsPersistent`: 指示认证 cookie 是否标记为持续的。默认为 `false` 。
        * `SlidingExpiration`: 指示认证 cookie 是否为滑动的，这意味这在用户活动的情况下它是会被自动更新的。默认为 `false` 。
        * `Path`: 设置 cookie 的路径。默认为 IdentityServer 在托管应用中的基路径。
        * `AllowRememberMe`: 指示“记住我”选项是否展示在登录页面。如果勾选了这个选项将会颁发一个持续的认证 cookie 。默认为 `true` 。
          * 如果启用这个设置，用户的选择（不管是 yes 还是 no）都会覆盖 `IsPresistent` 设置。换句话说就是，如果同时启用 `IsPersistent` 和 `AllowRememberMe` 并且用户在登录也选择了不记住我，那么将不会颁发持续的 cookie 。
        * `RememberMeDuration`: 在登录页面上勾选“记住我”之后签署的持续的 cookie 的持续时间。默认为 `30` 天。
        * `SecureMode`: 获取或设置在签发的 cookie 上签署安全标记的模式。默认为 SameAsRequest 。
* `RequireSignOutPrompt` （v2.4 新增）
    * 获取或设置指示 IdentityServer 是否在登出的时候总是展示一个确认页的值。默认为 `false` 。
* `EnableSignOutPrompt`
    * 指示 IdentityServer 是否在登出的时候展示一个确认页。当客户端初始化登出的时候，默认情况 IdentityServer 会征询用户的确认。这是针对 "logout spam" 的缓解技术。默认为 `true` 。
* `EnablePostSignOutAutoRedirect`
    * 获取或设置指示 IdentityServer 是否自动重定向回一个合法 `post_logout_redirect_uri` 的值。默认为 `false` 。
* `PostSignOutAutoRedirectDelay`
    * 获取或设置重定向到一个 `post_logout_redirect_uri` 前的延迟。默认为 `0` 。
* `SignInMessageThreshold`
    * 获取或设置清除旧的登录消息 (cookie) 后的限制。默认为 `5` 。
* `InvalidSignInRedirectUrl`
    * 获取或设置非法的登录重定向 URL 。如果用户打开登录页面的时候没有携带合法的登录请求，然后他们将会被重定向到这个 URL 。这个 URL 必须是绝对地址或者是开头为 "~/" 的相对 URL 。
