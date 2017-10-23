---
layout: docs-default
---

# Refresh Token (Refresh Tokens)

- Refresh token 在以下的流中使用：授权码，混合和资源所有者密码凭据流。
- 客户端需要被允许请求 `offline_access` scope 来获取一个 refresh token 。

### Client 类的设置 (Settings on the Client class)
- `RefreshTokenUsage` 
    - ReUse: 当刷新 token 的时候 refresh token 句柄保持不变
    - OneTime: 当刷新 token 的时候 refresh token 句柄将会被更新
- `RefreshTokenExpiration`
    - Absolute: refresh token 在一个固定的时间点过期（通过 AbsoluteRefreshTokenLifetime 指定）
    - Sliding: 当刷新 token 的时候，refresh token 的生命周期将会被更新（数量将在 SlidingRefreshTokenLifetime 中指定）。生命周期不超过绝对生命周期。
- `AbsoluteRefreshTokenLifetime`
    - refresh token 的最大生命周期以秒为计量单位。默认为 2592000 秒 / 30 天
- `SlidingRefreshTokenLifetime`
    - refresh token 的滑动生命周期以秒为计量单位。默认为 1296000 秒 / 15 天

### 使用 (Usage)

- 请求 `offline_access` scope （使用授权码或者资源所有者密码凭据流）
- 使用 `refresh_token` 许可来刷新 token 


### Samples

- Client [示例](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Clients) 既有 [资源所有者](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Clients/ConsoleResourceOwnerRefreshTokenClient) 也有 [授权码流](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Clients/MvcCodeFlowClientManual) 客户端。
