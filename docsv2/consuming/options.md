---
layout: docs-default
---

`IdentityServerBearerTokenAuthenticationOptions` 类有许多选项用于配置 token 验证。

## 常规 (General)

* `Authority` 设置 IdentityServer 的基地址。这可以自动配置 (JWT) 并且访问 token 验证端点 (reference token)。
* `RequiredScopes` 设置一个 **或者** 多个 `scope` 声明，access token 中需要包含。
* `ValidationMode` 可以设置为 `Local` （仅 JWT），`ValidationEndpoint` （JWT 和 reference token 都可以使用验证端点），`Both` （JWT 使用本地验证，reference token 使用验证端点，默认为 `Both`）。
* `TokenProvider` 定义了怎么在 HTTP 请求中获取 token 。默认简单 `Authorization` 报文头。自定义的 token provider 可以用于从可选的报文头或者查询字符串中获取 token 。
* `NameClaimType` 设置 `ClaimsIdentity` 的 name 声明类型（默认为 `name`）。
* `RoleClaimType` 设置 `ClaimsIdentity` 的 role 声明类型（默认为 `role`）。
* `PreserveAccessToken` 如果设为 true ，就会创建一个 `token` 类型的声明用于保存接收到的 access token （默认为 `false`）。当您想要 access token 传递给另一个 API ，比如调用用户信息端点的时候就会起到作用。 
* `BackChannelHttpHandler` 允许为所有的反向通道通讯指定自定义处理器（比如，discovery 端点或者 token 验证端点）。
* `BackchannelCertificateValidator` 允许为反向通道通讯指定一个自定义的证书验证器。
* `DelayLoadMetadata` 告诉中间件在 startup 期间不要加载元数据，而是在有第一个请求的时候加载（默认为 `false`）。如果 discovery 端点在 startup 阶段的时候不可用时就会有用——比如，使用方作为 token 服务托管在同一个进程中。

## JWT 的静态配置 (Static Configuration for JWTs)

除了自动从 discovery 端点获取配置外，你也可以静态地配置中间件。

* `IssuerName` 设置预期的 IdentityServer 颁发方名称
* `SigningCertificate` 设置 X.509 证书用于验证 access token 的签名

**备注** 如果因为一些原因，discovery 文档对你来说不可用，比如 IdentityServer 和客户端或者 API 运行在同一个应用中，这将变得有用。

## 使用自省端点 (Using the Introspection Endpoint) （v2.2 新增）

2.2 版的 IdentityServer 添加了对 token 自省规范的支持。当时用 reference token 的时候这是推荐的（参见 [这里](../endpoints/introspection.html)）。

在这种情况下，你需要指定 `ClientId` 和 `ClientSecret` 来匹配 IdentityServer 中 scope 配置的名称和 secret （参见 [scopes](../configuration/scopesAndClaims.html)）。

## 启用缓存 (Enabling Caching)

当使用 reference token 的时候，你可能不想为每一个请求都向 IdentityServer 发送请求。在这种情况下你可以在本地缓存验证结果。

* `EnableValidationResultCache` 启用或者禁用验证结果的缓存（默认为 `false`）。
* `ValidationResultCacheDuration` 设置缓存持续时间（默认为 5 分钟）。
* `ValidationResultCache` 设置缓存实现。默认为 in-memory 缓存，但是可以通过实现 `IValidationResultCache` 接口实现自定义。
