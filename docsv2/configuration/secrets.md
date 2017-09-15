---
layout: docs-default
---

# Secrets

Secret 定义了机器（比如，client 或 scope）该怎么使用 IdentityServer 来认证。

Client secret 定义示例：

```csharp
var client = new Client
{
    ClientName = "Client Credentials Flow Client",
    
    ClientId = "client",
    ClientSecrets = new List<Secret>
    { 
        new Secret("secret".Sha256())
    },

    Flow = Flows.ClientCredentials   
};
```

Scope secret 定义示例：

```csharp
var scope = new Scope
{
    Name = "api1",
    DisplayName = "Our API",
    Type = ScopeType.Resource,

    ScopeSecrets = new List<Secret>
    {
        new Secret("secret".Sha256())
    }
};
```

上面的代码片段将 `secret` 值设为共享的 secret ——并使用 SHA256 进行哈希。`ClientSecret` 是一个列表，也就是说 client 可以有多个 secret 。这对于 key rotation 很有用。

让我们仔细看一下 `Secret` 类：

* `Value` secret 的值。通过 secret validator 翻译（比如，一个类 "password" 的共享 secret 或者其它标识凭据的东西）
* `Description` secret 的描述——用于将其它信息附加到 secret 上
* `Expiration` secret 过期的时间点
* `Type` 一些用于向 secret validator 提示 secret 的类型的字符串（比如，"SharedSecret" 或者 "X509CertificateThumbprint"）

## Secret 解析 (Secret parsers)

client 传输 secret 有多种方式—— OAuth2 规范提及了 HTTP Basic 认证或使用 POST body 。另外 IdentityServer 还支持 X.509 client 证书（参见 [这里](../advanced/clientCerts.html)）。

Secret 解析是一个扩展点——如果你想支持除了上面所提及的机制来传输 secret ，那么你可以实现 `ISecretParser` 接口并将你的实现在服务工厂里添加到 `SecretParsers` 集合中。

## Secret 验证 (Secret validation)

从传入的请求中提取出 secret 后，必须要对它进行验证。默认，我们支持使用 SHA256 或 SHA512 哈希或者 X.509 证书（证书指纹默认用于对访问者进行认证）存储的共享 secret 。

Secret 验证也是一个扩展点——如果你需要支持不同的验证方法，比如，X.509 证书使用 chain/peer 可信任的专有名称，你可以实现 `ISecretValidator` 接口并将你的实现在服务工厂里添加到 `SecretValidators` 集合中。
