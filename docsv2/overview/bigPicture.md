---
layout: docs-default
---

> [原文](https://identityserver.github.io/Documentation/docsv2/overview/bigPicture.html)

# 整体概况 (The big Picture)

大多数现代应用的架构多少和下图中的相似：

![modern application architecture]({{ site.baseurl }}/assets/images/appArch.png)

典型的交互是：

* 浏览器与 Web 应用通讯

* Web 应用与 Web API 通讯（有时代表它们自己，有时代表用户）

* 基于浏览器的应用与 Web API 通讯

* 本地应用与 Web API 通讯

* 基于服务器端的应用与 Web API 通讯

* Web API 之间互相通讯（有时代表它们自己，有时代表用户）

通常来说每一层（前端，中间层和后端）都需要对资源进行保护并实现验证及（或）授权——并且一般是针对相同的用户存储。

这就是我们为什么不在业务应用/端点中实现这些基础的安全功能，而是将这些重要的功能外包给一个服务的原因——安全令牌服务 (STS) 。

这就促生了下面的安全架构以及协议的使用：

![security protocols]({{ site.baseurl }}/assets/images/protocols.png)

这就将安全问题分为了两个部分：

**验证**

当应用需要知道当前用户的身份时就需要用到验证。通常这些应用帮用户管理他们的数据，因此需要确保用户只能访问他们所被允许访问的那部分数据。最常见的例子就是（典型的） Web 应用——但是本地应用和基于 JS 的应用也需要验证。

最常见的验证协议就是 SAML2p ，WS-Federation 和 OpenID Connect —— SAML2p 是用得最多并且是部署最广泛的。

OpenID Connect 是这三个当中最新的一个，也是普遍被认为是未来可能用得最多的一个，因为它针对现代应用来说最具潜力。一开始它就是针对移动应用场景做的构建并且 API 设计得很友好。

**API 访问**

应用与 API 通讯有两种基本的方式——使用应用身份或者代表用户身份。有时需要将两者结合起来使用。

OAuth2 是一个允许应用从安全令牌服务请求访问令牌并在与 API 的通讯中使用它们的协议。由于验证和授权的集中化，这会同时减小客户端应用和 API 的复杂度，

**OpenID Connect 和 OAuth2 ——结合起来使用更好**

OpenID Connect 和 OAuth2 非常相似——事实上 OpenID Connect 是构建在 OAuth2 上的一个扩展。这就意味着你可以将两个基本安全问题结合起来——验证和 API 的访问结合到一个协议中——常常与安全令牌服务之间只需一个来回。

这就是为什么我们相信，在可预见的未来，OpenID Connect 和 OAuth2 的结合是确保现代应用程序安全的最佳方式。IdentityServer3 是这两个协议的一个实现，并且经过了高度的优化以解决如今移动，本地以及 Web 应用的安全问题。