---
layout: page
title: latest version
permalink: /docsv2/
---

### 概览 (Overview)

* [The big Picture](overview/bigPicture.html)
* [高级特性 (High level Features)](overview/highLevelFeatures.html)
* [术语 (Terminology)](overview/terminology.html)
* [特性和规范 (Features and Specifications)](overview/featuresAndSpecs.html)
* [包 (Packaging)](overview/packaging.html)
* [起步：创建最简单的 OAuth2 授权服务器，客户端和 API (Getting Started: Creating the simplest OAuth2 Authorization Server, Client and API)](overview/simplestOAuth.html)
* [起步：MVC 认证 & Web API (Getting Started: MVC Authentication & Web APIs)](overview/mvcGettingStarted.html)
* [起步：JS 认证 & Web API (Getting Started: JS Authentication & Web APIs)](overview/jsGettingStarted.html)

### 配置 (Configuration)

* [概览 (Overview)](configuration/overview.html)
* [选项 (Options)](configuration/identityServerOptions.html)
* [服务工厂 (Service Factory)](configuration/serviceFactory.html)
* [驻内存服务和仓储 (In-Memory Services and Stores)](configuration/inMemory.html)
* [客户端 (Clients)](configuration/clients.html)
* [Scope 和 Cliam (Scopes and Claims)](configuration/scopesAndClaims.html)
* [Secrets](configuration/secrets.html)
* [密钥，签名和加密 (Keys, Signatures and Cryptography)](configuration/crypto.html)
* [Authentication Options](configuration/authenticationOptions.html)
* [Identity Providers](configuration/identityProviders.html)
* [HSTS](configuration/hsts.html)
* [CORS](configuration/cors.html)
* [Logging](configuration/logging.html)
* [Events](configuration/events.html)

### 端点 (Endpoints)

* [授权/验证 (Authorization/Authentication)](endpoints/authorization.html)
* [令牌 (Token)](endpoints/token.html)
* [UserInfo](endpoints/userinfo.html)
* [Discovery](endpoints/discovery.html)
* [登出 (Logout)](endpoints/endSession.html)
* [令牌撤销 (Token Revocation)](endpoints/revocation.html)
* [令牌自省 (Token Introspection)](endpoints/introspection.html)
* [访问令牌验证 (Access Token Validation)](endpoints/accessTokenValidation.html)
* [身份令牌验证 (Identity Token Validation)](endpoints/identityTokenValidation.html)
* [CSP 错误报告 (CSP Error Report)](endpoints/csp.html)

### Advanced

* [Refresh Tokens](advanced/refreshTokens.html)
* [Registering Services](advanced/customServices.html)
* [DI for Services](advanced/di.html)
* [Caching for client, scope, and user stores](advanced/caching.html)
* [Customizing Views](advanced/customizingViews.html)
* [Localizing messages](advanced/localization.html)
* [CSP](advanced/csp.html)
* [User Service](advanced/userService.html)
* [OWIN environment extension methods](advanced/owin.html)
* [Deployment](advanced/deployment.html)
* [Authenticating Clients with X.509 Certificates](advanced/clientCerts.html)
* [Custom Grant Types](advanced/customGrantTypes.html)
* [Signout](advanced/signout.html)
  * [Session Management](advanced/signout-session.html)
  * [HTTP based logout](advanced/signout-http.html)
* [Federated Signout](advanced/federated-signout.html)
* [Federated post-logout redirects](advanced/federated-post-logout-redirect.html)
* [Invalidating existing login sessions](advanced/session-invalidation.html)

### Consuming Tokens

* [The Katana Access Token Validation Middleware](consuming/overview.html)
* [Options](consuming/options.html)
* [Diagnostics](consuming/diagnostics.html)

### Entity Framework support for Clients, Scopes, and Operational Data
* [Overview](ef/overview.html)
* [Client 和 Scope (Clients and Scopes)](ef/clients_scopes.html)
* [Operational Data](ef/operational.html)
* [Schema Changes and Migrations](ef/migrations.html)
	* [Migrating from 1.x to 2.0](ef/migrations_to_v2.html)
    * [Using Entity Framework migrations with SQL Azure](ef/sql_azure.html)


### WS-Federation

* [Overview](wsFederation/overview.html)
* [Options](wsFederation/options.html)
* [Service Factory](wsFederation/serviceFactory.html)
* [Relying Parties](wsFederation/relyingParties.html)
* [Endpoints](wsFederation/endpoints.html)
* [Entity Framework](wsFederation/entityFramework.html)

### Proof of Possession

* [Overview](pop/overview.html)
* [Requesting PoP Tokens](pop/requesting.html)
* [Consuming PoP Tokens](pop/consuming.html)

### Resources

* [Overview](resources/home.html)
* [Community Add-ons](resources/community.html)
* [Middleware for external Authentication](resources/externalAuthentication.html)