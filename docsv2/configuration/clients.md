---
layout: docs-default
---

# 客户端 (Clients)

`Client` 类是对 OpenID Connect 或者 OAuth2 客户端的建模——比如，本地应用，Web 应用或者基于 JS 的应用（[链接](https://github.com/IdentityServer/IdentityServer3/blob/master/source/Core/Models/Client.cs)）。

* `Enabled`
    * 客户端是否启用。默认为 `true` 。
* `ClientId`
    * 客户端唯一的 ID
* `ClientSecrets`
    * 客户端 secret 列表——只与需要 secret 的流相关
* `ClientName`
    * 客户端显示的名称（用于登录和 consent screen）
* `ClientUri`
    * 用于获取客户端更多信息的 URI （用于 consent screen）
* `LogoUri`
    * 客户端 Logo URI （用于 consent screen）
* `RequireConsent`
    * 指示是否需要 consent screen 。默认为 `true` 。
* `AllowRememberConsent`
    * 用户是否可以存储 consent 决定。默认为 `true` 。
* `Flow`
    * 客户端允许使用的流 (`AuthorizationCode`, `Implicit`, `Hybrid`, `ResourceOwner`, `ClientCredentials`, `Custom`) 。默认为 `Implicit` 。
* `AllowClientCredentialsOnly `
    * 获取或者设置用于指示客是否允许客户端仅使用客户端凭据来请求令牌的值。如果你想让一个客户端同时使用以用户为中心的流（比如，隐式流）和额外的客户端凭据流时就会有用。默认为 `fasle` 。只应该用于机密性的客户端（比如，不是采用隐式流的）。
* `RedirectUris`
    * 指定允许返回令牌或者授权码的 URI
* `PostLogoutRedirectUris`
    * 指定登出之后允许重定向的 URI
* `LogoutUri` （v2.2 新增）
    * 为基于 HTTP 登出的客户端指定登出 URI
* `LogoutSessionRequired` （v2.2 新增）
    * 指定用户会话 id 是否要发送到 LogoutUri 。默认为 `true` 。
* `RequireSignOutPrompt` (added in v2.4)
    * 指定客户端在登出的时候是否总是要展示一个确认页。默认为 `false` 。
* `AllowedScopes`
    * 默认情况，客户端不能获取任何的域——可以显式指定相应的域（推荐），也可以将 `AllowAccessToAllScopes` 设置为 `true` 。
* `AllowAccessTokensViaBrowser` (added in v2.5)
    * 指定客户端是否允许通过浏览器来请求访问令牌。这对于允许多种响应类型的流来说非常有用（比如，通过禁用混合流客户端通过使用 `code id_token` 来添加 `token` 响应类型，这会导致将令牌泄露给浏览器）。
* `AllowedCustomGrantTypes`
    * 当使用 `Custom` 流，你需要指定客户端可以使用哪一种自定义的许可类型。这里（推荐）明确指定许可类型或者将 `AllowAccessToAllCustomFrantTypes` 设置为 `true` 。
* `IdentityTokenLifetime`
    * 以秒为计量单位的身份令牌的生命周期（默认为 300 秒 / 5 分钟）
* `AccessTokenLifetime`
    * 以秒为计量单位的访问令牌的生命周期（默认为 3600 秒 / 1 小时）
* `AuthorizationCodeLifetime`
    * 以秒为计量单位的授权码的生命周期（默认为 300 秒 / 5 分钟）
* `AbsoluteRefreshTokenLifetime`
    * 以秒为计量单位的刷新令牌的最大生命周期。默认为 2592000 秒 / 30 天
* `SlidingRefreshTokenLifetime`
    * 以秒为计量单位的刷新令牌的滑动生命周期 (sliding lifetime) 。默认为 1296000 秒 / 15 天
* `RefreshTokenUsage`
    * `ReUse`: 当刷新令牌的时候刷新令牌句柄 (refresh token handle) 会保持不变
    * `OneTime`: 当刷新令牌的时候刷新令牌句柄会被更新
* `RefreshTokenExpiration`
    * `Absolute`: 刷新令牌在固定的时间点失效（通过 `AbsoluteRefreshTokenLifetime` 指定）
    * `Sliding`: 当刷新令牌时候，刷新令牌的生命周期会被更新（通过 `SlidingRefreshTokenLifetime` 指定数量）。声明周期不会超过 `AbsoluteRefreshTokenLifetime` 。
* `UpdateAccessTokenClaimsOnRefresh`
    * 获取或者设置指示访问令牌（包括它的值）是否应该在刷新令牌请求的时候进行更新的值。
* `AccessTokenType`
    * 指定访问令牌是否为一个引用令牌还是一个自包含的 JWT 令牌（默认为 `Jwt`）。
* `EnableLocalLogin`
    * 指定客户端是否可以使用本地账户还是仅仅为外部身份提供商。默认为 `true` 。
* `IdentityProviderRestrictions`
    * 指定这个客户端可以使用的外部身份提供商（如果列表为空，则可以使用所有的身份提供商）。默认为空。
* `IncludeJwtId`
    * 指定 JWT 访问令牌是否应该有一个唯一的 ID （通过 `jti` 声明）。
* `AllowedCorsOrigins`
    * 如果指定，会被默认的 CORS 策略服务实现（驻内存和 EF）使用，用于为 JavaScript 客户端构建一个 CORS 策略。
* `Claims`
    * 允许为客户端设置声明（会被包含在访问令牌中）。
* `AlwaysSendClientClaims`
    * 如果设置了，客户端声明会在每一个流中发送。如果没有，仅用于客户端凭据流（默认为 `false`）
* `PrefixClientClaims`
    * 如果设置，所有的客户端声明将会附上 `client_` 前缀，用于确保不会与用户声明发生冲突。默认为 `true` 。

另外有许多设置用于控制刷新令牌的行为——参见 [这里](../advanced/refreshTokens.html)

## 示例：配置隐式流客户端 (Example: Configure a client for implicit flow)

```csharp
var client = new Client
{
    ClientName = "JS Client",
    Enabled = true,

    ClientId = "implicitclient",
    Flow = Flows.Implicit,

    RequireConsent = true,
    AllowRememberConsent = true,

    RedirectUris = new List<string>
    {
        "https://myapp/callback.html",
    },

    PostLogoutRedirectUris = new List<string>
    {
        "http://localhost:23453/index.html",
    }
}
```

## 示例：配置资源所有者流客户端 (Example: Configure a client for resource owner flow)

```csharp
var client = new Client
{
    ClientName = "Legacy Client",
    Enabled = true,

    ClientId = "legacy",
    ClientSecrets = new List<Secret>
    {
        new Secret("4C701024-0770-4794-B93D-52B5EB6487A0".Sha256())
    },

    Flow = Flows.ResourceOwner,

    AbsoluteRefreshTokenLifetime = 86400,
    SlidingRefreshTokenLifetime = 43200,
    RefreshTokenUsage = TokenUsage.OneTimeOnly,
    RefreshTokenExpiration = TokenExpiration.Sliding
}
```
