---
layout: docs-default
---

# Scope 和 Claim (Scopes and Claims)

**`IdentityServer.Core.Models.Scope` 类是对 OpenID Connect 或 OAuth2 scope 的建模。**

* `Enabled`
    * 指示 scope 是否启用且可被请求。默认为 `true` 。
* `Name`
    * Scope 的名称。客户端会使用这个值来请求 scope 。
* `DisplayName`
    * 在 consent screen 中显示的名称。
* `Description`
    * 在 consent screen 中的描述。
* `Required`
    * 指定用户是否可以在 consent screen 中反选这个 scope 。默认为 `false` 。
*  `ScopeSecrets` （v2.2 新增）
    * 在 scope 中添加 secret （用于 introspection endpoint ）——参见 [这里](secrets.html) 。
*  `AllowUnrestrictedIntrospection` （v2.3 新增）
    * 当时使用 introspection endpoint 的时候允许这个 scope 能看到 access token 中其它所有的 scope 。
* `Emphasize`
    * 指定在 consent screen 中是否强调这个 scope 。对于敏感或者重要的 scope 使用这个设置。默认为 `false` 。
* `Type`
    * `Identity` （OpenID Connect 相关）或 `Resource` （OAuth2 资源）。默认为 `Resource` 。
* `Claims`
    * 应该包括在 identity token (identity scope) 或者 access token (resource scope) 中的用户 claim 列表。
* `IncludeAllClaimsForUser`
    * 如启用，用户所有的 claim 将会包含在 token 中。默认为 `false` 。
* `ClaimsRule`
    * 决定什么 claim 应该包含在 token 中的规则（明确实现）。
* `ShowInDiscoveryDocument`
    * 指定这个 scope 是否显示在 discovery 文档中。默认为 `true` 。

**Scope 也可以将 claim 指定到对应的 token 中—— `ScopeClaim` 类有以下的属性：**

* `Name`
    * Claim 名称
* `Description`
    * Claim 描述
* `AlwaysIncludeInIdToken`
    * 指定当前 claim 是否应该总是存在与 identity token 中（即使也请求了 access token ）。只应用在 identity scope 上。默认为 `false` 。

**`role` identity scope 示例：**

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
'AlwaysIncludeInIdentityToken' 属性用于指定某个特定的 claim 应该总是为 identity token 的一部分，即使为 userinfo endpoint 请求了 access token 。

**一个 `IdentityManager` API 的 scope 示例：**

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
