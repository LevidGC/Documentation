---
layout: docs-default
---

# Discovery 端点 (Discovery Endpoint)

Discovery 端点可用于检索 IdentityServer 的元数据 - 它返回诸如发行方 (issuer) 名称、关键材料 (key material) 、支持范围等信息。

参见 [spec](http://openid.net/specs/openid-connect-discovery-1_0.html)

### 示例 (Example)

```
GET /.well-known/openid-configuration
```