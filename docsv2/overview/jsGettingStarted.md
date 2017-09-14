---
layout: docs-default
---

本教程带你过一遍在 JS 应用中集成 IdentityServer 所要的必需步骤。

由于所有的步骤都在客户端完成，我们需要一个 JS 类库，[oidc-client-js](https://github.com/IdentityModel/oidc-client-js), 来帮助我们完成类似获取和验证令牌的过程。

你可以在 [这里](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/JavaScript%20Walkthrough) 找到与本教程相关联的代码。

本教程被分为以下三个部分：

 - 在 JS 应用中结合 IdentityServer 进行认证
 - 从 JS 应用中发起 API 调用
 - 了解怎么更新令牌，登出以及检查会话。

# 第一部分——结合 IdentityServer 进行认证 (Part 1 - Authentication against IdentityServer)

第一部分关注于怎么在 JS 应用中进行认证。想要完成这一部分，我们将会创建两个项目；一个是 JS 应用，另一个是 IdentityServer。

## 创建 JS 应用项目 (Create the JS application project)

在 Visual Studio 中创建一个空 Web 项目。

![create js app](https://cloud.githubusercontent.com/assets/6102639/12247759/9563a71a-b909-11e5-9823-6ba598b74bad.png)

注意项目设置的 URL：

![js app url](https://cloud.githubusercontent.com/assets/6102639/12252652/4324b3b2-b92d-11e5-9641-772efe43c8e8.png)

## 创建 IdentityServer 项目 (Create the IdentityServer project)

在 Visual Studio 中，为 IdentityServer 创建一个空的 Web 应用。

![create web app](https://cloud.githubusercontent.com/assets/6102639/12247585/bd0e33e4-b908-11e5-90bb-b6f3e9b2c060.png)

现在，你可以使用属性窗口将项目切换到 SSL 。

![set ssl](https://cloud.githubusercontent.com/assets/6102639/12252653/43288fc8-b92d-11e5-93eb-25821a64ceb2.png)

**重要**

不要忘记在项目属性中更新起始 URL ，它可以反应项目的 HTTPS URL 。

## 添加 IdentityServer (Adding IdentityServer)

IdentityServer 是基于 OWIN/Katana 的并且作为 Nuget 包发布。想要将其添加到刚刚创建的 Web 宿主中，安装如下的两个包：

````
Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName IdentityServer
Install-Package IdentityServer3 -ProjectName IdentityServer
````

## 配置 IdentityServer ——客户端 (Configuring IdentityServer - Clients)

IdentityServer 需要知道它将要支持的客户端的一些信息，这很容易实现，只需要提供一个 `Client` 对象的集合：

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
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true
            }
        };
    }
}
```

这里有一个特殊的设置就是 `AllowedCorsOrigins` 属性。这使得 IdentityServer 仅接受来自注册的 URL 发送来的基于浏览器请求。更多详情将会在后面的 `popup.html` 有所了解。

**备注** 现在客户端可以访问所有的域（通过设置 `AllowAccessToAllScopes`）。对于产品应用，你应该通过 `AllowedScopes` 属性来缩小它所期望的域。

## 配置 IdentityServer ——用户 (Configuring IdentityServer - Users)

接下来，我们将要在 IdentityServer 中添加一些用户——同样，这次也是通过提供一个简单的 C# 类来完成。你可以从任何用户仓储中检索用户信息并且我们为 ASP.NET Identity 和 MembershipReboot 提供了开箱即用的支持。

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
                    new Claim(Constants.ClaimTypes.FamilyName, "Smith"),
                    new Claim(Constants.ClaimTypes.Email, "bob.smith@email.com")
                }
            }
        };
    }
}
```

## 配置 IdentityServer ——域 (Configuring IdentityServer - Scopes)

最后，我们需要在 IdentityServer 中添加一些域。为了认证的目的，我们仅需要放置一些标准的 OIDC 域。当我们集成 API 调用的时候，会创建供我们自己使用的域。

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile
        };
    }
}
```

## 添加 Startup (Adding Startup)

IdentityServer 是在 Startup 类中进行配置的。在这里，我么会提供客户端，用户，域，签名证书以及一些其它配置选项的信息。

在生产环境中，你应该从 Windows certificate store 或者其它安全的源那里加载签名证书。而在这个样例中，我们只需要简单地将其作为文件添加到项目中（你可以从 [这里](https://github.com/identityserver/Thinktecture.IdentityServer3.Samples/tree/master/source/Certificates) 下载一个测试证书）。将它添加到项目中，并设置它的 `Copy to Output Directory` 属性为 `Copy always` 。

想要获取怎么从 Azure Website 加载证书的信息，参见 [这里](http://azure.microsoft.com/blog/2014/10/27/using-certificates-in-azure-websites-applications/) 。

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseIdentityServer(new IdentityServerOptions
        {
            SiteName = "Embedded IdentityServer",
            SigningCertificate = LoadCertificate(),

            Factory = new IdentityServerServiceFactory()
                .UseInMemoryUsers(Users.Get())
                .UseInMemoryClients(Clients.Get())
                .UseInMemoryScopes(Scopes.Get())
        });
    }

    private static X509Certificate2 LoadCertificate()
    {
        return new X509Certificate2(
            Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"bin\Config\idsrv3test.pfx"), "idsrv3test");
    }
}
```

到目前为止，你有了一个完整功能的 IdentityServer 并且你可以浏览 Discovery 端点来查看相关配置：

![disco](https://cloud.githubusercontent.com/assets/6102639/12252651/431f61f0-b92d-11e5-9ca8-a49c3db5ea52.png)

## RAMMFAR

最后一件事，不要忘记在 web.config 中添加 RAMMFAR ，不然的话，我们的一些嵌入式资产将不会在 IIS 中被正确地加载：

```xml
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true" />
</system.webServer>
```

## JS 客户端——设置 (JS Client - setup)

我们会使用几个第三方类库来支持我们的应用程序：

 - [jQuery](http://jquery.com)
 - [Bootstrap](http://getbootstrap.com)
 - [oidc-client-js](https://github.com/IdentityModel/oidc-client-js)

我们将使用 [npm](https://www.npmjs.com/) 来安装这些类库，它是 Node.js 的前端包管理器。如果你还没有安装 npm ，你可以参考这个文档 [these instructions on the npm website](https://docs.npmjs.com/getting-started/installing-node) 。一旦安装完成 npm ， 在 `JsApplication` 目录下打开一个命令行提示符，并键入以下命令：

```sh
$ npm install jquery
$ npm install bootstrap
$ npm install oidc-client
```

默认情况，npm 会在 `node_modules` 目录下安装这些包。

**重要** npm 包通常是不加入到源代码控制中的。如果你克隆包含最终源代码的仓储，然后想要恢复包的话，只需要在 `JsApplication` 目录的命令行提示符中输入 `npm install` 即可恢复包。

我们也创建一个基础的 `index.html` 文件：

```html
<!DOCTYPE html>
<html>
<head>
    <title>JS Application</title>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.css" />
    <style>
        .main-container {
            padding-top: 70px;
        }

        pre:empty {
            display: none;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <a class="navbar-brand" href="#">JS Application</a>
            </div>
        </div>
    </nav>

    <div class="container main-container">
        <div class="row">
            <div class="col-xs-12">
                <ul class="list-inline list-unstyled requests">
                    <li><a href="index.html" class="btn btn-primary">Home</a></li>
                    <li><button type="button" class="btn btn-default js-login">Login</button></li>
                </ul>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-12">
                <div class="panel panel-default">
                    <div class="panel-heading">ID Token Contents</div>
                    <div class="panel-body">
                        <pre class="js-user"></pre>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="node_modules/jquery/dist/jquery.js"></script>
    <script src="node_modules/bootstrap/dist/js/bootstrap.js"></script>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
</body>
</html>
```

和一个 `popup.html` 文件：

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
</head>
<body>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
</body>
</html>
```

我们有了两个 HTML 文件，因为 `oidc-client` 会打开一个弹窗为用户展示登录表单。

## JS 客户端——认证 (JS Client - authentication)

现在，有我们所需要的一切东西了，多亏了有 `UserManager` JS 类，我们可以在 `index.html` 页面中配置登录设置。

```js
// 向用户展示数据的辅助函数
function display(selector, data) {
    if (data && typeof data === 'string') {
        data = JSON.parse(data);
    }
    if (data) {
        data = JSON.stringify(data, null, 2);
    }

    $(selector).text(data);
}

var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    response_type: 'id_token',
    scope: 'openid profile',

    filterProtocolClaims: true
};

var manager = new Oidc.UserManager(settings);
var user;

manager.events.addUserLoaded(function (loadedUser) {
    user = loadedUser;
    display('.js-user', user);
});

$('.js-login').on('click', function () {
    manager
        .signinPopup()
        .catch(function (error) {
            console.error('error while logging in through the popup', error);
        });
});
```

我们现在快速过一遍这个设置：

 - `authority` 是 IdentityServer 的基 URL 。这将会允许 `oidc-client` 查询元素据端点，然后可以验证令牌
 - `client_id` 是客户端的 id ，在授权端点会用得到
 - `popup_redirect_uri` 是重定向 URL ，当使用 `signinPopup` 方法的时候会使用得到。如果你不倾向于使用一个弹窗而想要在主窗口来执行重定向，你可以使用 `redirect_uri` 属性和 `signinRedirect` 方法
 - `response_type` 定义令牌类型，在我们的例子中，我们期望的是返回身份令牌
 - `scope` 定义应用想要的域
 - `filterProtocolClaims` 指示 oidc-client 是否需要过滤一些响应中的 OIDC 协议声明：`nonce`, `at_hash`, `iat`, `nbf`, `exp`, `aud`, `iss` 和 `idp`

我们也需要处理 Login 按钮的点击，用于打开一个登录页面弹窗。`signinPopup` 会返回一个 `Promise` ，当完成用户数据的获取以及验证的时候会被解析。

可以通过以下两个方法来获取数据：

 - 作为 Promise 下解析过的数据
 - 作为与 `userLoaded` 事件相关联的数据

在我们的例子中，我们在 `userLoaded` 事件中添加了一个处理器，通过向 `events.addUserLoaded` 方法传递一个回调函数。这个数据包含了几个属性，比如 `id_token`, `scope` 和 `profile` ，都包含了不同的数据项。

我们也需要配置 `popup.html`：

```js
new Oidc.UserManager().signinPopupCallback();
```

在背后，`index.html` 页面中的 `UserManager` 实例会打开一个弹窗并将其重定向到登录页面。当 IdentityServer 将用户重定向到弹窗页面时，信息将会回传给主页面，然后弹窗会自动关闭。

此阶段，你可以登录了：

![login-popup](https://cloud.githubusercontent.com/assets/6102639/17659258/bae8530e-6314-11e6-8b1a-0570a26476cc.PNG)

![login-complete](https://cloud.githubusercontent.com/assets/6102639/17659287/e05f86f2-6314-11e6-941d-1fffe9b94678.PNG)

你可以尝试将 `filterProtocolClaims` 属性设置为 `false` ，然后你将会在 `profile` 属性中看到额外存储的声明。

## JS 应用——域 (JS application - scopes)

是否记得我们的用户有一个叫做 `email` 的声明？但是它并没有在身份令牌中展示，这是因为客户端要求的域只是 `openid` 和 `profile` ——却没有包含这个声明。如果我们想要获取用户的 email ，就需要编辑 `UserManager` 的 `scopes` 属性配置，添加一个叫做 `email` 的域。

在我们的例子中，我们只需要做一些修改，来让 IdentityServer 知道这个域存在于 `Scopes` 类中：

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile,
            // New scope
            StandardScopes.Email
        };
    }
}
```

在客户端的配置中我们并不需要做任何的修改，因为我们已经指定了它可以访问到所有的域。在真实情况中，你应该只给客户端它请求所期望的域，所以这同样需要在客户端做一些更改。（译者注：index.html 中的脚本 `scope` 部分需要添加 `email` 声明）

在这之后，我们就会在用户 profile 中看到 `email` 声明：

![login-email](https://cloud.githubusercontent.com/assets/6102639/17659371/6a822c18-6315-11e6-9311-8bbf80e4bdc0.PNG)

# 第二部分—— API 调用 (Part 2 - API call)

在第二部分，我们将会了解一下怎么从 JS 应用发起对受保护资源的访问。我们现在还需要一个访问令牌，这在我们登录的时候，访问令牌和身份令牌将一并从 IdentityServer 那里获取。

## 创建 API 项目 (Create the API project)

在 Visual Studio 中创建一个空的 Web 应用。

![create api](https://cloud.githubusercontent.com/assets/6102639/12252754/251cd1f0-b92e-11e5-8f8f-469cfc2a0103.png)

这里项目的 URL 被设置成 `http://localhost:60136` 。

## 配置 API (Configuring the API)

出于演示，这我们将会基于 ASP.NET Web API 创建一个非常简单的 API 并安装以下的包：

```
Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName Api
Install-Package Microsoft.Owin.Cors -ProjectName Api
Install-Package Microsoft.AspNet.WebApi.Owin -ProjectName Api
Install-Package IdentityServer3.AccessTokenValidation -ProjectName Api
```

**重要** `IdentityServer3.AccessTokenValidation` 包会间接依赖 `System.IdentityModel.Tokens.Jwt` 包。目前为止，全局更新 `Api` 项目的  NuGet 包会将 `System.IdentityModel.Tokens.Jwt` 包升级为 `5.0.0` ，这就导致 `Api` 项目启动的时候会报错：

![api-update-microsoft-identity-tokens](https://cloud.githubusercontent.com/assets/6102639/17659609/119a6be0-6317-11e6-9ca1-24889b813463.PNG)

解决方案就是将 `System.IdentityModel.Tokens.Jwt` 恢复为旧的兼容版本：

```
Install-Package System.IdentityModel.Tokens.Jwt -ProjectName Api -Version 4.0.2.206221351
```

现在我们创建一个 `Startup` 类并构建我们的 OWIN/Katana 管道。

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        // 允许所有源
        app.UseCors(CorsOptions.AllowAll);

        // 嵌入令牌验证
        app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
        {
            Authority = "https://localhost:44300",

            // 用于访问自省端点
            ClientId = "api",
            ClientSecret = "api-secret",

            RequiredScopes = new[] { "api" }
        });

        // 嵌入 Web API
        var httpConfiguration = new HttpConfiguration();
        httpConfiguration.MapHttpAttributeRoutes();
        httpConfiguration.Filters.Add(new AuthorizeAttribute());

        app.UseWebApi(httpConfiguration);
    }
}
```

这部分代码非常直观，但是还是让我们进一步看一下在管道中使用了什么：

由于 JS 应用会向 API 发起调用，所以必须启用 CORS 。在我们的例子中，我们允许所有的源都可以访问它。同样，在真实的场合中，我们应该将源锁定为我们所期望的源。

我们然后会使用 `IdentityServer3.AccessTokenValidation` 包提供的令牌验证功能。通过设置 `Authority` 属性，那么元数据文档将会被检索到并且用于配置令牌验证设置。自 v2.2 之后，IdentityServer 实现了 [自省端点](../endpoints/introspection.html) 并用于验证令牌。这个端点需要使用到域认证，这就使得它相比于传统的访问令牌验证端点更加安全。

最后，我们添加一个 Web API 配置。注意，我们使用了一个全局 `AuthorizeAttribute` 属性，它会使得每一个 API 端点仅对认证的请求可访问。

我们现在在 API 中添加一个基础的端点：

```csharp
[Route("values")]
public class ValuesController : ApiController
{
    private static readonly Random _random = new Random();

    public IEnumerable<string> Get()
    {
        var random = new Random();

        return new[]
        {
            _random.Next(0, 10).ToString(),
            _random.Next(0, 10).ToString()
        };
    }
}
```

## 更新 IdentityServer 配置 (Updating identityServer configuration)

我们已经引入了一个新的 `api` 域，现在需要将它在 IdentityServer 中进行注册。这可以通过编辑 `IdentityServer` 项目中的 `Scopes` 类来实现：

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile,
            StandardScopes.Email,

            // 注册新的域
            new Scope
            {
                Name = "api",

                DisplayName = "Access to API",
                Description = "This will grant you access to the API",

                ScopeSecrets = new List<Secret>
                {
                    new Secret("api-secret".Sha256())
                },

                Type = ScopeType.Resource
            }
        };
    }
}
```

这个新增的域就是一个资源域，这意味着它将会出现在访问令牌中。同样，我们不需要做什么设置来允许客户端来请求这个新域，是因为有前面所提到的那个特殊的设置，但是在真实的情况下，它却是一个必不可少的步骤。

## 更新 JS 应用 (Updating the JS application)

我们现在需要更新 JS 应用的设置来让它在登录用户的时候可以请求这个新增的 `api` 域。

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    // 添加 `token` 来指定我们也期望得到一个访问令牌
    response_type: 'id_token token',
    // 将新的 `api` 域添加到请求域的列表
    scope: 'openid profile email api',

    filterProtocolClaims: true
};
```

修改如下：

 - 新增一个用于展示访问令牌的面板
 - 更新 `response_type` 来指定我们同时需要使用到身份令牌和访问令牌
 - 登录请求的时候将新增的 `api` 域作为请求的一部分

访问令牌是通过 `access_token` 属性暴露的而过期时间是通过 `expires_at` 属性获取。

值得注意的是 `oidc-client` 帮我们完成了许多痛苦的事情，比如使用签名证书来验证令牌，等等，这些我们都不需要写一行代码。

登录之后，这就是我们得到的：

![access-token](https://cloud.githubusercontent.com/assets/6102639/17659923/1ebf7a16-6319-11e6-99f7-33bef104d0e7.PNG)

## 调用 API (Calling the API)

现在，我们有了一个访问令牌，现在我们可以在 API 中包含这个调用：

```html
[...]
<div class="container main-container">
    <div class="row">
        <div class="col-xs-12">
            <ul class="list-inline list-unstyled requests">
                <li><a href="index.html" class="btn btn-primary">Home</a></li>
                <li><button type="button" class="btn btn-default js-login">Login</button></li>
                <!-- 新增一个按钮用于触发 API 调用 -->
                <li><button type="button" class="btn btn-default js-call-api">Call API</button></li>
            </ul>
        </div>
    </div>

    <div class="row">
        <!-- 将现有的区域改为 6 栏宽 -->
        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">User data</div>
                <div class="panel-body">
                    <pre class="js-user"></pre>
                </div>
            </div>
        </div>

        <!-- 新增一个区域用于展示 API 调用的结果 -->
        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">API call result</div>
                <div class="panel-body">
                    <pre class="js-api-result"></pre>
                </div>
            </div>
        </div>
    </div>
</div>
```

```js
[...]
$('.js-call-api').on('click', function () {
    var headers = {};
    if (user && user.access_token) {
        headers['Authorization'] = 'Bearer ' + user.access_token;
    }

    $.ajax({
        url: 'http://localhost:60136/values',
        method: 'GET',
        dataType: 'json',
        headers: headers
    }).then(function (data) {
        display('.js-api-result', data);
    }).catch(function (error) {
        display('.js-api-result', {
            status: error.status,
            statusText: error.statusText,
            response: error.responseJSON
        });
    });
});
```

现在我们有了一个可以触发 API 调用的按钮以及一个展示调用响应的面板。注意，访问令牌是在请求 `Authorization` 报头添加的。

这里就是我们如果在登录之前调用 API 的结果：

![api-without-access-token](https://cloud.githubusercontent.com/assets/6102639/17660059/2ce58a44-631a-11e6-8f68-c7a4edf85074.PNG)

登录之后再调用的结果：

![api-with-access-token](https://cloud.githubusercontent.com/assets/6102639/17660060/2e898dfa-631a-11e6-9721-a0b98bd3fdcf.PNG)

第一个案例中，还没有访问令牌，所以在请求中没有 `Authorization` 报头，所以访问令牌验证中间件没有做任何事情。因此流向 API 的请求被认为是未认证的，全局 `AuthorizeAttribute` 拒绝了这个请求并使用 `401 Unauthorized` 错误进行响应。

第二个案例中，令牌验证中间件在 `Authorization` 报文头中发现了令牌，然后将其传递给自省端点，自省端点将其标记为合法的令牌，随之创建了一个身份以及它所包含的声明。本次流向 Web API 的请求经过了认证，`AuthorizeAttribute` 限制非常满意，因此 API 端点得到了调用。

# 第三部分——更新令牌，登录和会话检测 (Part 3 - Renewing tokens, logging out and checking sessions)

我们现在已经有了一个能正常运行的 JS 应用，它能结合 IdentityServer 进行登录，同样它也能成功向受保护的资源发起调用。但是用户很快就会遇到一个问题，就是访问令牌过期之后，API 中的访问令牌验证中间件就会拒绝后续的请求。

为了解决这个问题，我们可以设置 `oidc-token-manager` 在令牌即将要过期的时候来自动刷新访问令牌，而不需要用户来完成这件事。


## 过期的令牌 (Expired tokens)

我们首先看一下如何让一个令牌过期。我们需要削减访问令牌的生命周期。这是每个客户端的设置，所以我们需要在 IdentityServer 项目中编辑 `Clients` 类：

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
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 10
            }
        };
    }
}
```

访问令牌的默认生命周期是 1 小时，现在已经更改为 10 秒钟。现在，当你再次成功登录应用后，10 秒后继续调用 API 就会得到一个 `401 Unauthorized` 错误。

## 更新令牌 (Renewing tokens)

我们将要依赖 `oidc-client-js` 提供的特性来帮助我们更新令牌。

在内部，JS 类库会跟踪访问令牌的过期时间，然后通过向 IdentityServer 发起授权请求来获取一个新的令牌。

通过对 [`prompt`](../endpoints/authorization.html) 进行设置，这个过程对用户是不可见的，这会防止在用户有一个合法会话的期间让用户继续登录或者征求他的同意 (consent) 。IdentityServer 会返回一个新的访问令牌用于替换即将过期的令牌。

这里有几个设置与访问令牌过期和更新有关：

 - [`accessTokenExpiring`](https://github.com/IdentityModel/oidc-client-js/wiki#events) 事件会在访问令牌即将过期的时候触发
 - [`accessTokenExpiringNotificationTime`](https://github.com/IdentityModel/oidc-client-js/wiki#configuration) 可以被用来调整在令牌过期之前，据 `accessTokenExpiring` 事件的时间，默认值是 `60` 秒
 - `automaticSilentRenew` 用于指示类库在令牌即将过期的时候自动更新访问令牌
 - `silent_redirect_uri` 需要对其进行配置，这样类库在尝试获取一个新令牌的时候可以指定它的返回 URL 。

这就是 `oidc-client-js` 处理令牌更新自动化的配置。当令牌将要过期的时候，将会创建一个隐藏的动态 `iframe` 。在这个 `iframe` 中，一个新的授权请求将会发送到 IdentityServer 。如果请求成功，IdentityServer 会将 `iframe` 重定向到指定的静默重定向 URL ，在那里，有一段 JS 代码将会更新用户信息，这样在主窗口就会获取到这个更新的用户信息。

现在让我们在配置中做一些修改。

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',
    // 添加静默更新重定向 URL
    silent_redirect_uri: 'http://localhost:56668/silent-renew.html',

    response_type: 'id_token token',
    scope: 'openid profile email api',

    // 添加过期通知时间
    accessTokenExpiringNotificationTime: 4,
    // 设置自动更新访问令牌
    automaticSilentRenew: true,

    filterProtocolClaims: true
};
```

由于我们在 `silent_redirect_uri` 指定了一个新的页面，我们现在需要创建这个页面。

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
</head>
<body>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
    <script>
        new Oidc.UserManager().signinSilentCallback();
    </script>
</body>
</html>
```

第二步就是让 IdentityServer 知道用户认证成功之后重定向的 URL 是合法的：

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
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html",
                    // 这个新页面是登录之后合法的重定向页面
                    "http://localhost:56668/silent-renew.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 10
            }
        };
    }
}
```

当更新成功之后，`UserManager` 会抛出一个 `userLoaded` 事件。由于我们已经处理了这个事件，那么更新的数据将会被自动捕获并展示到 UI 中。

当失败了，它将会抛出一个 `silentRenewError` 事件，我们可以订阅这个事件来了解究竟什么出了错

```js
manager.events.addSilentRenewError(function (error) {
    console.error('error while renewing the access token', error);
});
```

我们已经将访问令牌的生命周期更新为 10 秒钟并且指示 `oidc-client-js` 在访问令牌过期前 4 秒前更新它。那么现在，在我们登录之后，就会看到每隔 6 秒钟就会到 IdentityServer 那里刷新一下访问令牌。

## 登出 (Logging out)

相比于服务器端应用，从一个 JS  应用中登出有不同的意义，因为如果你刷新主页，你将会丢失掉令牌并且需要重新登录。但是当登录弹窗打开，你却仍然有一个合法的会话 cookie 。那么这个弹窗就不会就不会向你寻要凭据并且自动关闭掉。这和令牌管理器自动刷新令牌很相似。

这里的登出意味着从 IdentityServer 那里登出，那么下一次你想要登录 IdentityServer 保护的应用，你需要再次输入你的凭据。

处理这个过程是非常简单的，我们只需要添加一个登出按钮来调用 `UserManager` 实例的 `signoutRedirect` 方法。我们同样需要让 IdentityServer 知道指定的 post-logout 重定向地址是合法的就行：

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
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html",
                    "http://localhost:56668/silent-renew.html"
                },

                // 登出之后合法的重定向 URLs
                PostLogoutRedirectUris = new List<string>
                {
                    "http://localhost:56668/index.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 70
            }
        };
    }
```

```html
[...]
<div class="row">
    <div class="col-xs-12">
        <ul class="list-inline list-unstyled requests">
            <li><a href="index.html" class="btn btn-primary">Home</a></li>
            <li><button type="button" class="btn btn-default js-login">Login</button></li>
            <li><button type="button" class="btn btn-default js-call-api">Call API</button></li>
            <!-- 新增的注销按钮 -->
            <li><button type="button" class="btn btn-danger js-logout">Logout</button></li>
        </ul>
    </div>
</div>
```

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',
    silent_redirect_uri: 'http://localhost:56668/silent-renew.html',
    // 添加 post logout 重定向 URL
    post_logout_redirect_uri: 'http://localhost:56668/index.html',

    response_type: 'id_token token',
    scope: 'openid profile email api',

    accessTokenExpiringNotificationTime: 4,
    automaticSilentRenew: true,

    filterProtocolClaims: true
};
[...]
$('.js-logout').on('click', function () {
    manager
        .signoutRedirect()
        .catch(function (error) {
            console.error('error while signing out user', error);
        });
});
```

当点击 `Logout` 按钮，用户将会被重定向到 IdentityServer 来清除会话 cookie 。

![logout](https://cloud.githubusercontent.com/assets/6102639/12256384/d9de8df0-b950-11e5-91b2-650a0a749a7f.png)

_请注意上面页面的截图是由 IdentityServer 提供的，而不是 JS 应用_

虽然在这个例子中我们是通过主窗口来演示用户登出的过程，但是 `oidc-client-js` 同样提供了在弹窗中登出的方法，和登录的实现方式很像。访问 [`oidc-client-js` 文档](https://github.com/IdentityModel/oidc-client-js/wiki#methods) 获取更多信息。

## 会话检测 (Check session)

我们的 JS 应用中的会话开始于当我们从 IdentityServer 那里获取到身份令牌。IdentityServer 本身支持会话管理，所以在授权响应中一并返回了，就是 `session_state` 属性的值。你可以从 OpenID Connect [spec](http://openid.net/specs/openid-connect-session-1_0.html) 中找到与此相关的规范。

某些情况下，你可能想要知道用户是否在 IdentityServer 中结束了会话，举个例子，在一个应用中登出会让他们在 IdentityServer 中登出。通过计算 `session_state` 的值可以得出结果。如果它等于 IdentityServer 发送的会话状态，那么意味着当前的会话状态没有改变，所以用户仍然处于登录状态。如果不同，那么肯定发生了变化，很可能用户就已经登出了。在这种情况下，建议发出一个静默授权请求，使用 `prompt=none` 设置。如果成功，我们就会得到一个新的身份令牌，这就意味着 IdentityServer 那边的会话仍然合法。如果失败，那么用户就已经登出了，我们就需要让用户重新登录。

不幸的是，JS 应用本身还不能计算 `session_state` 的值，因为它依赖于 IdentityServer 的会话值，而这个值他没法获取到。
[spec](https://openid.net/specs/openid-connect-session-1_0.html) 的设计是需要在一个隐藏的 `iframe` 中从 IdentityServer 加载会话检测端点。然后 JS 应用可以与这个 `iframe` 使用 [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) API 进行通信.

### 会话检测端点 (The checksession endpoint)

这个端点服务一个简答的页面，用于监听 `postMessage` 发送的消息。这个消息中传递的数据用于计算会话状态哈希。如果它匹配 IdentityServer 发送的会话状态中的一个，那么这个页面将会发送一个 `unchanged` 消息给调用它的窗口。没有没有匹配的，将会发送 `changed` 。如果出错，就会发送 `error` 。

### 构建会话检测特性 (Building the session check feature)

幸运的是，`oidc-client-js` 会帮你负责一切的事情。事实上，默认的设置已经对会话状态进行了检测。相关的属性名称为 [`monitorSession`](https://github.com/IdentityModel/oidc-client-js/wiki#usermanager) 。

这就意味着用户一旦登录之后，`oidc-client-js` 就会创建一个隐藏的 `iframe` ，并在当中加载 IdentityServer 的会话检测端点。每隔一段时间，就有一个消息发送到 `iframe` 里，同时包含了客户端 id 和会话状态。发送到 `iframe` 消息会被处理并且接收到的值会用于确认会话是否发生更改。

想要确认它是如我们期望的那样工作，我们会利用到 `oidc-client-js` 提供的日志系统。默认情况下使用的是 no-op logger ，但是我们可以让类库将日志输出到浏览器控制台中。

```js
Oidc.Log.logger = console;
```

想要缩小日志消息数量，我们会延长访问令牌的生命周期。刷新令牌的时候会有很多日志消息，使用现在的设置，这个会每 6 秒钟发生一次。让我们将生命周期延长到 1 分钟。

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
            ClientName = "JS Client",
            ClientId = "js",
            Flow = Flows.Implicit,

            RedirectUris = new List<string>
            {
                "http://localhost:56668/popup.html",
                "http://localhost:56668/silent-renew.html"
            },

            PostLogoutRedirectUris = new List<string>
            {
                "http://localhost:56668/index.html"
            },

            AllowedCorsOrigins = new List<string>
            {
                "http://localhost:56668"
            },

            AllowAccessToAllScopes = true,
            // 将访问令牌生命周期延长到 1 分钟
            AccessTokenLifetime = 60
        }
    };
}
```

最后，如果发现会话发生了改变并且自动登录也失败了，`UserManager` 就会抛出一个 `userSignedOut` 事件。让我们为这个事件添加一个处理器。

```js
manager.events.addUserSignedOut(function () {
    alert('The user has signed out');
});
```

现在导航回应用，登出，打开控制台，再登录进去，我们会在控制台中看到每隔 2 秒钟（默认的间隔）—— `oidc-client-js` 会帮我们从 IdentityServer 那边检测会话的合法性。

![session-check](https://cloud.githubusercontent.com/assets/6102639/17755020/e5fcc630-651a-11e6-8b4b-72b35d3d4f67.png)

为了证实它是正常工作的，让我们打开第二个浏览器选项卡，导航到 JS 应用并登录进去。现在两个选项卡都会帮助我们从 IdentityServer 那里检测会话的合法性。现在我们在当中的一个选项卡中登出，然后就会看到 `userSignedOut` 已经被处理了并出现了一个弹窗。

![logout-event](https://cloud.githubusercontent.com/assets/6102639/17755105/5f317a64-651b-11e6-8c5b-1a2a7cc2b61c.png)