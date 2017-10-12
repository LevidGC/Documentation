---
layout: docs-default
---

# 依赖注入 (Dependency Injection)

**此部分示例参见 [这里](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/DependencyInjection)**

IdentityServer3 中多个服务支持扩展。这些服务的默认实现设计时彼此相互分离，因此我们使用了依赖注入来将所有东西整合在一起。

## 注入 IdentityServer 服务 (Injecting IdentityServer services)

如果需要，IdentityServer 提供的默认服务可以在宿主应用中替换掉。自定义的实现同样可以使用依赖注入注入 IdentityServer 类型或者甚至是自定义的类型。这仅在使用 `Registration` 方法来注册的自定义服务和仓储中受支持。

对于接收 IdentityServer 中定义的类型的自定义服务，只需要简单地将这些依赖作为构造器参数就行。当 IdentityServer 实例化你注册的类型时，会自动解析它们。举个例子：

```csharp
public class MyCustomTokenSigningService: ITokenSigningService
{
    private readonly IdentityServerOptions _options;

    public MyCustomTokenSigningService(IdentityServerOptions options)
    {
        _options = options;
    }

    public Task<string> SignTokenAsync(Token token)
    {
        // ...
    }
}
```

注册方式如下：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(typeof(MyCustomTokenSigningService));
```

也可以使用这个语法：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
```

## 注入自定义服务 (Injecting custom services)

您的自定义服务也可以有您自己类型的依赖。您自己的类型也可以被注入进去，只需要您在 IdentityServer 的注入系统中有对应的配置。这是通过  `IdentityServerServiceFactory` 的 `Register()` 方法完成的。举个例子，在你的服务中需要一个自定义的 logger ：

```csharp
public interface ICustomLogger
{
    void Log(string message);
}

public class DebugLogger : ICustomLogger
{
    public void Log(string message)
    {
        Debug.WriteLine(message);
    }
}

public class MyCustomTokenSigningService: ITokenSigningService
{
    private readonly IdentityServerOptions _options;
    private readonly ICustomLogger _logger;

    public MyCustomTokenSigningService(IdentityServerOptions options, ICustomLogger logger)
    {
        _options = options;
        _logger = logger;
    }

    public Task<string> SignTokenAsync(Token token)
    {
        // ...
    }
}
```

然后可以通过如下方式完成注册：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
factory.Register(new Registration<ICustomLogger, MyCustomDebugLogger>());
```

### 没有接口的自定义服务 (Custom services without an interface)

在上面的例子中，注入的类型是 `ICustomLogger` ，其实现为 `MyCustomDebugLogger` 。如果您自定义的服务在设计时没有使用接口将契约与实现分离开，那么具体类型本身也可以注册用于注入。

举个例子，如下，`MyCustomTokenSigningService` 的构造器没有接收 logger 的接口：

```csharp
public class MyCustomTokenSigningService: ITokenSigningService
{
    public MyCustomTokenSigningService(IdentityServerOptions options, MyCustomDebugLogger logger)
    {
        _options = options;
        _logger = logger;
    }

    // ...
}
```

然后注册使用如下方式配置：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
factory.Register(new Registration<MyCustomDebugLogger>());
```

简言之，这种类型的注册方式意味着 `MyCustomDebugLogger` 具体类型是需要被注入的依赖类型。

## 自定义创建 (Customizing the creation)

如果您的服务需要被手动构造（比如，您需要向构造器传递特定参数），然后您需要 `Registration` 类来使用一个工厂回调。这个 `Registration` 的签名如下：

```csharp
new Registration<T>(Func<IDependencyResolver, T> factory) 
```

返回的值必须是 `T` 接口的一个实例，例子如下：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService("SomeSigningKeyValue")
);
```

### 获取其它依赖 (Obtaining other dependencies)

虽然这种方式可以使您最大程度地创建您的服务，但是您仍然需要使用到其它的服务。这就是 `IDependencyResolver` 的用途。它允许您在回调函数中获取服务。举个例子，如果在 IdentityServer 中您的自定义 client 仓储需要其它服务的依赖，它可以通过下面的方式实现：

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService("SomeSigningKeyValue", resolver.Resolve<ICustomLogger>())
);
```

### 命名的依赖 (Named dependencies)

最后，当通过 `IdentityServerServiceFactory` 的 `Register()` 方法注册自定义依赖的时候，依赖可以被命名。这个名称是通过 `Registration` 类中的 `name` 构造器参数指示的。这仅供自定义注册使用，并且名称仅可以在自定义工厂回调中 `IDependencyResolver` 解析依赖的时候使用。

命名的依赖在注册同个 `T` 类型的多个实例时非常有用，同时也提供了一个机制来区分这些实现。举个例子：

```csharp
string mode = "debug"; // or "release", for example

var factory = new IdentityServerServiceFactory();

factory.Register(new Registration<ICustomLogger, MyFileSystemLogger>("debug"));
factory.Register(new Registration<ICustomLogger, MyDatabaseLogger>("release"));

factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService(resolver.Resolve<ICustomLogger>(mode))
);
```

在这个例子中，`mode` 用作一个配置标记，在运行时用于决定使用哪一个 `ICustomLogger` 的实现。

## 使用注册模式实例化 (Instancing with the Registration Mode)

`Registration` 类允许服务指示需要多少个服务的实例将要被创建。`Mode` 属性就是这些可能值的一个枚举：

* `InstancePerHttpRequest`
    每个 HTTP 请求一个实例。这就意味着如果服务在一个依赖链中被请求两次，将会共用同一个实例。
* `InstancePerUse`
    每个地方一个实例。这就意味着如果服务在一个依赖链中请求两次，将会创建两个不同的实例。
* `Singleton`
    仅有一个实例被创建。当使用 `Registration` 时接收一个 singleton 实例作为构造器参数时，这个模式会自动被设置。
