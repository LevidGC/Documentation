---
layout: docs-default
---

# 包和构建 (Packaging and Builds)

IdentityServer 包含了许多 nuget 包。

### 核心 (Core)

[nuget](https://www.nuget.org/packages/IdentityServer3/) | [github](https://github.com/identityserver/IdentityServer3)

包含了核心的 IdentityServer 对象模型，服务和服务器。核心部分仅包含对驻内存 (in-memory) 配置和用户仓储 (user store) 的支持——但是你可以通过配置插入对其它仓储的支持。这就是其它仓储 (repos) 和包所关心的事。

### 配置仓储 (Configuration stores)

配置数据（客户端和域 (scope)）以及运行时数据（consent ，令牌处理 (token handle)，刷新令牌）的存储。

Entity Framework [nuget](https://www.nuget.org/packages/IdentityServer3.EntityFramework/) | [github](https://github.com/identityserver/IdentityServer3.EntityFramework)

(Community contribution) MongoDb [github](https://github.com/jageall/IdentityServer.v3.MongoDb)

### 用户仓储 (User stores)

身份管理类库的支持。

MembershipReboot [nuget](https://www.nuget.org/packages/IdentityServer3.MembershipReboot/) | [github](https://github.com/identityserver/IdentityServer3.MembershipReboot)

ASP.NET Identity [nuget](https://www.nuget.org/packages/IdentityServer3.AspNetIdentity/) | [github](https://github.com/identityserver/IdentityServer3.AspNetIdentity)

### 插件 (Plugins)

协议插件

WS-Federation [nuget](https://www.nuget.org/packages/IdentityServer3.WsFederation/) | [github](https://github.com/identityserver/IdentityServer3.WsFederation)

### 访问令牌验证中间件 (Access token validation middleware)

API 的 OWIN 中间件。提供了一个简单的方式来验证访问令牌并且实施对域的需求。

[nuget](https://www.nuget.org/packages/IdentityServer3.AccessTokenValidation/) | [github](https://github.com/IdentityServer/IdentityServer3.AccessTokenValidation)

## 开发构建 (Dev builds)

In addition we publish dev/interim builds to MyGet.
Add the following feed to your Visual Studio if you want to give them a try:

[https://www.myget.org/F/identity/](https://www.myget.org/F/identity/)