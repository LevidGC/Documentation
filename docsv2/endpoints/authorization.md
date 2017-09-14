---
layout: docs-default
---

# 授权/验证端点 (Authorization/Authentication Endpoint)

授权端点既可以用于请求访问令牌 (access token) 也可以用于授权码 (authorization code) （隐式流 (implicit flow) 和授权码流 (authorization code flow)）。你可以使用一个 Web 浏览器或者一个 Web view 来发起请求。

### 支持的参数 (Supported Parameters)

参见 [spec](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest).

- `client_id` （必选）
    - 客户端标识符
- `scope` （必选）
    - 一个或者多个注册的域 (scope)
- `redirect_uri` （必选）
    - **必须** 精确匹配客户端注册的任意一个允许的重定向 URL 
- `response_type` （必选）
    - `code` 请求一个授权码
    - `token` 请求一个访问令牌（只允许资源域 (resource scope)）
    - `id_token token` 请求一个身份令牌 (identity token) 和一个访问令牌（同时允许资源域和身份域 (identity scope)）
- `response_mode` （可选）
    - `form_post` 使用表单提交 (form post) 而非编码的片段重定向 (fragment encoded redirect) 来发送令牌响应
- `state` （推荐）
    - idsrv 在令牌响应中会原封不动返回 `state` 值，这是用于关联请求和响应的
- `nonce` （使用隐式流来请求身份令牌则为必选）
    - idsrv 在身份令牌中会原封不动返回 `nonce` 值，这是为了将令牌与请求关联起来。
- `prompt` （可选）
    - `none` 请求期间不会展示任何 UI 。如果不展示任何 UI 是不可能的（例如，因为用户必须登录或同意 (consent)），就会返回一个错误
    - `login` 会展示登录 UI ，即使用户已经登录并有一个有效的会话
- `code_challenge` （使用 proof keys 则为必选 - v2.5 中添加）
    - 在 proof key 流中发送编码质询 (challenge)
- `code_challenge_method` （可选 - 当使用 proof keys 的时候默认为纯文本 - v2.5 中添加）
    - `plain` 指示质询使用的是纯文本（不推荐）
    - `S256` 指示质询是使用 SHA256 哈希过的
- `login_hint` （可选）
    - 在登录页面可用于预先填写 username 字段
- `ui_locales` （可选）
    - 给出了登录 UI 所需的显示语言的提示
- `max_age` （可选）
    - 如果用户的登录会话超过了最大的时限（秒），将显示登录 UI
- `acr_values` （可选）
    - 允许将附加的认证相关信息传递给用户服务——也有一些具有特殊含义的值:
        - `idp:name_of_idp` 绕过 login/home 领域 (realm) 屏幕，并将用户直接转发到所选的身份提供商 (identity provider) （如果每个客户端配置允许）
        - `tenant:name_of_tenant` 可以将一个租户 (tenant) 名称传递给用户服务

### 示例 (Example)
（为了可读性移除了 URL 编码）

```
GET /connect/authorize?client_id=client1&scope=openid email api1&response_type=id_token token&redirect_uri=https://myapp/callback&state=abc&nonce=xyz
```