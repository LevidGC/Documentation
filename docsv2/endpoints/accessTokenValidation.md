---
layout: docs-default
---

# 访问令牌验证端点 (Access token validation endpoint)

访问令牌验证端点可用于验证参考令牌 (reference token) 。如果使用者 (consumer) 不支持适当的 JWT 或密码库，那么它也可以用于验证自包含 (self-contained) 的 JWT 。

你可以使用 GET 或者 POST 请求来访问验证端点。由于查询字符串大小有限制，所以推荐使用 POST 请求。

### 示例 (Example)

```
POST /connect/accesstokenvalidation

token=<token>
```

或

```
GET /connect/accesstokenvalidation?token=<token>
```

成功的响应将会返回 200 状态码以及与令牌关联的声明 (claims) 。失败的响应将会返回 400 状态码以及一个错误消息。

也可以在里面传递一个令牌期望的域 (scope) ：

```
POST /connect/accesstokenvalidation

token=<token>&
expectedScope=calendar
```

**备注** 访问令牌验证端点不执行客户端验证。不要使用参考令牌来进行机密性的使用！