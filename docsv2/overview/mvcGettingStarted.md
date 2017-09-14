---
layout: docs-default
---

本教程将指导您完成必要的步骤来创建一个最基本的 IdentityServer 并使其运行。为了尽量 **简单** 我们将 IdentityServer 和客户端托管在同一个 Web 应用下面——在真实的场景下面并不推荐使用，但是它让你在起步阶段并不会感到太复杂。

完整的代码参见 [这里](https://github.com/identityserver/Thinktecture.IdentityServer3.Samples/tree/master/source/MVC%20Authentication).

# 第一部分——MVC 认证和授权 (Part 1 - MVC Authentication & Authorization)

在第一部分，我们将会创建一个简单的 MVC 应用并通过 IdentityServer 来向其添加认证。然后，你会近距离看一下声明，声明的转换以及授权。

## 创建 Web 应用 (Create the web application)

在 Visual Studio 2013 中，创建一个标准的 MVC 应用，并将认证设置为 "No authentication" 。

![create mvc app](https://cloud.githubusercontent.com/assets/1454075/4604880/16fa22f0-51bd-11e4-96aa-e82206d21e26.png)

现在使用属性窗口将项目切换到 SSL ：

![set ssl](https://cloud.githubusercontent.com/assets/1454075/4604894/9f18b656-51bd-11e4-935c-e3ecdb3d1905.png)

**重要**
在你的项目属性中不要忘记更新启动 URL 。

## 添加 IdentityServer (Adding IdentityServer)

IdentityServer 是基于 OWIN/Katana 的并且通过 Nuget 包进行分发。想要将它添加到刚创建的 Web 宿主中，先安装下面的两个包：

````
install-package Microsoft.Owin.Host.Systemweb
install-package IdentityServer3
````

## 配置 IdentityServer ——客户端 (Configuring IdentityServer - Clients)

IdentityServer 需要一些它将要支持的客户端的信息，这可以简单地使用 `Client` 对象来提供：

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client 
            {
                Enabled = true,
                ClientName = "MVC Client",
                ClientId = "mvc",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "https://localhost:44319/"
                },
                
                AllowAccessToAllScopes = true
            }
        };
    }
}
```

**备注** 到目前为止，客户端可以访问所有的域（通过 `AllowAccessToAllScopes` 设置）。对于生产环境，你应该对此有所限制。

## 配置 IdentityServer ——用户 (Configuring IdentityServer - Users)

接下来我们将要在 IdentityServer 中添加一些用户——同样我们也是通过简单的 C# 类来提供。你可以从任何数据仓储中检索用户的信息并且我们对 ASP.NET Identity 和 MembershipReboot 都提供了开箱即用的支持。

```csharp
public static class Users
{
    public static List<InMemoryUser> Get()
    {
        return new List<InMemoryUser>
        {
            new InMemoryUser
            {
                Username = "bob",
                Password = "secret",
                Subject = "1",

                Claims = new[]
                {
                    new Claim(Constants.ClaimTypes.GivenName, "Bob"),
                    new Claim(Constants.ClaimTypes.FamilyName, "Smith")
                }
            }
        };
    }
}
```

## 添加 Startup (Adding Startup)

IdentityServer 是在 startup 类中配置的。在这里，我们将会提供客户端，用户，域，签名证书和一些其它配置选项的信息。在生产环境中，你应该从 Windows certicate store 或者其它安全的数据源那里加载签名证书。在这个样例中，我们只是简单地将其作为文件添加到项目中（你可以从 [这里](https://github.com/identityserver/Thinktecture.IdentityServer3.Samples/tree/master/source/Certificates) 下载一个测试证书）。将其添加到项目中并将它的 build action 设置为 `Copy to output` 。


想要获取更多关于怎么从 Azure 站点加载证书的信息，参见 [这里](http://azure.microsoft.com/blog/2014/10/27/using-certificates-in-azure-websites-applications/) 。

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.Map("/identity", idsrvApp =>
            {
                idsrvApp.UseIdentityServer(new IdentityServerOptions
                {
                    SiteName = "Embedded IdentityServer",
                    SigningCertificate = LoadCertificate(),

                    Factory = new IdentityServerServiceFactory()
                                .UseInMemoryUsers(Users.Get())
                                .UseInMemoryClients(Clients.Get())
                                .UseInMemoryScopes(StandardScopes.All)
                });
            });
    }

    X509Certificate2 LoadCertificate()
    {
        return new X509Certificate2(
            string.Format(@"{0}\bin\identityServer\idsrv3test.pfx", AppDomain.CurrentDomain.BaseDirectory), "idsrv3test");
    }
}
```

到目前为止，你已经有了一个完整功能的 IdentityServer ，浏览 discovery 端点并查看其配置：

![disco](https://cloud.githubusercontent.com/assets/1454075/4604932/43a3d7da-51c0-11e4-8a88-b74db2b7e771.png)

## RAMMFAR

最后一件事，不要忘记在你的 web.config 中添加 RAMMFAR ，不然，我们的一些嵌入式资产在 IIS 中将不会被正确加载：

```xml
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true" />
</system.webServer>
```

## 添加并配置 OpenID Connect 认证中间件 (Adding and configuring the OpenID Connect authentication middleware)

想要将 OIDC 认证添加到 MVC 应用中，我们需要添加两个包：

````
install-package Microsoft.Owin.Security.Cookies
install-package Microsoft.Owin.Security.OpenIdConnect
````

在 Startup.cs 中使用默认值配置 cookie 中间件：

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions
    {
        AuthenticationType = "Cookies"
    });
```

将 OpenID Connect 中间件（也在 Startup.cs 中）指向我们的嵌入式 IdentityServer ，并使用之前配置的客户端配置：

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectAuthenticationOptions
    {
        Authority = "https://localhost:44319/identity",
        ClientId = "mvc",
        RedirectUri = "https://localhost:44319/",
        ResponseType = "id_token",

        SignInAsAuthenticationType = "Cookies"
    });
```

## 添加一个受保护资源并显示声明 (Adding a protected resource and showing claims)

想要使用 IdentityServer 来初始化认证，你需要创建一个受保护的资源，比如，添加一个全局的授权过滤器。在我们的样例中，我们会简单地保护 `Home` controller 下的 `About` action 。另外，我们会将声明提交给视图，这样我们将会看到 IdentityServer 所发出的声明内容：

````csharp
[Authorize]
public ActionResult About()
{
    return View((User as ClaimsPrincipal).Claims);
}
````

对应的视图如下：

````html
@model IEnumerable<System.Security.Claims.Claim>

<dl>
    @foreach (var claim in Model)
    {
        <dt>@claim.Type</dt>
        <dd>@claim.Value</dd>
    }
</dl>
````

## 认证和声明 (Authentication and claims)

点击 about 链接将会触发认证。IdentityServer 将会显示登录窗口并将一个令牌发送会主窗口。OpenID Connect 中间件会验证这个令牌，提取声明并将它们发送给 cookie 中间件，而它将会设置认证 cookie 。用户现在就已经登录了。


![login](https://cloud.githubusercontent.com/assets/1454075/4604964/7f5d6eb0-51c2-11e4-8016-feac5ed2f67a.png)

![claims](https://cloud.githubusercontent.com/assets/1454075/4604966/91126e8a-51c2-11e4-8fd9-d9c7af1d096b.png)

## 添加角色声明和域 (Adding role claims and scope)

下一步，我们想要给用户添加一些角色声明，在后面的授权中会使用到。

现在，我们已经离开了 OIDC 标准域——让我们定义一个角色域，它包含了角色声明并将其添加到标准域：

```csharp
public static class Scopes
{
    public static IEnumerable<Scope> Get()
    {
        var scopes = new List<Scope>
        {
            new Scope
            {
                Enabled = true,
                Name = "roles",
                Type = ScopeType.Identity,
                Claims = new List<ScopeClaim>
                {
                    new ScopeClaim("role")
                }
            }
        };

        scopes.AddRange(StandardScopes.All);

        return scopes;
    }
}
```

同样更改 `Startup` 的工厂来使用新域：

```csharp
Factory = new IdentityServerServiceFactory()
    .UseInMemoryUsers(Users.Get())
    .UseInMemoryClients(Clients.Get())
    .UseInMemoryScopes(Scopes.Get()),
```

接下来，我们给 Bob 添加一对角色声明：

```csharp
public static class Users
{
    public static IEnumerable<InMemoryUser> Get()
    {
        return new[]
        {
            new InMemoryUser
            {
                Username = "bob",
                Password = "secret",
                Subject = "1",

                Claims = new[]
                {
                    new Claim(Constants.ClaimTypes.GivenName, "Bob"),
                    new Claim(Constants.ClaimTypes.FamilyName, "Smith"),
                    new Claim(Constants.ClaimTypes.Role, "Geek"),
                    new Claim(Constants.ClaimTypes.Role, "Foo")
                }
            }
        };
    }
}
```

## 更改中间件配置来寻要 roles (Changing the middleware configuration to ask for roles)

默认情况下，OIDC 中间件寻要的是两个域：`openid` 和 `profile` ——这就是为什么 IdentityServer 包含了主体和名称声明。现在，我们添加一个对 `roles` 域的请求：

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectAuthenticationOptions
    {
        Authority = "https://localhost:44319/identity",
                    
        ClientId = "mvc",
        Scope = "openid profile roles",
        RedirectUri = "https://localhost:44319/",
        ResponseType = "id_token",

        SignInAsAuthenticationType = "Cookies"
    });
```

成功认证之后，现在你应该在用户声明集合中看到角色声明：

![role claims](https://cloud.githubusercontent.com/assets/1454075/4605904/0397adc2-5203-11e4-9e20-32b1b53c7570.png)

## 声明转换 (Claims transformation)

当你在 about 页面中查看声明，你将会注意到两件事：某些声明有比较长的奇怪的类型名称以及一些你在应用中可能不需要的声明。

长的声明名称来自于微软的 JWT 处理器，用于尝试将一些声明类型映射到 .NET 的 `ClaimTypes` 类型。你可以使用下面的代码（在 `Startup` 中）来关闭这个行为。

这样做你同样需要为防跨站请求伪造 (anti-CSRF) 保护将配置调整为新的唯一的 `sub` 声明类型：

```csharp
AntiForgeryConfig.UniqueClaimTypeIdentifier = Constants.ClaimTypes.Subject;
JwtSecurityTokenHandler.InboundClaimTypeMap = new Dictionary<string, string>();
```

现在声明看起来将会如下：

![shorter claims](https://cloud.githubusercontent.com/assets/1454075/4606129/2fb799fa-5210-11e4-8b30-22a47e9cbeb1.png)

这是一个提升，但是仍然有一些低层的协议声明在典型的业务逻辑中并不需要。那么将原始传入的声明转换为应用相关的声明就叫做声明转换。在处理你的传入的声明期间，决定哪些声明你需要保留可能需要与额外的数据仓储进行通讯来检索你的应用需要用到的声明。

OIDC 中间件有一个通知可以用于做声明转换——结果声明将会存储在 cookie 中：

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectAuthenticationOptions
    {
        Authority = "https://localhost:44319/identity",
                    
        ClientId = "mvc",
        Scope = "openid profile roles",
        RedirectUri = "https://localhost:44319/",
        ResponseType = "id_token",

        SignInAsAuthenticationType = "Cookies",
        UseTokenLifetime = false,

        Notifications = new OpenIdConnectAuthenticationNotifications
        {
            SecurityTokenValidated = n =>
                {
                    var id = n.AuthenticationTicket.Identity;

                    // 我们想要保留名，姓，主体和角色
                    var givenName = id.FindFirst(Constants.ClaimTypes.GivenName);
                    var familyName = id.FindFirst(Constants.ClaimTypes.FamilyName);
                    var sub = id.FindFirst(Constants.ClaimTypes.Subject);
                    var roles = id.FindAll(Constants.ClaimTypes.Role);

                    // 创建新的身份并设置名和角色声明类型
                    var nid = new ClaimsIdentity(
                        id.AuthenticationType,
                        Constants.ClaimTypes.GivenName,
                        Constants.ClaimTypes.Role);

                    nid.AddClaim(givenName);
                    nid.AddClaim(familyName);
                    nid.AddClaim(sub);
                    nid.AddClaims(roles);

                    // 添加一些其它与应用相关的声明
                    nid.AddClaim(new Claim("app_specific", "some data"));                   

                    n.AuthenticationTicket = new AuthenticationTicket(
                        nid,
                        n.AuthenticationTicket.Properties);
                    
                    return Task.FromResult(0);    
                }
        }
    });
```

添加了上面的代码之后，我们的声明将会如下所示：

![transformed claims](https://cloud.githubusercontent.com/assets/1454075/4606148/00033a74-5211-11e4-8cec-ed456a271d6e.png)

## 授权 (Authorization)

现在，我们有了认证和一些声明，那么可以开始添加简单的授权规则了。

MVC 有一个内置的属性叫做 `[Authorize]` ，它需要认证的用户，你也可以使用这个属性来标注角色成员资格需求。我们并不推荐这个方式，因为这通常会导致关注点杂糅，比如业务/controller 逻辑和授权策略。我们更推荐将授权逻辑与 controller 进行分离，这会带来更简洁的代码和更好的可测试性（阅读 [这篇文章](http://leastprivilege.com/2014/06/24/resourceaction-based-authorization-for-owin-and-mvc-and-web-api) 了解更多）

### 资源授权 (Resource Authorization)

添加一个 Nuget 包来增加新的授权基础结构和新属性：

```
install-package Thinktecture.IdentityModel.Owin.ResourceAuthorization.Mvc
```

接下来，我们对 `Home` controller 中的 `Contact` action 进行标注，使用一个能够表达当前执行行为的属性—— `Read` `ContactDetails` 资源：

```csharp
[ResourceAuthorize("Read", "ContactDetails")]
public ActionResult Contact()
{
    ViewBag.Message = "Your contact page.";

    return View();
}
```

注意这个属性 **并不是** 表达谁被允许读取联系人——我们将逻辑拆分进一个授权管理器，它知道 actions ，资源以及在应用中谁被允许做什么样的操作：

```csharp
public class AuthorizationManager : ResourceAuthorizationManager
{
    public override Task<bool> CheckAccessAsync(ResourceAuthorizationContext context)
    {
        switch (context.Resource.First().Value)
        {
            case "ContactDetails":
                return AuthorizeContactDetails(context);
            default:
                return Nok();
        }
    }

    private Task<bool> AuthorizeContactDetails(ResourceAuthorizationContext context)
    {
        switch (context.Action.First().Value)
        {
            case "Read":
                return Eval(context.Principal.HasClaim("role", "Geek"));
            case "Write":
                return Eval(context.Principal.HasClaim("role", "Operator"));
            default:
                return Nok();
        }
    }
}
```

最后，我们在 `Startup` 中将授权管理器插入到 OWIN 管道中：

```csharp
app.UseResourceAuthorization(new AuthorizationManager());
```

运行这个例子并浏览这些代码来使你熟悉这个流。

### 角色授权 (Role Authorization)

然而，如果你选择使用 `[Authorize(Roles = "Foo,Bar")]` ，那么请明白一件事，就是如果当前用户经过了认证，但是用户并不属于 `Authorize` 属性中的任一个角色（MVC 5.2 已证实），那么站点将会进入无线重定向循环。这个不良的结果是因为 `Authorize` 属性在用户是认证的情况下，但是并不属于当中的某个角色会将 action 的结果设置为 401 。这个 401 结果会触发到 IdentityServer 进行认证，而认证会继续将用户重定向回去，然后又开启了新一轮的重定向。这种行为可以通过重写 `Authorize` 属性的 `HandleUnauthorizedRequest` 方法来解决，方法如下，然后使用这个自定义的授权属性来替换 MVC 提供的授权属性。


```csharp
// 修改后的授权属性：
public class AuthAttribute : AuthorizeAttribute
{
    protected override void HandleUnauthorizedRequest(AuthorizationContext filterContext)
    {
        if (filterContext.HttpContext.User.Identity.IsAuthenticated)
        {
            // 403 我们知道你是谁，但是你并没有被许可访问
            filterContext.Result = new HttpStatusCodeResult(System.Net.HttpStatusCode.Forbidden);
        }
        else
        {
            // 401 你是谁？先去登录然后再试一次
            filterContext.Result = new HttpUnauthorizedResult();
        }
    }
}

// 使用：
[Auth(Roles = "Geek")]
public ActionResult About()
{
    // ...
}
```

## 更多关于授权以及处理访问拒绝的场景 (More authorization and dealing with access denied scenarios)

让我们在授权上多做一点事，现在 `Home` controller 中添加一个新的 action 方法：

```csharp
[ResourceAuthorize("Write", "ContactDetails")]
public ActionResult UpdateContact()
{
    ViewBag.Message = "Update your contact details!";

    return View();
}
```

当你尝试导航到 `/home/updatecontact` URL 来调用这个 action 的时候，你将会看到一个 `forbidden` 错误页。

![iis forbidden](https://cloud.githubusercontent.com/assets/1454075/4611482/35584714-52bb-11e4-97f1-12be348905d7.png)

事实上，基于用户是否经过认证会出现不同的响应。如果没有经过认证 MVC 将会重定向到登录页面，如果经过认证，那么将会看到 forbidden 响应。这是设计使然（了解更多，参见 [这里](http://leastprivilege.com/2014/10/02/401-vs-403/)）。

我们可以通过检测 `403` 状态码来处理禁止情形——我们一个开箱即用的过滤器：

```csharp
[ResourceAuthorize("Write", "ContactDetails")]
[HandleForbidden]
public ActionResult UpdateContact()
{
    ViewBag.Message = "Update your contact details!";

    return View();
}
```

`HandleForbidden` 过滤器（当然也可以是全局的）将会在任何时候发出 403 的时候重定向到一个指定的视图——默认我们会查找一个称为 `Forbidden` 视图。

![forbidden](https://cloud.githubusercontent.com/assets/1454075/4611314/0fcaa7aa-52b9-11e4-8a1a-c158a3d89697.png)

你还可以使用授权管理器，这会给你提供更多的选择：

```csharp
[HandleForbidden]
public ActionResult UpdateContact()
{
    if (!HttpContext.CheckAccess("Write", "ContactDetails", "some more data"))
    {
        // 不管 401 还是 403 都基于认证状态
        return this.AccessDenied();
    }

    ViewBag.Message = "Update your contact details!";
    return View();
}
```

## 添加布局 (Adding Logout)

添加布局是很简单的，简单添加一个新的 action 并调用 Katana 认证管理器的 `Signout` 方法：

```csharp
public ActionResult Logout()
{
    Request.GetOwinContext().Authentication.SignOut();
    return Redirect("/");
}
```

这会在 IdentityServer *endsession* 端点间初始化一个往返。这个端点将会清空认证 cookie 并终止你的会话：

![simple logout](https://cloud.githubusercontent.com/assets/1454075/4641154/71d7e23a-5432-11e4-9fb7-94dc8d53e5d0.png)

通常来说，现在做的最安全的事就是简单地关闭浏览器窗口来摆脱所有的会话数据。一些应用也会给用户一个机会返回到匿名用户的状态。

这是可能的，但是需要完成一些步骤——首先你得注册一个合法的 URL 供登出处理过程完成之后来返回。对于 MVC 应用这是在客户端定义中做的（注意这个新的 `PostLogoutRedirectUris` 设置）：

```csharp
new Client 
{
    Enabled = true,
    ClientName = "MVC Client",
    ClientId = "mvc",
    Flow = Flows.Implicit,

    RedirectUris = new List<string>
    {
        "https://localhost:44319/"
    },
    PostLogoutRedirectUris = new List<string>
    {
        "https://localhost:44319/"
    }
}

```

接下来，客户端需要向登出端点证实它的身份来确保我们将会重定向到正确的 URL （而不是一些垃圾/钓鱼页面）。这是通过发送客户端在认证过程中获取到的初始身份令牌实现的。而现在，我们已经丢弃了这个令牌，是时候更改声明转换来保存它了。

在 `SecurityTokenValidated` 通知中添加下面这行代码：

```csharp
// 为登出保存 id_token
nid.AddClaim(new Claim("id_token", n.ProtocolMessage.IdToken));
```

最后一个步骤，在用户登出的时候我们需要附加上 id_token 来向 IdentityServer 发起一个来回。这在 OIDC 中间件中也是通过使用一个通知来完成的。 

```csharp
RedirectToIdentityProvider = n =>
    {
        if (n.ProtocolMessage.RequestType == OpenIdConnectRequestType.LogoutRequest)
        {
            var idTokenHint = n.OwinContext.Authentication.User.FindFirst("id_token");

            if (idTokenHint != null)
            {
                n.ProtocolMessage.IdTokenHint = idTokenHint.Value;
            }
        }

        return Task.FromResult(0);
    }
```

有了这些改变，IdentityServer 将会给用户提供一个链接供用户返回到调用应用：

![logout with redirect](https://cloud.githubusercontent.com/assets/1454075/4641278/b7db4130-5434-11e4-94c3-6f3ff2d5d0ce.png)

**Ti提示p** 在 `IdentityServerOptions` 中你可能发现到一个 `AuthenticationOptions` 对象。它有一个属性叫做 `EnablePostSignOutAutoRedirect` 。正如你想的那样，将这个设置为 `true` 在登出之后会自动将用户重定向回客户端。 

## 添加 Google 认证 (Adding Google Authentication)

接下来我们想要启用外部认证。这需要向 IdentityServer 添加额外的 Katana 认证中间件来实现——在我们的例子中会使用 Google 。

### 使用 Google 注册 IdentityServer (Registering IdentityServer with Google)

首先我们需要在 Google 的开发者控制台中注册 IdentityServer 。这包含了下面的几个步骤：

首先，导航到：

https://console.developers.google.com

**创建一个新项目**

![googlecreateproject](https://cloud.githubusercontent.com/assets/1454075/4843029/d6a3eb68-602c-11e4-83a1-edfceea419e5.png)

**启用 Google+ API**

![googleapis](https://cloud.githubusercontent.com/assets/1454075/4843041/ebb8ed46-602c-11e4-8932-73b48d6a83fc.png)

**使用 email 地址和产品名称来注册同意屏幕 (consent screen)**

![googleconfigureconsent](https://cloud.githubusercontent.com/assets/1454075/4843066/2214d8fa-602d-11e4-8686-f6d6ba6ab6e8.png)

**创建一个客户端应用**

![googlecreateclient](https://cloud.githubusercontent.com/assets/1454075/4843077/44071554-602d-11e4-9214-191168ba425a.png)

在创建完客户端应用之后，开发者控制台将会给你提供一个客户端 id 和一个客户端 secret 。在后面配置 Google 中间件的时候会使用到这两个值。

### 添加 Google 认证中间件 (Adding the Google authentication middleware)

安装下面的包来添加中间件：

`install-package Microsoft.Owin.Security.Google`

### 配置中间件 (Configure the middleware)

向 `Startup` 中添加下面的代码：

```csharp
private void ConfigureIdentityProviders(IAppBuilder app, string signInAsType)
{
    app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions
        {
            AuthenticationType = "Google",
            Caption = "Sign-in with Google",
            SignInAsAuthenticationType = signInAsType,

            ClientId = "...",
            ClientSecret = "..."
        });
}

```

接下来我们将 IdentityServer 选项类指向到这个方法：

```csharp
idsrvApp.UseIdentityServer(new IdentityServerOptions
{
    SiteName = "Embedded IdentityServer",
    SigningCertificate = LoadCertificate(),

    Factory = new IdentityServerServiceFactory()
        .UseInMemoryUsers(Users.Get())
        .UseInMemoryClients(Clients.Get())
        .UseInMemoryScopes(Scopes.Get()),

    AuthenticationOptions = new IdentityServer3.Core.Configuration.AuthenticationOptions
    {
        IdentityProviders = ConfigureIdentityProviders
    }
});
```

就是这样！下一次我们登录的时候——在登录页上面就会出现一个 "Sign-in with Google" ：

![googlesignin](https://cloud.githubusercontent.com/assets/1454075/4843133/46d07194-602e-11e4-9530-c84b6544bfcb.png)

注意当使用 Google 登录的时候会丢失 `role` 声明。 这在情理之中，由于 Google 并没有角色这个概念。要做好准备，并不是所有的身份验证提供商都会提供相同的声明类型。

# 第二部分——添加并调用 Web API (Part 2 - Adding and calling a Web API)

在这部分，我们将会在解决方案中添加一个 Web API 。这个 API 将由 IdentityServer 保护。接下来，我们的 MVC 应用将会调用同时使用信任的子系统和身份委托方式来调用 API 。

## 添加 Web API 项目 (Adding the Web API Project)

创建一个干净的 API 项目最简单的方式就是添加一个空的 Web 项目。

![add empty api](https://cloud.githubusercontent.com/assets/1454075/4625264/e745ebda-5373-11e4-827a-122ad39b7ef0.png)

接下来我们使用 Nuget 来添加 Web API 和 Katana 宿主：

```
install-package Microsoft.Owin.Host.SystemWeb
install-package Microsoft.Aspnet.WebApi.Owin
```

## 添加一个测试 Controller (Adding a Test Controller)

下面的 controller 会将所有的声明返还给调用者——这将允许我们审查发送给 API 的令牌。

```csharp
[Route("identity")]
[Authorize]
public class IdentityController : ApiController
{
    public IHttpActionResult Get()
    {
        var user = User as ClaimsPrincipal;
        var claims = from c in user.Claims
                        select new
                        {
                            type = c.Type,
                            value = c.Value
                        };

        return Json(claims);
    }
}
```

## 在 Startup 中接入 Web API 和安全 (Wiring up Web API and Security in Startup)

基于 Katana 托管的应用，所有的配置总是发生在 `Startup` 中：

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        // Web API 配置
        var config = new HttpConfiguration();
        config.MapHttpAttributeRoutes();

        app.UseWebApi(config);
    }
}
```

除了使用 IdentityServer 来保护我们的 API ——还需要做两件事：

* 仅接收由 IdentityServer 颁发的令牌
* 仅接收为我们 API 颁发的令牌——为此，我们将会给 API 一个 *sampleApi* 名称（也称为 `scope`）

想要实现这个，我们需要添加一个 Nuget 包：

```
install-package IdentityServer3.AccessTokenValidation
```

..并在 `Startup` 中使用它们：

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
        {
            Authority = "https://localhost:44319/identity",
            RequiredScopes = new[] { "sampleApi" }
        });
        
        // Web API 配置
        var config = new HttpConfiguration();
        config.MapHttpAttributeRoutes();

        app.UseWebApi(config);
    }
}
```

**注意**

IdentityServer 颁发标准的 JSON Web Tokens (JWT) ，你可以使用简单的 Katana JWT 中间件来验证它们。我们的中间件很便捷是由于它可以使用 IdentityServer discovery 文档 (metadata) 来自动完成配置。

## 在 IdentityServer 中注册 API (Registering the API in IdentityServer)

接下来我们需要注册 API ——通过扩展域来完成。这一次，我们会添加一个所谓的资源域：

```csharp
public static class Scopes
{
    public static IEnumerable<Scope> Get()
    {
        var scopes = new List<Scope>
        {
            new Scope
            {
                Enabled = true,
                Name = "roles",
                Type = ScopeType.Identity,
                Claims = new List<ScopeClaim>
                {
                    new ScopeClaim("role")
                }
            },
            new Scope
            {
                Enabled = true,
                DisplayName = "Sample API",
                Name = "sampleApi",
                Description = "Access to a sample API",
                Type = ScopeType.Resource
            }
        };

        scopes.AddRange(StandardScopes.All);

        return scopes;
    }
}
```

## 注册一个 Web API 客户端 (Registering a Web API Client)

接下来我们将要调用 API 。你既可以使用客户端凭据（认为是服务账号）也可以代表用户身份来完成。

我们先使用客户端凭据。

首先，我们需要为 MVC 应用注册一个新的客户端。为了安全考虑，IdentityServer 只允许每个客户端采用一种流方式，由于我们现在的 MVC 客户端已经使用了隐式流，我们需要为服务创建一个新的客户端来进行服务通信。

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client 
            {
                ClientName = "MVC Client",
                ClientId = "mvc",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "https://localhost:44319/"
                },
                PostLogoutRedirectUris = new List<string>
                {
                    "https://localhost:44319/"
                },
                AllowedScopes = new List<string>
                {
                    "openid",
                    "profile",
                    "roles",
                    "sampleApi"
                }
            },
            new Client
            {
                ClientName = "MVC Client (service communication)",   
                ClientId = "mvc_service",
                Flow = Flows.ClientCredentials,

                ClientSecrets = new List<Secret>
                {
                    new Secret("secret".Sha256())
                },
                AllowedScopes = new List<string>
                {
                    "sampleApi"
                }
            }
        };
    }
}
```

**备注** 上面的代码片段使用 `AllowedScopes` 设置锁定了客户端可以访问的域的范围。

## 调用 API (Calling the API)

调用 API 包含两个部分：

* 使用客户端凭据从 IdentityServer 为 API 请求一个令牌
* 使用这个访问令牌来调用 API

要想在 OAuth2 令牌端点间的交互更加简单，可以通过 Nuget 包向 MVC 应用添加客户端包：

```
install-package IdentityModel
```

在 `Controller` 目录下，添加一个新的 `CallApiController` 类。下面的代码片段使用客户端凭据为 *sampleApi* 请求令牌：

```csharp
private async Task<TokenResponse> GetTokenAsync()
{
    var client = new TokenClient(
        "https://localhost:44319/identity/connect/token",
        "mvc_service",
        "secret");

    return await client.RequestClientCredentialsAsync("sampleApi");
}
```

下面的代码片段就是使用获取到的访问令牌来访问我们的身份端点：

```csharp
private async Task<string> CallApi(string token)
{
    var client = new HttpClient();
    client.SetBearerToken(token);

    var json = await client.GetStringAsync("https://localhost:44321/identity");
    return JArray.Parse(json).ToString();
}
```

将所有的放在一起，一个新添加的 controller 用来访问服务并在视图中展示声明结果：

```csharp
public class CallApiController : Controller
{
    // GET: CallApi/ClientCredentials
    public async Task<ActionResult> ClientCredentials()
    {
        var response = await GetTokenAsync();
        var result = await CallApi(response.AccessToken);

        ViewBag.Json = result;
        return View("ShowApiResult");
    }

    // 忽略掉的助手方法
}
```

创建 `ShowApiResult.cshtml` 文件，一个简单的视图来展示结果：

````html
<h2>Result</h2>

<pre>@ViewBag.Json</pre>
````

结果如下：

![callapiclientcreds](https://cloud.githubusercontent.com/assets/1454075/4625853/2ba6d94a-537b-11e4-9ad0-3be144a913f0.png)

也就是说——API 对访问者有如下了解：

* 颁发者名称，audience 和过期时间（用于令牌验证中间件）
* 令牌中所颁发的域（用于域验证中间件）
* 客户端 id

所有的声明将会转换为一个 ClaimsPrincipal ，在 controller 中可以通过 *.User* 属性获取到。

## 代表用户访问 API (Calling the API on behalf of the User)

接下来我们想要使用用户的身份来访问 API 。可以通过在 OpenID Connect 中间件配置中将 `sampleApi` 域添加到域列表中实现。我们也需要通过更改相应类型来指示想要请求一个访问令牌：

```csharp
Scope = "openid profile roles sampleApi",
ResponseType = "id_token token"
```

当发出 `token` 的响应类型的请求，IdentityServer 就不会在身份令牌中包含声明了。这是处于优化的目的，由于你现在已经有了访问令牌，它允许你可以从 userinfo 端点获取声明，因此使得身份令牌尽可能小。

访问 userinfo 端点并不难——`UserInfoClient` 类甚至使这个工作更加简单。另外，我们也需要在 cookie 中存储访问令牌，所以，当我们想要代表用户访问 API 的时候就会使用到它：

```csharp
SecurityTokenValidated = async n =>
    {
        var nid = new ClaimsIdentity(
            n.AuthenticationTicket.Identity.AuthenticationType,
            Constants.ClaimTypes.GivenName,
            Constants.ClaimTypes.Role);

        // 获取 userinfo 数据
        var userInfoClient = new UserInfoClient(
            new Uri(n.Options.Authority + "/connect/userinfo"),
            n.ProtocolMessage.AccessToken);

        var userInfo = await userInfoClient.GetAsync();
        userInfo.Claims.ToList().ForEach(ui => nid.AddClaim(new Claim(ui.Item1, ui.Item2)));

        // 为登出保存 id_token
        nid.AddClaim(new Claim("id_token", n.ProtocolMessage.IdToken));

        // 为示例 API 添加访问令牌
        nid.AddClaim(new Claim("access_token", n.ProtocolMessage.AccessToken));

        // 跟踪访问令牌过期时间
        nid.AddClaim(new Claim("expires_at", DateTimeOffset.Now.AddSeconds(int.Parse(n.ProtocolMessage.ExpiresIn)).ToString()));

        // 添加其它与应用相关的声明
        nid.AddClaim(new Claim("app_specific", "some data"));

        n.AuthenticationTicket = new AuthenticationTicket(
            nid,
            n.AuthenticationTicket.Properties);
    }
```

另一个选线就是在 IdentityServer 中重新配置域并且在域声明中设置 `AlwaysIncludeInIdToken` 标志来强制在身份令牌中包含声明——这是留给读者的一个练习。

**访问 API (Calling the API)**

由于现在访问令牌已经存储在 cookie 中了，我们现在可以简单地从声明主体中检索它，并使用它来访问服务：

```csharp
// GET: CallApi/UserCredentials
public async Task<ActionResult> UserCredentials()
{
    var user = User as ClaimsPrincipal;
    var token = user.FindFirst("access_token").Value;
    var result = await CallApi(token);

    ViewBag.Json = result;
    return View("ShowApiResult");
}
```

登录之后，你就会在结果页面中看到已经包含了 `sub` 声明，这意味着 API 已经代表用户了：

![userdelegation](https://cloud.githubusercontent.com/assets/1454075/5453086/246392fc-8523-11e4-9a3f-8100af390d53.png)

如果你现在在 `sampleApi` 域中添加一个 `role` 域声明——用户的角色同样也会被包含在访问令牌中：

```csharp
new Scope
{
    Enabled = true,
    DisplayName = "Sample API",
    Name = "sampleApi",
    Description = "Access to a sample API",
    Type = ScopeType.Resource,

    Claims = new List<ScopeClaim>
    {
        new ScopeClaim("role")
    }
}
```

![delegationroles](https://cloud.githubusercontent.com/assets/1454075/5453110/9ca835ec-8523-11e4-874f-771b648b7016.png)
