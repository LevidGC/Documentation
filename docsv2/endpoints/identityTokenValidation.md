---
layout: docs-default
---

# 身份令牌验证端点 (Identity Token Validation Endpoint)

身份令牌验证端点可用于验证身份令牌。这对于无法访问适当的 JWT 或加密库的客户端来说，是非常有用的（比如，JavaScript）。

你可以使用 GET 或者 POST 请求来访问验证端点。由于查询字符串大小有限制，所以推荐使用 POST 请求。

### 示例 (Example)

```
POST /connect/identitytokenvalidation

token=<token>&
client_id=<expected_client_id>
```

```
GET /connect/identitytokenvalidation?token=<token>&client_id=<expected_client_id>
```

成功的响应将会返回 200 状态码以及与令牌关联的声明 (claims) 。失败的响应将会返回 400 状态码以及一个错误消息。

```json
{
  "nonce": "nonce",
  "iat": "1413203421",
  "sub": "88421113",
  "amr": "password",
  "auth_time": "1413203419",
  "idp": "idsrv",
  "iss": "https://idsrv3.com",
  "aud": "implicitclient",
  "exp": "1413203781",
  "nbf": "1413203421"
}
```