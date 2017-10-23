---
layout: docs-default
---

# 使用 X.509 证书验证客户端 (Authenticating Clients using X.509 Certificates)

通常客户端的验证是通过共享的 key 实现的（也就是 client secret）。另一种方式就是使用 X.509 证书。

## 注册客户端 (Registering the client)

你可以通过实现 `ISecretValidator` 接口来完全控制怎么将一个客户端证书映射到对应的 client secret 。默认的实现是将证书的指纹映射到对应的客户端。

下面的片段为客户端凭据流注册一个客户端：

```csharp
var certClient = new Client
{
    ClientName = "Client Credentials Flow Client with Client Certificate",                   
    ClientId = "certclient",
    
    ClientSecrets = new List<Secret>
    { 
        new Secret
        {
            Value = "61B754C541BBCFC6A45A9E9EC5E47D8702B78C29",
            Type = Constants.SecretTypes.X509CertificateThumbprint,
        }
    },

    Flow = Flows.ClientCredentials,
                    
    AllowedScopes = new List<string> 
    {
        "read", 
        "write"
    },
}
```

## 配置宿主 (Configuring the host)

你需要配置宿主来接收客户端证书。对于 IIS ，你需要为 token 端点创建一个 location 元素来配置客户端证书和 SSL 设置：

```xml
<location path="core/connect/token">
  <system.webServer>
    <security>
      <access sslFlags="Ssl, SslNegotiateCert" />
    </security>
  </system.webServer>
</location>
```

**备注** 默认情况下 IIS 是锁定 SSL 设置的——你可能需要在特性委托配置中将它们设置为 `Read/Write` 。

## 请求 token (Requesting the token)

想要请求 token ，你需要为 HTTP 客户端提供客户端证书并将客户端 ID 添加到请求体中。以下示例使用的是 IdentityModel OAuth2 客户端：

```csharp
async Task<TokenResponse> RequestTokenAsync()
{
    var cert = new X509Certificate2("Client.pfx");

    var handler = new WebRequestHandler();
    handler.ClientCertificates.Add(cert);

    var client = new OAuth2Client(
        new Uri("https://identityserver.io/core/connect/token"),
        "certclient",
        handler);

    return await client.RequestClientCredentialsAsync("read write");
}
```
