---
layout: docs-default
---

# IdentityServer 选项 (IdentityServer Options)

 `IdentityServerOptions` [类](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FConfiguration%2FIdentityServerOptions.cs) 是 IdentityServer 所有配置设置的顶层容器。

* `IssuerUri`
    * server 实例的唯一名称，比如：https://myissuer.com 。默认值就是 IdentityServer 安装的 server 的基 URL 。
* `SiteName`
    * 在标准视图中会使用到，用于展示站点的名称。
* `SigningCertificate`
    * X.509 证书（包括对应的私钥），用于签署安全令牌。
* `SecondarySigningCertificate`
    * 出现在 discovery 文档中的二级证书。用于为客户端准备证书翻转 (certificate rollover) 。
* `RequireSsl`
    * 指示 IdentityServer 是否需要 SSL 。默认值为 `true`
* `PublicOrigin`
    * 默认情况下，IdentityServer 会使用 HTTP 请求的 host，protocol 和 port 来创建链接。但是在使用反向代理或者负载均衡的情况下这并不准确。所以可以为链接的生成来重写这个属性。
* `Endpoints`
    * 启用或者禁用相关端点（默认情况下所有端点都是启用的）
* `Factory` (required)
    * 设置 [IdentityServerFactory](serviceFactory.html)
* `DataProtector`
    * 设置自定义数据保护器 (data protector) 。默认为 Katana 宿主数据保护器 (Katana host data protector) 。
* `AuthenticationOptions`
    * 配置 [AuthenticationOptions](authenticationOptions.html)
* `PluginConfiguration`
    * 添加协议插件，比如添加对 WS-Federation 的支持
* `CspOptions`
    * 配置 [CSP](../advanced/csp.html)
* `ProtocolLogoutUrls`
    * 配置登出环节需要访问的回调 URLs （主要用于协议插件）
* `LoggingOptions`
    * 配置与 [logging](logging.html) 相关的设置
* `EventsOptions`
    * 配置与 [events](events.html) 相关的设置
* `EnableWelcomePage`
    * 启用或禁用默认的欢迎页面。默认为 `true`。
