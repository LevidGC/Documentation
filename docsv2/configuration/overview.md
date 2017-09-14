---
layout: docs-default
---

# 概览 (Overview)

IdentityServer3 是一个中间件包，使用典型的 "Options" 模式来做配置：

```csharp
public void Configuration(IAppBuilder appBuilder)
{
    var options = new IdentityServerOptions
    {
        SigningCertificate = Certificate.Get(),
        Factory = factory,
    };

    appBuilder.UseIdentityServer(options);
}
```

`identityServerOption` 类攘括了 IdentityServer 的所有配置。一部分是由简单的属性构成的，比如 issuer name 或者 site title, 这些数据可以存放在你认为合理的任何地方（代码中的静态部分，配置文件或者数据库）。另一部分就是所谓的服务工厂，作为 IdentityServer 某些特定内部处理过程来注册。

#### IIS 托管和 RAMMFAR (Hosting in IIS and RAMMFAR)

Web 页面相关的文件是作为嵌入式资产存放在 IdentityServer 程序集当中的。当使用 IIS 或者 IIS Express 托管的时候，需要在 `web.config` 中启用 RAMMFAR (_runAllManagedModulesForAllRequests_) ：

```
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true">
  </modules>
</system.webServer>
```

更多 IIS 与自托管相关的例子，请访问 [samples](https://github.com/IdentityServer/IdentityServer3.Samples) 。
