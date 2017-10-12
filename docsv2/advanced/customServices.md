---
layout: docs-default
---

# 自定义服务 (Custom Services)

IdentityServer3 为数据的存储，验证逻辑和支撑 IdentityServer 操作一些必要的通用功能提供了许多可扩展性点。这些可扩展点被统称为“服务”。

参见 [这里](../configuration/serviceFactory.html) 获取完整的服务列表。

## 强制性服务 (Mandatory services)

有三种服务是强制的，并且需要通过实现者来完成相关配置，它们是：

* User 服务 (`IUserService`)

* Client 服务 (`IClientStore`)

* Scope 服务 (`IScopeStore`)

我们为这三个服务提供了简单的 in-memory 版本，同样也支持社区版相关的数据仓储。
参见 [这里](../configuration/inMemory.html) 获取更多详情。

## 注册自定义服务 (Registering custom Services)

您可以替换每一个服务并注册其它自定义的服务。这是通过 `Registration` 类封装的。`Registration` 表示的就是 IdentityServer 获取您的服务实例的方式。

鉴于您服务的设计，可能在每次请求的时候都会有一个新的实例，使用 singleton （单例），或者在每次创建实例的时候需要特殊的初始化逻辑。为了适应不同的情况，`Registration` 类提供了多种构造器供注册您的服务使用：

* `new Registration<T>(Type yourImplementation)`
    * 将 `yourImplementation` 作为实现了 `T` 接口的类来注册。
* `new Registration<T, Impl>()`
    * 将 `Impl` 作为实现了 `T` 接口的类来注册。这个接口是上一个的便捷版本。
* `new Registration<T>(T singleton)`
    * 将 `singleton` 实例作为 `T` 接口的 singleton 实现来注册。
* `new Registration<T>(Func<IDependencyResolver, T> factory)`
    * 注册一个回调函数用于返回 `T` 接口的一个实现。

```csharp
var factory = new IdentityServerServiceFactory();
factory.UserService = new Registration<IUserService, MyCustomUserService>();
```

参见 [依赖注入 (DI)](di.html) 页面获取更多详情。

### 服务清理 (Service cleanup)

除 Singleton 外，如果您的服务实现了 `IDisposable` 接口，那么在每次 HTTP 请求的结尾都会调用 `Dispose` 方法。

