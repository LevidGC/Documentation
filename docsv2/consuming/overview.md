---
layout: docs-default
---

# Katana Access Token 验证中间件 (The Katana Access Token Validation Middleware)

在 Web API 中使用 IdentityServer access token 是很简单的——您只需要将我们的验证中间件放入到您的 Katana 管道中并将 URL 设置到 IdentityServer 。所有的配置和验证都已经为您实现了。

您可以在这里获取到中间件：[nuget](https://www.nuget.org/packages/IdentityServer3.AccessTokenValidation/)
或者 [源码](https://github.com/IdentityServer/IdentityServer3.AccessTokenValidation).

高级特性：

* 支持 JWT 和 reference token 的验证
* 支持 scope 的验证
* 内置可配置的 in-memory 缓存用于缓存 reference token 验证结果
* 缓存配置可以被替换

典型的用例是，您提供 IdentityServer 的 URL 和 API 的 scope 名称：

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        // 关闭 JWT 句柄的任何默认映射
        JwtSecurityTokenHandler.InboundClaimTypeMap = new Dictionary<string, string>();

        app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
            {
                Authority = "https://localhost:44333/core",
                RequiredScopes = new[] { "api1" }
            });

        app.UseWebApi(WebApiConfig.Register());
    }
}
```

中间件首先会检查 token ——如果是 JWT ，token 验证会在本地完成（使用在 discovery 文档中找到的签署方名称和 key 材质）。如果 token 是 reference token ，中间件将会使用 IdentityServer 中的 access token 验证 [端点](../endpoints/accessTokenValidation.html)（如果配置了凭据就会使用 [自省端点](../endpoints/introspection.html)）。