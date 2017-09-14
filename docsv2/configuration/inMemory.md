---
layout: docs-default
---

# 驻内存服务和仓储 (In-Memory Services and Stores)

驻内存服务和仓储可以很简单地让你的测试/开发版 IdentityServer 上线运行。

如果没有针对性地进行配置，我们会为授权码，consent ，引用和刷新令牌提供驻内存版本的仓储。

对于客户端，仓储和用户，你需要提供一个 `Client` ，`Scope` 和 `InMemoryUser` 静态列表。

**这只适用于测试和开发。**

```csharp
var factory = new IdentityServerServiceFactory()
        .UseInMemoryUsers(Users.Get())
        .UseInMemoryClients(Clients.Get())
        .UseInMemoryScopes(Scopes.Get());

var idsrvOptions = new IdentityServerOptions
{
    Factory = factory,
    SigningCertificate = Cert.Load(),
};

app.UseIdentityServer(idsrvOptions);
```
