---
layout: docs-default
---

# 令牌端点 (Token Endpoint)

令牌端点可用于请求或者刷新令牌（资源所有者密码凭据流，授权码流，客户端凭据流以及自定义许可类型）。

### 支持的参数 (Supported Parameters)

参见 [spec](http://openid.net/specs/openid-connect-core-1_0.html#TokenRequest).

- `grant_type` （必选）
    - `authorization_code`, `client_credentials`, `password`, `refresh_token` 或自定义
- `scope` （对于所有许可类型除了 refresh_token 和 code 外为必选）
- `redirect_uri` （code 许可类型为必选）
- `code` （code 许可为必选）
- `code_verifier` （当使用 proof keys 时为必选 - v2.5 中添加)
- `username` （密码许可类型为必选）
- `password` （密码许可类型为必选）
- `acr_values` （在密码许可类型中可向用户传递额外的信息）
    - 一些具有特殊含义的值：
        - `idp:name_of_idp` 绕过 login/home 领域 (realm) 屏幕，并将用户直接转发到所选的身份提供商 (identity provider) （如果每个客户端配置允许）
        - `tenant:name_of_tenant` 可用于向用户服务传递额外的信息
- `refresh_token` （刷新令牌许可为必选）
- `client_id` （既可以在 post body 中，也可以作为基础认证 (basic authentication) 的 header）
- `client_secret` （既可以在 post body 中，也可以作为基础认证 (basic authentication) 的 header）

### 认证 (Authentication)
所有发送到令牌端点的请求必须被认证 - 既可以通过基础认证来传递客户端 id 和 secret 也可以在 POST body 中添加 `client_id` 和 `client_secret` 字段

当在 `Authorization` 中提供 `client_id` 和 `client_secret` 时应使用如下形式：

* `client_id:client_secret`
* Base64 编码

```csharp
var clientId = "...";
var clientSecret = "...";

var encoding = Encoding.UTF8;
var credentials = string.Format("{0}:{1}", clientId, clientSecret);

var headerValue = Convert.ToBase64String(encoding.GetBytes(credentials));
```

### 示例 (Example)
（为了可读性移除了表单编码并增加了换行）

```
POST /connect/token
Authorization: Basic abcxyz

grant_type=authorization_code&
code=hdh922&
redirect_uri=https://myapp.com/callback
```