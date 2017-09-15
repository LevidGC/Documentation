---
layout: docs-default
---

# 域和声明 (Scopes and Claims)

**`IdentityServer.Core.Models.Scope` 类是对 OpenID Connect 或者 OAuth2 域的一个建模。**

* `Enabled`
    * 指示域是否启用并可被请求。默认为 `true` 。
* `Name`
    * 域的名称。客户端会使用这个值来请求域。
* `DisplayName`
    * 在 consent screen 中显示的名称。
* `Description`
    * 在 consent screen 中的描述。
* `Required`
    * 指定用户是否可以在 consent screen 中反选这个域。默认为 `false` 。
*  `ScopeSecrets` （v2.2 新增）
    * 在域中添加 secret （用于自省端点）——参见 [这里](secrets.html) 。
*  `AllowUnrestrictedIntrospection` （v2.3 新增）
    * 当时用自省端点的时候允许这个域能看到访问令牌中其它所有的域。
* `Emphasize`
    * 指定在 consent screen 中是否强调这个域。对于敏感或者重要的域使用这个设置。默认为 `false` 。
* `Type`
    * `Identity` （OpenID Connect 相关）或 `Resource` （OAuth2 资源）。默认为 `Resource` 。
* `Claims`
    * 应该包括在身份令牌（身份域）或者访问令牌（资源域）中的用户声明列表。
* `IncludeAllClaimsForUser`
    * 如启用，用户的所有声明将会包含在令牌中。默认为 `false` 。
* `ClaimsRule`
    * 决定什么声明应该包含在令牌中的规则（明确实现）。
* `ShowInDiscoveryDocument`
    * 指定这个域是否显示在 discovery 文档中。默认为 `true` 。

**域也可以将声明指定到对应的令牌中—— `ScopeClaim` 类有以下的属性：**

* `Name`
    * 声明名称
* `Description`
    * 声明描述
* `AlwaysIncludeInIdToken`
    * 指定当前声明是否应该总是存在与身份令牌中（即使也请求了访问令牌）。只应用在身份域上。默认为 `false` 。

**`role` 身份域示例：**

```csharp
var roleScope = new Scope
{
    Name = "roles",
    DisplayName = "Roles",
    Description = "Your organizational roles",
    Type = ScopeType.Identity,

    Claims = new List<ScopeClaim>
    {
        new ScopeClaim(Constants.ClaimTypes.Role, alwaysInclude: true)
    }
};
```
'AlwaysIncludeInIdentityToken' 属性用于指定某个特定的声明应该总是为身份令牌的一部分，即使为 userinfo 端点请求了访问令牌。

**一个 `IdentityManager` API的域示例：**

```csharp
var idMgrScope = new Scope
{
    Name = "idmgr",
    DisplayName = "IdentityManager",
    Type = ScopeType.Resource,
    Emphasize = true,

    Claims = new List<ScopeClaim>
    {
        new ScopeClaim(Constants.ClaimTypes.Name),
        new ScopeClaim(Constants.ClaimTypes.Role)
    }
};
```
