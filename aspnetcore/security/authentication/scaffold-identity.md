---
title: Identité de l’échafaudage dans les projets ASP.NET Core
author: rick-anderson
description: Découvrez comment générer une structure d’identité dans un projet ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/24/2018
uid: security/authentication/scaffold-identity
ms.openlocfilehash: f3ae089d344d95ed84c9720ab4ba2c697400901e
ms.sourcegitcommit: dc96d76f6b231de59586fcbb989a7fb5106d26a8
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2019
ms.locfileid: "71703773"
---
# <a name="scaffold-identity-in-aspnet-core-projects"></a>Identité de l’échafaudage dans les projets ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core 2,1 et versions ultérieures fournissent [ASP.net Core identité](xref:security/authentication/identity) sous forme de [bibliothèque de classes Razor](xref:razor-pages/ui-class). Les applications qui incluent une identité peuvent appliquer l’échafaudage pour ajouter de manière sélective le code source contenu dans la bibliothèque de classes Razor d’identité (RCL). Vous pouvez souhaiter générer le code source afin de pouvoir modifier le code et changer le comportement. Par exemple, vous pouvez demander au générateur de modèles automatique de générer le code utilisé dans l’inscription. Le code généré est prioritaire sur le même code dans la bibliothèque de classes Razor d’identité. Pour obtenir le contrôle total de l’interface utilisateur et ne pas utiliser le RCL par défaut, consultez la section [créer une source d’interface utilisateur d’identité complète](#full).

Les applications qui n’incluent **pas** l’authentification peuvent appliquer l’échafaudage pour ajouter le package d’identité RCL. Vous pouvez sélectionner le code d’identité à générer.

Bien que le générateur de modèles génère la plus grande partie du code nécessaire, vous devez mettre à jour votre projet pour terminer le processus. Ce document explique les étapes nécessaires pour effectuer une mise à jour de l’échafaudage d’identité.

Lorsque l’échafaudage d’identité est exécuté, un fichier *ScaffoldingReadme. txt* est créé dans le répertoire du projet. Le fichier *ScaffoldingReadme. txt* contient des instructions générales sur ce qui est nécessaire pour effectuer la mise à jour de l’échafaudage d’identité. Ce document contient des instructions plus complètes que le fichier *ScaffoldingReadme. txt* .

Nous vous recommandons d’utiliser un système de contrôle de code source qui affiche les différences entre les fichiers et vous permet d’annuler les modifications. Inspectez les modifications après avoir exécuté l’échafaudage d’identité.

> [!NOTE]
> Les services sont requis lors de l’utilisation de [l’authentification à deux facteurs](xref:security/authentication/identity-enable-qrcodes), la [confirmation de compte et la récupération de mot de passe](xref:security/authentication/accconfirm), ainsi que d’autres fonctionnalités de sécurité avec identité. Les services ou stubs de service ne sont pas générés lors de la génération de modèles automatique d’identité. Les services pour activer ces fonctionnalités doivent être ajoutés manuellement. Par exemple, consultez [exiger une confirmation par courrier électronique](xref:security/authentication/accconfirm#require-email-confirmation).

## <a name="scaffold-identity-into-an-empty-project"></a>Identité de l’échafaudage dans un projet vide

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

Ajoutez les appels en surbrillance suivants à la classe `Startup` :

[!code-csharp[](scaffold-identity/sample/StartupEmpty.cs?name=snippet1&highlight=5,20-23)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

## <a name="scaffold-identity-into-a-razor-project-without-existing-authorization"></a>Identité de l’échafaudage dans un projet Razor sans autorisation existante

<!--  Updated for 3.0
set projNam=RPnoAuth
set projType=webapp

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet restore
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

L’identité est configurée dans *Areas/Identity/IdentityHostingStartup. cs*. Pour plus d’informations, consultez [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration).

<a name="efm"></a>

### <a name="migrations-useauthentication-and-layout"></a>Migrations, UseAuthentication et disposition

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

<a name="useauthentication"></a>

### <a name="enable-authentication"></a>Activer l’authentification

Dans la méthode `Configure` de la classe `Startup`, appelez [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication?view=aspnetcore-2.0#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_) après `UseStaticFiles` :

[!code-csharp[](scaffold-identity/sample/StartupRPnoAuth.cs?name=snippet1&highlight=29)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

### <a name="layout-changes"></a>Modifications de la disposition

Facultatif : Ajoutez la connexion partielle (`_LoginPartial`) au fichier de disposition :

[!code-html[Main](scaffold-identity/sample/_Layout.cshtml?highlight=37)]

## <a name="scaffold-identity-into-a-razor-project-with-authorization"></a>Identité de l’échafaudage dans un projet Razor avec autorisation

<!--
Use >=2.1: dotnet new webapp -au Individual -o RPauth
Use = 2.0: dotnet new razor -au Individual -o RPauth
uld option: Use Local DB, not SQLite

dotnet new webapp -au Individual -uld -o RPauth
cd RPauth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files Account.Register
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]
Certaines options d’identité sont configurées dans *Areas/Identity/IdentityHostingStartup. cs*. Pour plus d’informations, consultez [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration).

## <a name="scaffold-identity-into-an-mvc-project-without-existing-authorization"></a>Identité de l’échafaudage dans un projet MVC sans autorisation existante

<!--
set projNam=MvcNoAuth
set projType=mvc
set version=2.1.0

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -v %version%
dotnet restore
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

Facultatif : Ajoutez la connexion partielle (`_LoginPartial`) au fichier *Views/Shared/_ Layout. cshtml* :

[!code-html[](scaffold-identity/sample/_LayoutMvc.cshtml?highlight=37)]

* Déplacez le fichier *pages/Shared/_LoginPartial. cshtml* vers *Views/Shared/_LoginPartial. cshtml*

L’identité est configurée dans *Areas/Identity/IdentityHostingStartup. cs*. Pour plus d’informations, consultez IHostingStartup.

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

Appelez [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication?view=aspnetcore-2.0#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_) après `UseStaticFiles` :

[!code-csharp[](scaffold-identity/sample/StartupMvcNoAuth.cs?name=snippet1&highlight=23)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

## <a name="scaffold-identity-into-an-mvc-project-with-authorization"></a>Identité de l’échafaudage dans un projet MVC avec autorisation

<!--
dotnet new mvc -au Individual -o MvcAuth
cd MvcAuth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
dotnet aspnet-codegenerator identity -dc MvcAuth.Data.ApplicationDbContext --files Account.Register
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

Supprimez le dossier *pages/Shared* et les fichiers de ce dossier.

<a name="full"></a>

## <a name="create-full-identity-ui-source"></a>Créer une source d’interface utilisateur d’identité complète

Pour conserver le contrôle total de l’interface utilisateur de l’identité, exécutez le générateur de modèles d’identité et sélectionnez **remplacer tous les fichiers**.

Le code en surbrillance suivant montre les modifications pour remplacer l’interface utilisateur d’identité par défaut par l’identité dans une application Web ASP.NET Core 2,1. Vous pouvez le faire pour avoir le contrôle total de l’interface utilisateur de l’identité.

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet1&highlight=13-14,17-999)]

L’identité par défaut est remplacée dans le code suivant :

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet2)]

Le code suivant définit les [LoginPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.loginpath), [LogoutPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.logoutpath)et [AccessDeniedPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.accessdeniedpath):

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet3)]

Inscrire une implémentation `IEmailSender`, par exemple :

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet4)]

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet)]

## <a name="additional-resources"></a>Ressources supplémentaires

* [Modifications apportées au code d’authentification pour ASP.NET Core 2,1 et versions ultérieures](xref:migration/20_21#changes-to-authentication-code)
