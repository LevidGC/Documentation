---
layout: docs-default
---

# 自省端点 (Introspection Endpoint)

*v2.2 中添加*

自省端点是对 [RFC 7662](https://tools.ietf.org/html/rfc7662) 的实现。

它可以用于验证参考令牌（如果使用者 (consumer) 不支持适当的 JWT 或密码库，也可以用于验证 JWT）。

自省端点需要使用一个域 (scope) 凭据进行认证（访问令牌只有包含域才可以被允许来自省令牌）。

### 示例 (Example)

```
POST /connect/introspect
Authorization: Basic xxxyyy

token=<token>
```

成功的响应会返回 200 状态码以及有效或者无效的令牌：

```json
{
   "active": true,
   "sub": "123"
}
```

未知的或者过期的令牌被标记为无效：

```json
{
   "active": false,
}
```

如果域没有被授权，那么非法的请求将会返回 400 或 401 状态码。

**备注** 自省端点替代了更老的访问令牌验证端点。由于自省端点需要认证，它在参考令牌中添加了私有特性，而这在之前是没有的。访问令牌验证端点仍然存在，但是不推荐使用它，你可以在 `EndpointOptions` 中禁用它，并选用自省端点。