---
layout: docs-default
---

# Entity Framework 对 Client ，Scope 和操作型数据的支持 (Entity Framework support for Clients, Scopes, and Operational Data)

IdentityServer 的 Entity Framework 支持参见 [这个](https://github.com/IdentityServer/IdentityServer3.EntityFramework) 仓储。

## Client 和 Scope 配置 (Client and Scope configuration)

如果 scope 和 client 数据需要从数据库中加载（不是通过 in-memory 配置），那么我们提供了一个基于 `IClientStore` 和 `IScopeStore` 服务的 Entity Framework 实现。[更多信息](clients_scopes.html)。

## 操作型数据 (Operational data)

另外，同样强烈建议将操作型数据（比如，授权码，refresh token ，reference token 和用户同意）持久化到数据库中。因此，我们也对 `IAuthorizationCodeStore` ，`ITokenHandleStore` ，`IRefreshTokenStore` 和 `IConsentStore` 接口提供了Entity Framework 实现。[更多信息](operational.html)。

## 在 SQL Azure 中使用 Entity Framework 迁移 (Using Entity Framework migrations with SQL Azure)

Entity Framework 迁移可以在 SQL Azure 中使用，要这么做需要一个自定义的 `DbConfiguration` 。`DbConfiguration` 负责配置 `DbExecutionStrategy` ，而这需要设置为 `SqlAzureExecutionStrategy` [更多信息](sql_azure.html).
