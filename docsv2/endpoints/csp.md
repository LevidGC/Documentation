---
layout: docs-default
---

# CSP 端点 (CSP Endpoint)

[CSP](../advanced/csp.html) 允许配置一个报告端点 (reporting endpoint) 。IdentityServer 提供了一个端点来记录浏览器报告的 CSP 错误。CSP 错误可以在事件系统 (eventing system) 中以 [events](../configuration/events.html) 的形式抛起。

CSP 报告特性可以通过将 `EndpointOptions` 中的 `EnableCspReportEndpoint` 属性设置为 `false` 来禁用，它是 [`IdentityServerOptions`](../configuration/identityServerOptions.html) 的属性。