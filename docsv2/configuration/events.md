---
layout: docs-default
---

> [原文](https://identityserver.github.io/Documentation/docsv2/configuration/events.html)

# 事件 (Events)

IdentityServer 会在运行的时候触发许多事件，比如：

* 成功/失败的验证 (resource owner flow, pre, partial, local and external)
* Token 的颁发 (identity, access, refresh tokens)
* Token 处理相关事件 (authorization code, refresh token issued/redeemed/refreshed)
* 权限撤销
* 端点成功/失败
* 过期/失效/没有签名证书
* 未处理的异常和内部错误
* 浏览器报告的 CSP 错误。参见 [CSP](../advanced/csp.html) 获取更多信息。

默认情况，这些默认的事件都将会被转发给配置的日志提供商——自定义的事件服务可根据环境情况任意处理或者转发它们。

## 配置事件 (Configuring events)

`EventsOptions` 类有以下设置（默认为 `false`）：
* `RaiseSuccessEvents`
    * 比如，refresh token 被更新或者验证成功
* `RaiseFailureEvents`
    * 比如，验证失败，验证码兑换失败。
* `RaiseErrorEvents`
    * 比如，未处理的异常
* `RaiseInformationEvents`
    * 比如，token 颁发或者证书验证
    
参见 [这里](http://leastprivilege.com/2015/10/22/identityserver3-logging-monitoring-using-serilog-and-seq/) 获取更多关于日志和事件相关信息。
