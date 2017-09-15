---
layout: docs-default
---

# 密钥，签名和加密 (Keys, Signatures and Cryptography)

IdentityServer 在许多个方面都依赖于加密。这里是一个概览。

## 安全传输 (Secure Transport (HTTPS))

默认情况下，IdentityServer 要求所有的传入连接得使用 HTTPS 。必须保证与 IdentityServer 的通信只在安全传输上完成。有一些特定的部署场景，比如 SSL offloading，可以轻松地实现这个需求。参见 [部署](../advanced/deployment.html) 部分获取更多信息。

SSL/TLS 配置是在托管层级实现的——比如，在 IIS 中或直接使用 HTTP.SYS 。

## 令牌签名 (Token Signing)
Identity 和 JWT access token 是使用 RSA 算法 (RS256) 的 X.509 证书进行签名的。签名证书是在 `IdentityServerOptions` 中使用 `SigningCertificate` 属性设置的。对于 identity token 和 JWT access token 是需要强制设置此属性的。

如果你仅使用 reference token ，那么就不需要设置签名证书。

**备注** 密钥长度最少为 2048 字节。 

### 签名密钥翻转 (Signing Key Rollover)

X.509 证书具有有限的生命周期。在不让你的应用停机的情况下更新和翻转签名密钥就是一个挑战。IdentityServer 有几个特性来使这个过程更加简单：

* Discovery 文档发布了当前使用的公钥（以及一个备用的）。通过这种方式，使用者了解到密钥的内容。
* 所有的 JWT 包含一个密钥标识符，与 discovery 文档中发布的密钥相匹配。
* Access token 验证中间件会每隔 24 小时检查一下 discovery 文档以更新它的密钥配置。

密钥翻转可能如下工作：

1. 获取新的证书来替换旧的
2. 将新的证书设置为 `SecondarySigningCertificate` 。IdentityServer 现在会在 discovery 文档中同时发布这两个证书并且 access token 验证中间件会接收使用这两个密钥签名的 token 。
3. 至少等待 24 小时供每个使用者有机会来更新它的配置
4. 将新的签名设置为主要的 `SigningCertificate` 。
5. 将旧的证书作为备用证书保存以供后续需要（可能你有生命周期较长的 token 需要使用到旧的证书来验证）

## Cookie 保护 (Cookie Protection)

Cookie 必须也要受到保护。IdentityServer 为此使用的是 Katana 数据保护基础结构。

如果你的应用托管在 IIS 中，Katana 将会使用 ASP.NET machine key 来保护所有的 cookie 。如果是部署在 web farm 中，你需要在所有节点上同步这些密钥。

如果是自托管的，在你的宿主中既可以使用任何自定义的 Katana 数据保护提供商，也可以使用内置基于 X.509 证书的 IdentityServer 数据保护。
