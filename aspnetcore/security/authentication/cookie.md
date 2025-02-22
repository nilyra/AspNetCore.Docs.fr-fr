---
title: Utiliser l’authentification par cookie sans ASP.NET Core identité
author: rick-anderson
description: Découvrez comment utiliser l’authentification par cookie sans ASP.NET Core identité.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 08/20/2019
uid: security/authentication/cookie
ms.openlocfilehash: 76c7fc20c8870668ca7c65d975e2ed59f40f7dc8
ms.sourcegitcommit: 116bfaeab72122fa7d586cdb2e5b8f456a2dc92a
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/05/2019
ms.locfileid: "70384828"
---
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a>Utiliser l’authentification par cookie sans ASP.NET Core identité

Par [Rick Anderson](https://twitter.com/RickAndMSFT) et [Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core identité est un fournisseur d’authentification complet et complet pour la création et la gestion des connexions. Toutefois, il est possible d’utiliser un fournisseur d’authentification d’authentification basée sur les cookies sans ASP.NET Core d’identité. Pour plus d'informations, consultez <xref:security/authentication/identity>.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

À des fins de démonstration dans l’exemple d’application, le compte d’utilisateur de l’utilisateur hypothétique, Maria Rodriguez, est codé en dur dans l’application. Utilisez l' adresse `maria.rodriguez@contoso.com` de messagerie et un mot de passe pour vous connecter à l’utilisateur. L’utilisateur est authentifié dans la `AuthenticateUser` méthode dans le fichier *pages/Account/login. cshtml. cs* . Dans un exemple réel, l’utilisateur est authentifié par rapport à une base de données.

## <a name="configuration"></a>Configuration

Si l’application n’utilise pas le sous- [package Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), créez une référence de package dans le fichier projet pour le package [Microsoft. AspNetCore. Authentication. Cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .

Dans la `Startup.ConfigureServices` méthode, créez les services d’intergiciel (middleware) <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> d' <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> authentification avec les méthodes et :

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme>passé à `AddAuthentication` définit le schéma d’authentification par défaut pour l’application. `AuthenticationScheme`est utile lorsqu’il existe plusieurs instances d’authentification de cookie et que vous souhaitez [autoriser avec un schéma spécifique](xref:security/authorization/limitingidentitybyscheme). L’affectation `AuthenticationScheme` de la valeur à [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fournit la valeur « cookies » pour le schéma. Vous pouvez fournir n’importe quelle valeur de chaîne qui distingue le schéma.

Le schéma d’authentification de l’application est différent du schéma d’authentification des cookies de l’application. Lorsqu’un schéma d’authentification de cookie n' <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>est pas fourni `CookieAuthenticationDefaults.AuthenticationScheme` à, il utilise (« cookies »).

La propriété du <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> cookie d’authentification a la `true` valeur par défaut. Les cookies d’authentification sont autorisés lorsqu’un visiteur du site n’a pas consenti à la collecte de données. Pour plus d'informations, consultez <xref:security/gdpr#essential-cookies>.

Dans `Startup.Configure`, appelez `UseAuthentication` et `UseAuthorization` pour définir la `HttpContext.User` propriété et exécuter l’intergiciel (middleware) des autorisations pour les demandes. Appelez les `UseAuthentication` méthodes `UseAuthorization` et avant d' `UseEndpoints`appeler :

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe est utilisée pour configurer les options du fournisseur d’authentification.

Définissez `CookieAuthenticationOptions` dans la configuration de service pour l’authentification `Startup.ConfigureServices` dans la méthode :

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Intergiciel de stratégie de cookie

L’intergiciel (middleware) de [stratégie de cookie](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) active les fonctionnalités de stratégie de cookie. L’ajout de l’intergiciel au pipeline de traitement de l'&mdash;application est sensible à l’ordre. il affecte uniquement les composants en aval inscrits dans le pipeline.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

À <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> utiliser fourni à l’intergiciel (middleware) de stratégie de cookie pour contrôler les caractéristiques globales du traitement des cookies et raccorder des gestionnaires de traitement des cookies lorsque les cookies sont ajoutés ou supprimés.

La valeur <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> par défaut `SameSiteMode.Lax` est d’autoriser l’authentification OAuth2. Pour appliquer strictement la même stratégie de site de `SameSiteMode.Strict`, définissez le `MinimumSameSitePolicy`. Bien que ce paramètre interrompe OAuth2 et d’autres schémas d’authentification Cross-Origin, il élève le niveau de sécurité des cookies pour les autres types d’applications qui ne reposent pas sur le traitement des demandes Cross-Origin.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

Le paramètre d’intergiciel de stratégie de `MinimumSameSitePolicy` cookie pour peut affecter la `Cookie.SameSite` valeur `CookieAuthenticationOptions` de dans les paramètres en fonction du tableau ci-dessous.

| MinimumSameSitePolicy | Cookie.SameSite | Paramètre de cookie. SameSite résultant |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode.None     | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Lax      | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Lax<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Strict   | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Strict<br>SameSiteMode.Strict<br>SameSiteMode.Strict |

## <a name="create-an-authentication-cookie"></a>Créer un cookie d’authentification

Pour créer un cookie contenant des informations sur l’utilisateur <xref:System.Security.Claims.ClaimsPrincipal>, construisez un. Les informations utilisateur sont sérialisées et stockées dans le cookie. 

Créez un <xref:System.Security.Claims.ClaimsIdentity> avec les s <xref:System.Security.Claims.Claim>requis et appelez <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> pour vous connecter à l’utilisateur :

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync`crée un cookie chiffré et l’ajoute à la réponse actuelle. Si `AuthenticationScheme` n’est pas spécifié, le schéma par défaut est utilisé.

Le système de [protection des données](xref:security/data-protection/using-data-protection) de ASP.net Core est utilisé pour le chiffrement. Pour une application hébergée sur plusieurs ordinateurs, l’équilibrage de charge entre les applications ou l’utilisation d’une batterie de serveurs Web, [configurez la protection des données](xref:security/data-protection/configuration/overview) pour utiliser le même anneau de clé et l’identificateur d’application.

## <a name="sign-out"></a>Déconnexion

Pour déconnecter l’utilisateur actuel et supprimer son cookie, appelez <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>:

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Si `CookieAuthenticationDefaults.AuthenticationScheme` (ou « cookies ») n’est pas utilisé comme modèle (par exemple, « ContosoCookie »), fournissez le schéma utilisé lors de la configuration du fournisseur d’authentification. Dans le cas contraire, le schéma par défaut est utilisé.

## <a name="react-to-back-end-changes"></a>Réagir aux modifications principales

Une fois qu’un cookie est créé, le cookie est la seule source d’identité. Si un compte d’utilisateur est désactivé dans les systèmes principaux :

* Le système d’authentification de cookie de l’application continue à traiter les demandes en fonction du cookie d’authentification.
* L’utilisateur reste connecté à l’application tant que le cookie d’authentification est valide.

L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> événement peut être utilisé pour intercepter et remplacer la validation de l’identité du cookie. La validation du cookie à chaque demande atténue le risque d’accès des utilisateurs révoqués à l’application.

Une approche de la validation de cookie est basée sur le suivi de la modification de la base de données utilisateur. Si la base de données n’a pas été modifiée depuis l’émission du cookie de l’utilisateur, il n’est pas nécessaire de réauthentifier l’utilisateur si son cookie est toujours valide. Dans l’exemple d’application, la base de données `IUserRepository` est implémentée `LastChanged` dans et stocke une valeur. Lorsqu’un utilisateur est mis à jour dans la base `LastChanged` de données, la valeur est définie sur l’heure actuelle.

Afin d’invalider un cookie lorsque la base de données change en fonction `LastChanged` de la valeur, créez le cookie `LastChanged` avec une revendication contenant `LastChanged` la valeur actuelle de la base de données :

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

Pour implémenter une substitution pour l' `ValidatePrincipal` événement, écrivez une méthode avec la signature suivante dans une classe qui dérive de <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

Voici un exemple d’implémentation de `CookieAuthenticationEvents`:

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Inscrivez l’instance des événements lors de l’inscription du `Startup.ConfigureServices` service de cookie dans la méthode. Fournissez une [inscription de service étendue](xref:fundamentals/dependency-injection#service-lifetimes) pour `CustomCookieAuthenticationEvents` votre classe :

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Imaginez une situation dans laquelle le nom de l’utilisateur est&mdash;mis à jour une décision qui n’affecte en rien la sécurité. Si vous souhaitez mettre à jour le principal de l’utilisateur de manière non `context.ReplacePrincipal` destructrice, appelez `context.ShouldRenew` et affectez à `true`la propriété la valeur.

> [!WARNING]
> L’approche décrite ici est déclenchée à chaque demande. La validation des cookies d’authentification pour tous les utilisateurs à chaque demande peut entraîner une baisse importante des performances de l’application.

## <a name="persistent-cookies"></a>Cookies persistants

Vous pouvez souhaiter que le cookie soit rendu persistant entre les sessions de navigateur. Cette persistance doit être activée uniquement avec le consentement explicite de l’utilisateur avec une case à cocher « Mémoriser mes propres » lors de la connexion ou d’un mécanisme similaire. 

L’extrait de code suivant crée une identité et un cookie correspondant qui subsiste à travers les fermetures du navigateur. Les paramètres d’expiration décalés précédemment configurés sont honorés. Si le cookie expire pendant que le navigateur est fermé, le navigateur efface le cookie une fois qu’il a redémarré.

Définir <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> sur `true` dans :<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>Expiration absolue des cookies

Une heure d’expiration absolue peut être définie <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>avec. Pour créer un cookie persistant, `IsPersistent` doit également être défini. Dans le cas contraire, le cookie est créé avec une durée de vie basée sur la session et peut expirer avant ou après le ticket d’authentification qu’il contient. Lorsque `ExpiresUtc` est défini, il remplace la valeur de l' <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> option de <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>, si elle est définie.

L’extrait de code suivant crée une identité et un cookie correspondant qui dure 20 minutes. Cela ignore tous les paramètres d’expiration décalés précédemment configurés.

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core identité est un fournisseur d’authentification complet et complet pour la création et la gestion des connexions. Toutefois, il est possible d’utiliser un fournisseur d’authentification d’authentification basée sur les cookies sans ASP.NET Core d’identité. Pour plus d'informations, consultez <xref:security/authentication/identity>.

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

À des fins de démonstration dans l’exemple d’application, le compte d’utilisateur de l’utilisateur hypothétique, Maria Rodriguez, est codé en dur dans l’application. Utilisez l' adresse `maria.rodriguez@contoso.com` de messagerie et un mot de passe pour vous connecter à l’utilisateur. L’utilisateur est authentifié dans la `AuthenticateUser` méthode dans le fichier *pages/Account/login. cshtml. cs* . Dans un exemple réel, l’utilisateur est authentifié par rapport à une base de données.

## <a name="configuration"></a>Configuration

Si l’application n’utilise pas le sous- [package Microsoft. AspNetCore. app](xref:fundamentals/metapackage-app), créez une référence de package dans le fichier projet pour le package [Microsoft. AspNetCore. Authentication. Cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/) .

Dans la `Startup.ConfigureServices` méthode, créez le service d’intergiciel d’authentification avec <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> les <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> méthodes et :

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme>passé à `AddAuthentication` définit le schéma d’authentification par défaut pour l’application. `AuthenticationScheme`est utile lorsqu’il existe plusieurs instances d’authentification de cookie et que vous souhaitez [autoriser avec un schéma spécifique](xref:security/authorization/limitingidentitybyscheme). L’affectation `AuthenticationScheme` de la valeur à [CookieAuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) fournit la valeur « cookies » pour le schéma. Vous pouvez fournir n’importe quelle valeur de chaîne qui distingue le schéma.

Le schéma d’authentification de l’application est différent du schéma d’authentification des cookies de l’application. Lorsqu’un schéma d’authentification de cookie n' <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>est pas fourni `CookieAuthenticationDefaults.AuthenticationScheme` à, il utilise (« cookies »).

La propriété du <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> cookie d’authentification a la `true` valeur par défaut. Les cookies d’authentification sont autorisés lorsqu’un visiteur du site n’a pas consenti à la collecte de données. Pour plus d'informations, consultez <xref:security/gdpr#essential-cookies>.

Dans la `Startup.Configure` méthode, appelez la `UseAuthentication` méthode pour appeler l’intergiciel (middleware) d’authentification `HttpContext.User` qui définit la propriété. Appelez la `UseAuthentication` méthode avant d' `UseMvcWithDefaultRoute` appeler `UseMvc`ou :

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

La <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> classe est utilisée pour configurer les options du fournisseur d’authentification.

Définissez `CookieAuthenticationOptions` dans la configuration de service pour l’authentification `Startup.ConfigureServices` dans la méthode :

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>Intergiciel de stratégie de cookie

L’intergiciel (middleware) de [stratégie de cookie](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware) active les fonctionnalités de stratégie de cookie. L’ajout de l’intergiciel au pipeline de traitement de l'&mdash;application est sensible à l’ordre. il affecte uniquement les composants en aval inscrits dans le pipeline.

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

À <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> utiliser fourni à l’intergiciel (middleware) de stratégie de cookie pour contrôler les caractéristiques globales du traitement des cookies et raccorder des gestionnaires de traitement des cookies lorsque les cookies sont ajoutés ou supprimés.

La valeur <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> par défaut `SameSiteMode.Lax` est d’autoriser l’authentification OAuth2. Pour appliquer strictement la même stratégie de site de `SameSiteMode.Strict`, définissez le `MinimumSameSitePolicy`. Bien que ce paramètre interrompe OAuth2 et d’autres schémas d’authentification Cross-Origin, il élève le niveau de sécurité des cookies pour les autres types d’applications qui ne reposent pas sur le traitement des demandes Cross-Origin.

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

Le paramètre d’intergiciel de stratégie de `MinimumSameSitePolicy` cookie pour peut affecter la `Cookie.SameSite` valeur `CookieAuthenticationOptions` de dans les paramètres en fonction du tableau ci-dessous.

| MinimumSameSitePolicy | Cookie.SameSite | Paramètre de cookie. SameSite résultant |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode.None     | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Lax      | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Lax<br>SameSiteMode.Lax<br>SameSiteMode.Strict |
| SameSiteMode.Strict   | SameSiteMode.None<br>SameSiteMode.Lax<br>SameSiteMode.Strict | SameSiteMode.Strict<br>SameSiteMode.Strict<br>SameSiteMode.Strict |

## <a name="create-an-authentication-cookie"></a>Créer un cookie d’authentification

Pour créer un cookie contenant des informations sur l’utilisateur <xref:System.Security.Claims.ClaimsPrincipal>, construisez un. Les informations utilisateur sont sérialisées et stockées dans le cookie. 

Créez un <xref:System.Security.Claims.ClaimsIdentity> avec les s <xref:System.Security.Claims.Claim>requis et appelez <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> pour vous connecter à l’utilisateur :

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync`crée un cookie chiffré et l’ajoute à la réponse actuelle. Si `AuthenticationScheme` n’est pas spécifié, le schéma par défaut est utilisé.

Le système de [protection des données](xref:security/data-protection/using-data-protection) de ASP.net Core est utilisé pour le chiffrement. Pour une application hébergée sur plusieurs ordinateurs, l’équilibrage de charge entre les applications ou l’utilisation d’une batterie de serveurs Web, [configurez la protection des données](xref:security/data-protection/configuration/overview) pour utiliser le même anneau de clé et l’identificateur d’application.

## <a name="sign-out"></a>Déconnexion

Pour déconnecter l’utilisateur actuel et supprimer son cookie, appelez <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>:

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

Si `CookieAuthenticationDefaults.AuthenticationScheme` (ou « cookies ») n’est pas utilisé comme modèle (par exemple, « ContosoCookie »), fournissez le schéma utilisé lors de la configuration du fournisseur d’authentification. Dans le cas contraire, le schéma par défaut est utilisé.

## <a name="react-to-back-end-changes"></a>Réagir aux modifications principales

Une fois qu’un cookie est créé, le cookie est la seule source d’identité. Si un compte d’utilisateur est désactivé dans les systèmes principaux :

* Le système d’authentification de cookie de l’application continue à traiter les demandes en fonction du cookie d’authentification.
* L’utilisateur reste connecté à l’application tant que le cookie d’authentification est valide.

L' <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> événement peut être utilisé pour intercepter et remplacer la validation de l’identité du cookie. La validation du cookie à chaque demande atténue le risque d’accès des utilisateurs révoqués à l’application.

Une approche de la validation de cookie est basée sur le suivi de la modification de la base de données utilisateur. Si la base de données n’a pas été modifiée depuis l’émission du cookie de l’utilisateur, il n’est pas nécessaire de réauthentifier l’utilisateur si son cookie est toujours valide. Dans l’exemple d’application, la base de données `IUserRepository` est implémentée `LastChanged` dans et stocke une valeur. Lorsqu’un utilisateur est mis à jour dans la base `LastChanged` de données, la valeur est définie sur l’heure actuelle.

Afin d’invalider un cookie lorsque la base de données change en fonction `LastChanged` de la valeur, créez le cookie `LastChanged` avec une revendication contenant `LastChanged` la valeur actuelle de la base de données :

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

Pour implémenter une substitution pour l' `ValidatePrincipal` événement, écrivez une méthode avec la signature suivante dans une classe qui dérive de <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>:

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

Voici un exemple d’implémentation de `CookieAuthenticationEvents`:

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Inscrivez l’instance des événements lors de l’inscription du `Startup.ConfigureServices` service de cookie dans la méthode. Fournissez une [inscription de service étendue](xref:fundamentals/dependency-injection#service-lifetimes) pour `CustomCookieAuthenticationEvents` votre classe :

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

Imaginez une situation dans laquelle le nom de l’utilisateur est&mdash;mis à jour une décision qui n’affecte en rien la sécurité. Si vous souhaitez mettre à jour le principal de l’utilisateur de manière non `context.ReplacePrincipal` destructrice, appelez `context.ShouldRenew` et affectez à `true`la propriété la valeur.

> [!WARNING]
> L’approche décrite ici est déclenchée à chaque demande. La validation des cookies d’authentification pour tous les utilisateurs à chaque demande peut entraîner une baisse importante des performances de l’application.

## <a name="persistent-cookies"></a>Cookies persistants

Vous pouvez souhaiter que le cookie soit rendu persistant entre les sessions de navigateur. Cette persistance doit être activée uniquement avec le consentement explicite de l’utilisateur avec une case à cocher « Mémoriser mes propres » lors de la connexion ou d’un mécanisme similaire. 

L’extrait de code suivant crée une identité et un cookie correspondant qui subsiste à travers les fermetures du navigateur. Les paramètres d’expiration décalés précédemment configurés sont honorés. Si le cookie expire pendant que le navigateur est fermé, le navigateur efface le cookie une fois qu’il a redémarré.

Définir <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> sur `true` dans :<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>Expiration absolue des cookies

Une heure d’expiration absolue peut être définie <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>avec. Pour créer un cookie persistant, `IsPersistent` doit également être défini. Dans le cas contraire, le cookie est créé avec une durée de vie basée sur la session et peut expirer avant ou après le ticket d’authentification qu’il contient. Lorsque `ExpiresUtc` est défini, il remplace la valeur de l' <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> option de <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>, si elle est définie.

L’extrait de code suivant crée une identité et un cookie correspondant qui dure 20 minutes. Cela ignore tous les paramètres d’expiration décalés précédemment configurés.

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [Vérifications des rôles basés sur des stratégies](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
