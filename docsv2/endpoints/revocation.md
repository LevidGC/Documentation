---
layout: docs-default
---

# 令牌撤销 (Token Revocation)

这个端点可以撤销访问令牌（仅参考令牌）和刷新令牌。它实现了令牌撤销规范 ([RFC 7009](https://tools.ietf.org/html/rfc7009)) 。

### 支持的参数 (Supported parameters)

* `token` （必选）
    * 待撤销的令牌
* `token_type_hint`
    * `access_token` 或者 `refresh_token`

请求必须经过其中一个受支持的客户端认证方法认证。

### 示例 (Example)

```
POST /connect/revocation HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

token=45ghiukldjahdnhzdauz&token_type_hint=refresh_token
```