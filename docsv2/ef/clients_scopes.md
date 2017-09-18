---
layout: docs-default
---

# Client 和 Scope (Clients and Scopes)

## 仓储 (Stores)

### ClientStore

`ClientStore` 是对 `IClientStore` 接口基于 EF 的实现。它可以独立于 `ScopeStore` 使用。  

### ScopeStore

`ScopeStore` 是对 `IScopeStore` 接口基于 EF 的实现。它可以独立于 `ClientStore` 使用。 

## 注册 (Registration)

使用任意一个仓储，你都需要对其注册。`IdentityServerServiceFactory` 提供了扩展方法来注册这两个仓储。所有的扩展方法都接收一个 `EntityFrameworkServiceOptions` ，它包含以下的属性：

* `ConnectionString`：配置在 `.config` 文件中的连接字符串名称。
* `Schema`：可选的数据库表模式。如果没有提供，则使用数据库默认的表模式。

独立配置仓储的代码如下：

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterClientStore(efConfig);
factory.RegisterScopeStore(efConfig);
``` 

如果两个仓储使用的是同样的 `EntityFrameworkServiceOptions` ，那么可以使用一个便捷的扩展方法：
```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterConfigurationServices(efConfig);
``` 
