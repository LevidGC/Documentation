---
layout: docs-default
---

# UserInfo 端点 (UserInfo Endpoint)

UserInfo 端点可以用于检索一个主体 (subject) 的身份信息。它需要一个合法的访问令牌以及至少拥有 `openid` 域。

参见 [spec](http://openid.net/specs/openid-connect-core-1_0.html#UserInfo)

### 示例 (Example)

```
GET /connect/userinfo
Authorization: Bearer <access_token>
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
   "sub": "248289761001",
   "name": "Bob Smith",
   "given_name": "Bob",
   "family_name": "Smith",
   "role": [
       "user",
       "admin"
   ]
}
```