---
layout: docs-default
---

# CORS

IdentityServer 的许多端点在 JavaScript 中都需要通过 Ajax 调用来访问。由于 IdentityServer 很可能和客户端部署在不同的源上，这就意味着 [Cross-Origin Resource Sharing](http://www.html5rocks.com/en/tutorials/cors/) (CORS) 将会是一个问题。

## Cors Policy Service

IdentityServer3 允许宿主应用实现 `ICorsPolicyService` 来决定 CORS 策略。这个服务注册在 [`IdentityServerServiceFactory`](serviceFactory.html) 。

`ICorsPolicyService` 只有一个方法：

* `Task<bool> IsOriginAllowedAsync(string origin)`
 * 如果允许 `origin` 则返回 `true` ，否则为 `false` 。

你对此也可以创建自定义的实现。

### 已有的实现 (Provided implementations)

IdentityServer 核心部分提供了两个实现：

* `DefaultCorsPolicyService`
    * 如果允许的源的列表是固定的并且在应用启动的时候就是已知的，那么就可以使用这个实现。 `AllowedOrigins` 属性是一个集合，可以通过允许的源的列表来配置。
    * 也有一个 `AllowAll` 属性，如果允许所有的源就可以将其设为 `true` 。
* `InMemoryCorsPolicyService`
    * 这个实现将 `Client` 对象列表作为构造函数的参数。允许的源是通过 `Client` 对象的 `AllowedCorsOrigins` 属性来配置的。
    * 如果使用 `UseInMemoryClients` 扩展方法，那么会自动使用此实现。

最后一个实现是由 IdentityServer3.EntityFramework 提供的：

* `ClientConfigurationCorsPolicyService`
    * 这个实现根据存储在数据库中 `Client` 对象的 `AllowedCorsOrigins` 属性获取允许的源的列表。
    * 如果使用 `RegisterClientStore` 或者 `RegisterConfigurationServices` 扩展方法会自动使用此实现。

