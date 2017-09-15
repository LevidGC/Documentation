---
layout: docs-default
---

# 术语 (Terminology)

规范，文档以及对象模型会使用一个特定的术语，所以你应用先对它们有所了解。

![modern application architecture]({{ site.baseurl }}/assets/images/terminology.png)

## OpenID Connect 提供商 (OpenID Connect Provider (OP))

IdentityServer 是一个 OpenID Connect 提供商——它实现了 OpenID Connect 协议（同样包括 OAuth2）。

不同的文献可能对同一个角色使用不同的术语来描述——你很可能也看到过安全令牌服务 (security token service) ，身份提供商 (identity provider) ，授权服务器 (authorization server) ，IP-STS 等等。

但是它们本质上是一样的：就是向客户端颁发安全令牌的软件。

IdentityServer 有大量的工作 (job) 和特性——包括：

* 使用本地的账户仓储 (store) 或者外部分身份提供商来对用于进行认证

* 提供会话管理和单点登录 (single sign-on)

* 管理和认证客户端

* 向客户端颁发身份和访问令牌

* 验证令牌

## 客户端 (Client)

客户端就是一个从 IdentityServer 那里请求令牌的软件——既可以认证用户，也可以访问资源（也经常称为依赖方 (relying party) 或 RP）。一个客户端必须在 OP 那边进行注册。

客户端的例子：Web 应用，本地移动或者桌面应用，SPAs （单页面应用），服务器进程 等等。

## 用户 (User)

用户是一个使用注册客户端访问其数据的人。

## 域 (Scope)

域是客户端想要访问的资源的标识符。这个域在认证或者令牌请求的期间将会发送给 OP 。默认情况下，每一个客户端都可以请求所有域的令牌，但是你可以对其进行限制。

有以下两种类型的域。

### 身份域 (Identity scopes)

请求一个用户的身份信息（也叫做声明 (claim)），比如，在 OpenID Connect 中，用户的名称或者 email 地址被建模成一个域。

比如，有一个叫做 `profile` 的域，它包含了 first name, last name, preferred username, gender, profile picture 等等。

你可以在 [here](http://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) 了解更多标准的域，并且在 IdentityServer 中你可以根据你的需求来创建你自己的域。

### 资源域 (Resource scopes)

资源域标识的是 Web APIs （也叫做资源服务器）——比如，你可以将一个叫做 `calendar` 的域用来表示日历 API 。

## 认证/令牌请求 (Authentication/Token Request)

客户端向 OP 请求令牌。基于请求的域，OP 将会返回一个身份令牌，一个访问令牌，或者同时返回两个。

## 身份令牌 (Identity Token)

身份令牌表示的是认证过程的结果。它至少包含一个用户的标识符（叫做 `sub` ，也称为主体声明）。它也可以包含用户额外的信息以及用户在 OP 怎么认证的详细信息。

## 访问令牌 (Access Token)

访问令牌允许访问资源。客户端请求访问令牌，并将其传递给 API 。访问令牌包含了客户端以及用户（如果提供）的信息。API 使用这个信息来对它们的数据授权访问。