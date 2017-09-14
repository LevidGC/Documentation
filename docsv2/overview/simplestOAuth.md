---
layout: docs-default
---

# 创建最简单的 OAuth2 授权服务器，客户端和 API (Creating the simplest OAuth2 Authorization Server, Client and API)

此演示的目的是创建一个最简单的 IdentityServer ，并充当为 OAuth2 授权服务器。这是为了让你熟悉一些基本的特性和配置选项（完整的代码参见 [这里](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Simplest%20OAuth2%20Walkthrough)）。本文档中也有一些高级教程演示。教程如下：

* 创建自托管的 IdentityServer

* 为应用服务创建客户端（既可以使用应用账户也可以代表用户来通讯）

* 注册 API

* 请求访问令牌

* 调用 API

* 验证访问令牌

## 创建 IdentityServer (Setting up IdentityServer)

首先我们将要创建一个控制台宿主并建立一个 IdentityServer 。

先创建一个标准的控制台应用程序，然后通过 nuget 来添加 IdentityServer ：

```
install-package identityserver3
```

### 注册 API (Registering the API)

API 都建模为域 (scope) ——你需要注册所有的 API ，这样你就可以使用访问令牌来发送请求。因此，我们创建一个类来返回一个 `Scope` 列表：

```csharp
using IdentityServer3.Core.Models;

static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            new Scope
            {
                Name = "api1"
            }
        };
    }
}
```

### 注册客户端 (Registering the Client)

现在我们想要注册一个单独的客户端。这个客户端能够请求 `api1` 域的令牌。在我们的第一次迭代中，是没有任何人参与的，客户端是代表它们自己来简单请求令牌的（想象一下机器与机器之间的通信）。后期我们会在当中加入用户。


在这个客户端中，我们配置如下的信息：

* 展示名称和 id （名称唯一）

* 客户端 secret （结合令牌端点可用于认证客户端）

* 流 （本例中使用的是客户端凭据流）

* 所谓的参考令牌的使用。参考令牌不需要签名证书

* `api1` 域的访问

```csharp
using IdentityServer3.Core.Models;

static class Clients
{
    public static List<Client> Get()
    {
        return new List<Client>
        {
           // 没有人涉及
            new Client
            {
                ClientName = "Silicon-only Client",
                ClientId = "silicon",
                Enabled = true,
                AccessTokenType = AccessTokenType.Reference,

                Flow = Flows.ClientCredentials,

                ClientSecrets = new List<Secret>
                {
                    new Secret("F621F470-9731-4A25-80EF-67A6F7C5F4B8".Sha256())
                },

                AllowedScopes = new List<string>
                {
                    "api1"
                }
            }
        };
    }
}
```

### 配置 IdentityServer (Configuring IdentityServer)

IdentityServer 是作为 OWIN 中间件来实现的。它是在 `Startup` 类中通过 `UseIdentityServer` 扩展方法来配置的。下面的片段使用我们的域和客户端建立了一个服务器骨架。我们同样设置了一个空的用户列表——我们后期会添加用户。

```csharp
using Owin;
using System.Collections.Generic;
using IdentityServer3.Core.Configuration;
using IdentityServer3.Core.Services.InMemory;

namespace IdSrv
{
    class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            var options = new IdentityServerOptions
            {
                Factory = new IdentityServerServiceFactory()
                            .UseInMemoryClients(Clients.Get())
                            .UseInMemoryScopes(Scopes.Get())
                            .UseInMemoryUsers(new List<InMemoryUser>()),
                            
                RequireSsl = false
            };

            app.UseIdentityServer(options);
        }
    }
}
```

### 添加日志 (Adding Logging)

由于我们是在控制台中运行的，那么将日志输出到控制台窗口就非常方便。Serilog 就是一个非常不错的日志类库：

```
install-package serilog -Version 1.5.14
install-package serilog.sinks.literate -Version 1.2.0
```

### 托管 IdentityServer (Hosting IdentityServer)

最后的一步就是托管 IdentityServer 。因此我们需要将 Katana 自托管包添加到我们的控制台应用中：

```
install-package Microsoft.Owin.SelfHost
```

将下面的代码添加到 `Program.cs` ：

```csharp
// 记录日志
Log.Logger = new LoggerConfiguration()
    .WriteTo
    .LiterateConsole(outputTemplate: "{Timestamp:HH:mm} [{Level}] ({Name:l}){NewLine} {Message}{NewLine}{Exception}")
    .CreateLogger();

// 托管 Identityserver
using (WebApp.Start<Startup>("http://localhost:5000"))
{
    Console.WriteLine("server running...");
    Console.ReadLine();
}
```

当你运行这个控制台应用，你应该会看到一些诊断输出以及 `server running...` 。

## 添加 API (Adding an API)

在这个部分，我们会添加一个简单的 Web API ，并且配置它需要我们刚刚在 IdentityServer 中创建的访问令牌。

### 创建 Web Host (Creating the Web Host)

在解决方案中添加一个新的 `ASP.NET Web Application` 并选择 `Empty` 选项（没有框架引用）。

添加必要的 nuget 包：

```
install-package Microsoft.Owin.Host.SystemWeb
install-package Microsoft.AspNet.WebApi.Owin
install-package IdentityServer3.AccessTokenValidation
```

### 添加 Controller (Adding a Controller)

添加这个简单的测试 controller ：

```csharp
[Route("test")]
public class TestController : ApiController
{
    public IHttpActionResult Get()
    {
        var caller = User as ClaimsPrincipal;

        return Json(new
        {
            message = "OK computer",
            client =  caller.FindFirst("client_id").Value
        });
    }
}
```

controller 的 `User` 属性使你可以从访问令牌中获取到相关的声明。

### 添加 Startup (Adding Startup)

添加下面的 `Startup` 类，它既用于建立 Web API ，也用于配置可信的 IdentityServer

```csharp
using Microsoft.Owin;
using Owin;
using System.Web.Http;
using IdentityServer3.AccessTokenValidation;

[assembly: OwinStartup(typeof(Apis.Startup))]

namespace Apis
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            // 从 IdentityServer 接收访问令牌并需要一个 `api1` 域
            app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
                {
                    Authority = "http://localhost:5000",
                    ValidationMode = ValidationMode.ValidationEndpoint,

                    RequiredScopes = new[] { "api1" }
                });

            // 配置 Web API
            var config = new HttpConfiguration();
            config.MapHttpAttributeRoutes();
            
            // 所有 controller 需要认证
            config.Filters.Add(new AuthorizeAttribute());

            app.UseWebApi(config);
        }
    }
}
```

尝试打开浏览器并访问这个测试 controller ——你应该会看到一个 401 ，这是因为请求中没有包含必要的访问令牌。

## 添加控制台客户端 (Adding a Console Client)

在接下来的这部分，我们将会添加一个简单的控制台客户端，它将会请求访问令牌并使用它在请求 API 的时候用于认证。

首先，添加一个新的控制台项目并添加一个 nuget 包用于 OAuth2 客户端助手类：

```
install-package IdentityModel
```

第一个代码片段使用客户端凭据来请求访问令牌：

```csharp
using IdentityModel.Client;

static TokenResponse GetClientToken()
{
    var client = new TokenClient(
        "http://localhost:5000/connect/token",
        "silicon",
        "F621F470-9731-4A25-80EF-67A6F7C5F4B8");

    return client.RequestClientCredentialsAsync("api1").Result;
}
```

第二个代码片段就是使用访问令牌来访问 API ：

```csharp
static void CallApi(TokenResponse response)
{
    var client = new HttpClient();
    client.SetBearerToken(response.AccessToken);

    Console.WriteLine(client.GetStringAsync("http://localhost:14869/test").Result);
}
```

如果你执行这两个代码片段，你将会在控制台中看到 `{"message":"OK computer","client":"silicon"}` 。

## 添加用户 (Adding a User)

到目前为止，客户端是代表它自己来请求访问令牌的，中间并没有人参与进来。现在我们引入用户。

### 添加用户服务 (Adding a user service)

用户服务是用于管理用户的——在这个例子中，我们使用的是驻内存用户服务。

首先，我们需要定义一些用户：

```csharp
using IdentityServer3.Core.Services.InMemory;

static class Users
{
    public static List<InMemoryUser> Get()
    {
        return new List<InMemoryUser>
        {
            new InMemoryUser
            {
                Username = "bob",
                Password = "secret",
                Subject = "1"
            },
            new InMemoryUser
            {
                Username = "alice",
                Password = "secret",
                Subject = "2"
            }
        };
    }
}
```

`Username` 和 `Password` 是用于认证用户的，`Subject` 是嵌入到访问令牌中的唯一标识符。

在 `Startup` 中，将空的用户列表替换为对 `Get` 方法的调用。

### 添加客户端 (Adding a Client)

接下来，我们将要添加一个客户端定义，使用的流为 `资源所有者密码凭据许可` 。这个流允许客户端向令牌服务发送用户的用户名和密码，以此获取访问令牌。

最终，`Clients` 类应该如下：

```csharp
using IdentityServer3.Core.Models;
using System.Collections.Generic;

namespace IdSrv
{
    static class Clients
    {
        public static List<Client> Get()
        {
            return new List<Client>
            {
                // 没有人涉及
                new Client
                {
                    ClientName = "Silicon-only Client",
                    ClientId = "silicon",
                    Enabled = true,
                    AccessTokenType = AccessTokenType.Reference,

                    Flow = Flows.ClientCredentials,

                    ClientSecrets = new List<Secret>
                    {
                        new Secret("F621F470-9731-4A25-80EF-67A6F7C5F4B8".Sha256())
                    },

                    AllowedScopes = new List<string>
                    {
                        "api1"
                    }
                },

                // 有人涉及
                new Client
                {
                    ClientName = "Silicon on behalf of Carbon Client",
                    ClientId = "carbon",
                    Enabled = true,
                    AccessTokenType = AccessTokenType.Reference,

                    Flow = Flows.ResourceOwner,

                    ClientSecrets = new List<Secret>
                    {
                        new Secret("21B5F798-BE55-42BC-8AA8-0025B903DC3B".Sha256())
                    },

                    AllowedScopes = new List<string>
                    {
                        "api1"
                    }
                }
            };
        }
    }
}
```

### 更新 API (Updating the API)

当有人参与其中，访问令牌就会包含 `sub` 声明，用于唯一标识用户。

现在让我们对 API controller 做一些修改：

```csharp
[Route("test")]
public class TestController : ApiController
{
    public IHttpActionResult Get()
    {
        var caller = User as ClaimsPrincipal;

        var subjectClaim = caller.FindFirst("sub");
        if (subjectClaim != null)
        {
            return Json(new
            {
                message = "OK user",
                client = caller.FindFirst("client_id").Value,
                subject = subjectClaim.Value
            });
        }
        else
        {
            return Json(new
            {
                message = "OK computer",
                client = caller.FindFirst("client_id").Value
            });
        }
    }
}
```

### 更新客户端 (Updating the Client)

接下来在客户端中添加一个新方法，用于代表用户来请求访问令牌：

```csharp
static TokenResponse GetUserToken()
{
    var client = new TokenClient(
        "http://localhost:5000/connect/token",
        "carbon",
        "21B5F798-BE55-42BC-8AA8-0025B903DC3B");

    return client.RequestResourceOwnerPasswordAsync("bob", "secret", "api1").Result;
}
```

现在尝试使用两种方法来请求同一个令牌，并查看一下 API 响应中的声明。

## 接下来做什么 (What to do next)

这个演示只覆盖一个非常简单的 OAuth2 场景。接下来你应该尝试：

* 其它的流——比如，隐式流，编码或者混合流。它们都是高级场景的推动者，比如联合 (federation) 和外部身份。

* 连接到你的用户数据库——既可以编写你自己的用户服务，也可以使用对 ASP.NET Identity 和 MembershipReboot 开箱即用的支持。

* 在数据仓储中存储客户端和域的配置。我们对 Entity Framework 有开箱即用的支持。

* 使用 OpenID Connect 和身份域添加认证和身份令牌。

**许多技术在 [MVC 演示](mvcGettingStarted.html) 中都有使用，想要了解更多，参见此演示**
