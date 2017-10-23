---
layout: docs-default
---

# 操作型数据 (Operational Data)

IdentityServer3 的许多特性需要数据库来持久化操作型数据。这包括授权码，refresh token ，reference token 和用户同意。

## 注册 (Registration)

有许多仓储用于持久化操作型数据。这些都是通过 `IdentityServerServiceFactory`。所有的扩展方法都接收一个 `EntityFrameworkServiceOptions` ，它包含以下属性：

* `ConnectionString`：配置在 `.config` 文件中的连接字符串名称。
* `Schema`：可选的数据库表模式名称。如果没有提供，则使用数据库默认值。

以下代码用于配置操作型数据仓储：

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterOperationalServices(efConfig);
``` 

## 数据清理 (Data Cleanup)

大多数操作型数据都有过期时间。所以最好是将这些旧的数据在过期后清除掉。这可以在托管 IdentityServer 的宿主应用之外或者数据库中实现（可以通过多种机制）。如果你想让应用代码帮你执行这样的清理，那么 `TokenCleanup` 类将会帮到你。它接收一个 `EntityFrameworkServiceOptions` 和一个 `Int32` 间隔（秒）用于配置旧的数据被清理的频率。它将会异步地连接数据库，配置如下：

```csharp
var efConfig = new EntityFrameworkServiceOptions {
    ConnectionString = connString,
    //Schema = "foo"
};

var cleanup = new TokenCleanup(efConfig, 10);
cleanup.Start();
``` 
