---
title: Composants d’application dans ASP.NET Core
author: rick-anderson
description: Partager des contrôleurs, afficher, Razor Pages et plus avec des composants d’application dans ASP.NET Core
ms.author: riande
ms.date: 05/14/2019
uid: mvc/extensibility/app-parts
ms.openlocfilehash: 4b4c8c554a7045a180b56cf9998ab1a8496cde1b
ms.sourcegitcommit: 79eeb17604b536e8f34641d1e6b697fb9a2ee21f
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/24/2019
ms.locfileid: "71207349"
---
# <a name="share-controllers-views-razor-pages-and-more-with-application-parts-in-aspnet-core"></a>Partager des contrôleurs, des affichages, des Razor Pages et plus avec des composants d’application dans ASP.NET Core

Par [Rick Anderson](https://twitter.com/RickAndMSFT)

[Affichez ou téléchargez l’exemple de code](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts) ([procédure de téléchargement](xref:index#how-to-download-a-sample))

Une *partie d’application* est une abstraction sur les ressources d’une application. Les composants de l’application permettent à ASP.NET Core de découvrir des contrôleurs, des composants de vue, des tag helpers, des Razor Pages, des sources de compilation Razor et bien plus encore. [AssemblyPart](/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) est une partie de l’application. `AssemblyPart`encapsule une référence d’assembly et expose des types et des références de compilation.

Les *fournisseurs de fonctionnalités* utilisent des composants d’application pour remplir les fonctionnalités d’une application ASP.net core. Le principal cas d’utilisation des composants d’application consiste à configurer une application pour découvrir (ou éviter le chargement) ASP.NET Core fonctionnalités d’un assembly. Par exemple, vous pouvez souhaiter partager des fonctionnalités communes entre plusieurs applications. À l’aide de composants d’application, vous pouvez partager un assembly (DLL) contenant des contrôleurs, des vues, des Razor Pages, des sources de compilation Razor, des balises d’aide et bien plus encore avec plusieurs applications. Le partage d’un assembly est préférable à la duplication du code dans plusieurs projets.

Les applications ASP.NET Core chargent <xref:System.Web.WebPages.ApplicationPart>les fonctionnalités à partir de. La <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> classe représente une partie d’application qui est sauvegardée par un assembly.

## <a name="load-aspnet-core-features"></a>Charger les fonctionnalités de ASP.NET Core

Utilisez les `ApplicationPart` classes `AssemblyPart` et pour découvrir et charger des fonctionnalités de ASP.net Core (contrôleurs, composants de vue, etc.). Le <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> effectue le suivi des composants d’application et des fournisseurs de fonctionnalités disponibles. `ApplicationPartManager`est configuré dans `Startup.ConfigureServices`:

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup.cs?name=snippet)]

Le code suivant présente une approche alternative à la configuration `ApplicationPartManager` de `AssemblyPart`à l’aide de :

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup2.cs?name=snippet)]

Les deux exemples `SharedController` de code précédents chargent à partir d’un assembly. Le `SharedController` n’est pas dans le projet de l’application. Consultez le téléchargement de l’exemple de [solution WebAppParts](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample1/WebAppParts) .

### <a name="include-views"></a>Inclure les vues

Pour inclure des vues dans l’assembly :

* Ajoutez le balisage suivant au fichier projet partagé :

  ```csproj
  <ItemGroup>
      <EmbeddedResource Include="Views\**\*.cshtml" />
  </ItemGroup>
  ```

* Ajoutez le <xref:Microsoft.Extensions.FileProviders.EmbeddedFileProvider> <xref:Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine>au :

  [!code-csharp[](./app-parts/sample1/WebAppParts/StartupViews.cs?name=snippet&highlight=3-7)]

### <a name="prevent-loading-resources"></a>Empêcher le chargement des ressources

Les composants d’application peuvent être utilisés pour *éviter* de charger des ressources dans un assembly ou un emplacement particulier. Ajoutez ou supprimez des membres <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> du regroupement pour masquer ou rendre disponibles les ressources. L’ordre des entrées dans la collection `ApplicationParts` n’est pas important. Configurez l `ApplicationPartManager` 'avant de l’utiliser pour configurer les services dans le conteneur. Par exemple, configurez le `ApplicationPartManager` avant d’appeler. `AddControllersAsServices` Appelez `Remove` sur la `ApplicationParts` collection pour supprimer une ressource.

Le code suivant utilise <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> pour supprimer `MyDependentLibrary` de l’application :[!code-csharp[](./app-parts/sample1/WebAppParts/StartupRm.cs?name=snippet)]

Le `ApplicationPartManager` comprend des parties pour :

* Assembly de l’application et assemblys dépendants.
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

## <a name="application-feature-providers"></a>Fournisseurs de fonctionnalités d’application

Les fournisseurs de fonctionnalités d’application examinent les parties de l’application et fournissent des fonctionnalités pour ces parties. Il existe des fournisseurs de fonctionnalités intégrés pour les fonctionnalités de ASP.NET Core suivantes :

* [Contrôleurs](/dotnet/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [Les Tag Helpers](/dotnet/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [Les composants de vues](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

Les fournisseurs de fonctionnalités héritent de <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1>, où `T` correspond au type de la fonctionnalité. Les fournisseurs de fonctionnalités peuvent être implémentés pour l’un des types de fonctionnalités précédemment listés. L’ordre des fournisseurs de fonctionnalités dans `ApplicationPartManager.FeatureProviders` le peut avoir un impact sur le comportement de l’exécution. Les fournisseurs ajoutés ultérieurement peuvent réagir aux actions effectuées par les fournisseurs précédemment ajoutés.

### <a name="generic-controller-feature"></a>Fonctionnalité de contrôleur générique

ASP.NET Core ignore les [contrôleurs génériques](/dotnet/csharp/programming-guide/generics/generic-classes). Un contrôleur générique a un paramètre de type (par exemple `MyController<T>`,). L’exemple suivant ajoute des instances de contrôleur génériques pour une liste de types spécifiée :

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericControllerFeatureProvider.cs?name=snippet)]

Les types sont définis dans `EntityTypes.Types`:

[!code-csharp[](./app-parts/sample2/AppPartsSample/Models/EntityTypes.cs?name=snippet)]

Le fournisseur de fonctionnalités est ajouté dans `Startup` :

[!code-csharp[](./app-parts/sample2/AppPartsSample/Startup.cs?name=snippet)]

Les noms de contrôleurs génériques utilisés pour le routage se présentent sous la forme *GenericController' 1 [widget]* au lieu du *widget*. L’attribut suivant modifie le nom pour qu’il corresponde au type générique utilisé par le contrôleur :

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericControllerNameConvention.cs)]

La classe `GenericController` :

[!code-csharp[](./app-parts/sample2/AppPartsSample/GenericController.cs)]

Par exemple, la demande d’une `https://localhost:5001/Sprocket` URL de entraîne la réponse suivante :

```text
Hello from a generic Sprocket controller.
```

### <a name="display-available-features"></a>Afficher les fonctionnalités disponibles

Les fonctionnalités disponibles pour une application peuvent être énumérées en demandant un `ApplicationPartManager` par [injection de dépendances](../../fundamentals/dependency-injection.md):

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

L' [exemple Download](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample2) utilise le code précédent pour afficher les fonctionnalités de l’application :

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```
