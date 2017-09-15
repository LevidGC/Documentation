---
layout: docs-default
---

# 服务工厂 (Service Factory)

IdentityServer3 有许多特性用来实现 OpenID Connect 和 OAuth2 。许多特性都已经设计好了，也可以被替换掉。当默认的逻辑没法满足宿主应用程序的需求时这非常有用，或者简单点说，应用程序想要提供一个完全不同的实现。事实上，IdentityServer3 中有一些扩展点需要宿主应用程序来提供（比如配置数据的存储或者用于验证用户凭据的身份管理的实现）。

`IdentityServer3.Core.Configuration.IdentityServerServiceFactory` 拥有所有的这些构建块，而且必须在 startup 阶段通过 `IdentityServerOptions` 类来提供（参见 [这里](http://identityserver.github.io/Documentation/docsv2/configuration/identityServerOptions.html) 获取更多关于配置选项的信息）。

扩展点有以下三类。

## 强制实现 (Mandatory)

* `UserService`
    * 实现结合本地用户仓储，外部用户，声明的检索和登出逻辑来实现用户验证。 [MembershipReboot](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.MembershipReboot) 和 [ASP.NET Identity](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.AspNetIdentity) 都有默认的实现
* `ScopeStore`
    * 实现域配置数据的检索
* `ClientStore`
    * 实现客户端配置数据的检索

`IdentityServerServiceFactory` 允许建立一个服务工厂来提供用户，客户端和域的 in-memory 仓储（参见 [这里](inMemory.html)）。

## 生产场景强制实现（但是有默认的 in-memory 实现） (Mandatory for production scenarios (but with default in-memory implementations))

* `AuthorizationCodeStore`
    * 实现授权码的存储和检索 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FITransientDataRepository.cs))
* `TokenHandleStore` 
    * 实现参考令牌的存储和检索 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FITransientDataRepository.cs))
* `RefreshTokenStore` 
    * 实现刷新令牌的存储和检索 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FITransientDataRepository.cs))
* `ConsentStore` 
    * 实现 consent decisions 的存储和检索 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source/Core/Services/IConsentStore.cs))
* `ViewService`
    * 实现 UI 资产的检索，默认使用嵌入式资产 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIViewService.cs))

## 可选（可以被替换，但是有默认的实现） (Optional (can be replaced, but have default implementations))

* `TokenService`
    * 实现身份和访问令牌的创建 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FITokenService.cs))
* `ClaimsProvider`
    * 实现身份和访问令牌的声明的检索 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIClaimsProvider.cs))
* `TokenSigningService`
    * 实现安全令牌签名的创建 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FITokenSigningService.cs))
* `CustomGrantValidator`
    * 实现自定义许可类型的验证 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FICustomGrantValidator.cs))
* `CustomRequestValidator`
    * 为授权和令牌的请求实现额外的自定义验证 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FICustomRequestValidator.cs))
* `RefreshTokenService`
    * 实现刷新令牌的创建和更新 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIRefreshTokenService.cs))
* `ExternalClaimsFilter`
    * 实现外部身份提供商的声明的过滤和转换 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIExternalClaimsFilter.cs))
* `CustomTokenValidator`
    * 为令牌验证端点实现自定义额外的令牌验证 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FICustomTokenValidator.cs))
* `CustomTokenResponseGenerator`
    * 在令牌响应中添加额外的数据 [接口](https://github.com/IdentityServer/IdentityServer3/blob/dev/source/Core/Services/ICustomTokenResponseGenerator.cs)
* `ConsentService` 
    * 实现同意 (consent) 选择的逻辑 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source/Core/Services/IConsentService.cs))
* `ClientPermissionsService`
    * 实现 consents ，参考和刷新令牌的检索和撤销 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIClientPermissionsService.cs))
* `EventService`
    * 实现将事件转发到某个日志系统（比如，弹性搜索） ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIEventService.cs))
* `RedirectUriValidator`
    * 实现重定向和 post logout URIs 验证 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FIRedirectUriValidator.cs))
* `LocalizationService`
    * 实现本地化字符串显示 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FILocalizationService.cs))
* `CorsPolicyService`
    * 实现 CORS 规则 ([接口](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FServices%2FICorsPolicyService.cs))

参见 [这里](../advanced/customServices.html) 获取更多关于注册自定义服务和仓储实现的信息。
