---
title: Disposition dans ASP.NET Core
author: ardalis
description: Apprenez à utiliser des dispositions communes, à partager des directives et à exécuter le code commun avant d’afficher les vues dans une application ASP.NET Core.
ms.author: riande
ms.date: 07/30/2019
uid: mvc/views/layout
ms.openlocfilehash: 9a7b8003b24329f6e9cbd349ee47f6844b7c5f6d
ms.sourcegitcommit: e6bd2bbe5683e9a7dbbc2f2eab644986e6dc8a87
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/03/2019
ms.locfileid: "70238031"
---
# <a name="layout-in-aspnet-core"></a>Disposition dans ASP.NET Core

Article rédigé par [Steve Smith](https://ardalis.com/) et [Dave Brock](https://twitter.com/daveabrock)

Les pages et les vues ont souvent des éléments visuels et programmatiques en commun. Cet article montre comment :

* Utiliser des dispositions communes.
* Partager des directives.
* Exécuter du code commun avant d’afficher des pages ou des vues.

Ce document traite des dispositions pour les deux approches différentes d’ASP.NET Core MVC : Razor Pages et les contrôleurs avec vues. Pour cette rubrique, les différences sont minimes :

* Razor Pages se trouve dans le dossier *Pages*.
* Les contrôleurs avec vues utilisent un dossier *Views* pour les vues.

## <a name="what-is-a-layout"></a>Qu’est-ce qu’une disposition ?

La plupart des applications web ont une disposition commune pour offrir aux utilisateurs une expérience homogène quand ils naviguent de page en page. En général, la disposition inclut des éléments d’interface utilisateur communs à toute l’application, tels que l’en-tête, des éléments de menu ou de navigation et le pied de page.

![Exemple de disposition de page](layout/_static/page-layout.png)

Les structures HTML communes comme les scripts et les feuilles de style sont aussi fréquemment utilisées par bon nombre de pages dans une application. Tous ces éléments partagés peuvent être définis dans un fichier de *disposition*, qui peut ensuite être référencé par n’importe quelle vue utilisée dans l’application. Les dispositions réduisent les doublons de code dans les vues.

Par convention, la disposition par défaut d’une application ASP.NET Core se nomme *_Layout.cshtml*. Les fichiers de disposition pour les projets ASP.NET Core créés avec les modèles sont les suivants :

* Pages Razor : *Pages/Shared/_Layout.cshtml*

  ![Dossier Pages dans l’Explorateur de solutions](layout/_static/rp-web-project-views.png)

* Contrôleur avec vues : *Views/Shared/_Layout.cshtml*

  ![Dossier Views dans l’Explorateur de solutions](layout/_static/mvc-web-project-views.png)

La disposition définit un modèle général pour les vues dans l’application. Les applications ne nécessitent pas de disposition. Les applications peuvent définir plusieurs dispositions, avec des vues différentes spécifiant des dispositions différentes.

Le code suivant montre le fichier de disposition pour un projet créé avec un modèle, avec un contrôleur et des vues :

[!code-cshtml[](~/common/samples/WebApplication1/Views/Shared/_Layout.cshtml?highlight=44,72)]

## <a name="specifying-a-layout"></a>Spécification d’une disposition

Les vues Razor ont une propriété `Layout`. Chaque vue spécifie une disposition en définissant cette propriété :

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml?highlight=2)]

La disposition spécifiée peut utiliser un chemin complet (par exemple, */Pages/Shared/_Layout.cshtml* ou */Views/Shared/_Layout.cshtml*) ou un nom partiel (exemple : `_Layout`). Quand un nom partiel est fourni, le moteur de vue Razor recherche le fichier de disposition en utilisant son processus de détection habituel. Le dossier où se trouve la méthode de gestionnaire (ou contrôleur) est parcouru en premier, suivi du dossier *Shared*. Ce processus de détection est le même que celui utilisé pour détecter les [vues partielles](xref:mvc/views/partial#partial-view-discovery).

Par défaut, chaque disposition doit appeler `RenderBody`. À chaque appel de `RenderBody`, le contenu de la vue est affiché.

<a name="layout-sections-label"></a>
<!-- https://stackoverflow.com/questions/23327578 -->
### <a name="sections"></a>Sections

Une disposition peut éventuellement faire référence à une ou plusieurs *sections*, en appelant `RenderSection`. Les sections sont un moyen d’organiser certains éléments dans la page. Chaque appel à `RenderSection` peut spécifier si cette section est obligatoire ou facultative :

```html
@section Scripts {
    @RenderSection("Scripts", required: false)
}
```

Si une section obligatoire est introuvable, une exception est levée. Chacune des vues spécifient le contenu à afficher dans une section à l’aide de la syntaxe Razor `@section`. Si une page ou vue définit une section, elle doit être affichée (sinon, une erreur se produit).

Exemple de définition `@section` dans une vue Razor Pages :

```html
@section Scripts {
     <script type="text/javascript" src="/scripts/main.js"></script>
}
```

Dans le code précédent, *scripts/main.js* est ajouté à la section `scripts` sur une page ou vue. Il est possible que les autres pages ou vues de la même application ne nécessitent pas ce script et ne définissent pas de section de scripts.

Le balisage suivant utilise le [Tag Helper Partial](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper) pour afficher *_ValidationScriptsPartial.cshtml* :

```html
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

Le balisage précédent était le résultat de la [génération d’un modèle automatique d’identité](xref:security/authentication/scaffold-identity).

Les sections définies dans une page ou vue sont disponibles uniquement dans sa page de disposition la plus proche. Elles ne peuvent pas être référencées à partir de vues partielles, de composants de vue ou d’autres parties du système de vue.

### <a name="ignoring-sections"></a>Ignorer des sections

Par défaut, le corps et toutes les sections dans une page de contenu doivent intégralement être affichés par la page de disposition. Le moteur de vue Razor s’assure que c’est bien le cas en vérifiant que le corps et chaque section ont été affichés.

Pour indiquer au moteur de vue d’ignorer le corps ou les sections, appelez les méthodes `IgnoreBody` et `IgnoreSection`.

Le corps et toutes les sections dans une page Razor doivent être soit affichés, soit ignorés.

<a name="viewimports"></a>

## <a name="importing-shared-directives"></a>Importation de directives partagées

Les vues et les pages peuvent utiliser des directives Razor pour importer des espaces de noms et utiliser [l’injection de dépendances](dependency-injection.md). Les directives partagées par plusieurs vues peuvent être spécifiées dans un fichier *_ViewImports.cshtml* commun. Le fichier `_ViewImports` prend en charge les directives suivantes :

* `@addTagHelper`
* `@removeTagHelper`
* `@tagHelperPrefix`
* `@using`
* `@model`
* `@inherits`
* `@inject`

Le fichier ne prend pas en charge les autres fonctionnalités Razor, telles que les fonctions et les définitions de section.

Exemple de fichier `_ViewImports.cshtml` :

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewImports.cshtml)]

Le fichier *_ViewImports.cshtml* pour une application ASP.NET Core MVC est généralement créé dans le dossier *Pages* (ou *Views*). Un fichier *_ViewImports.cshtml* peut être créé dans un autre dossier ; dans ce cas, il s’applique uniquement aux pages ou vues contenues dans ce dossier et ses sous-dossiers. Les fichiers `_ViewImports` sont traités à partir de la racine, puis pour chaque dossier conduisant à l’emplacement de la page ou vue elle-même. Les paramètres `_ViewImports` spécifiés au niveau de la racine peuvent être remplacés au niveau du dossier.

Par exemple, supposons que :

* Le fichier *_ViewImports.cshtml* au niveau de la racine inclut `@model MyModel1` et `@addTagHelper *, MyTagHelper1`.
* Le fichier *_ViewImports.cshtml* dans un sous-dossier inclut `@model MyModel2` et `@addTagHelper *, MyTagHelper2`.

Les pages et vues dans le sous-dossier ont accès aux Tag Helpers et au modèle `MyModel2`.

Si la hiérarchie des fichiers comprend plusieurs fichiers *_ViewImports.cshtml*, le comportement combiné des directives est le suivant :

* `@addTagHelper`, `@removeTagHelper` : les deux directives sont exécutées, dans l’ordre
* `@tagHelperPrefix` : la directive la plus proche de la vue se substitue aux autres
* `@model` : la directive la plus proche de la vue se substitue aux autres
* `@inherits` : la directive la plus proche de la vue se substitue aux autres
* `@using` : toutes les directives sont incluses ; les doublons sont ignorés
* `@inject` : pour chaque propriété, la plus proche de la vue se substitue aux autres propriétés ayant le même nom

<a name="viewstart"></a>

## <a name="running-code-before-each-view"></a>Exécution du code avant chaque vue

Le code qui doit s’exécuter avant chaque vue ou page doit être placé dans le fichier *_ViewStart.cshtml*. Par convention, le fichier *_ViewStart.cshtml* se trouve dans le dossier *Pages* (ou *Views*). Les instructions contenues dans *_ViewStart.cshtml* sont exécutées avant chaque vue complète (donc hors dispositions et vues partielles). Comme [ViewImports.cshtml](xref:mvc/views/layout#viewimports), *_ViewStart.cshtml* est hiérarchique. Si un fichier *_ViewStart.cshtml* est défini dans le dossier des vues ou des pages, il est exécuté après celui qui est défini à la racine du dossier *Pages* (ou *Views*) (le cas échéant).

Exemple de fichier *_ViewStart.cshtml* :

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml)]

Le fichier ci-dessus spécifie que toutes les vues doivent utiliser la disposition *_Layout.cshtml*.

*_ViewStart.cshtml* et *_ViewImports.cshtml* **ne sont pas** généralement placés dans le dossier */Pages/Shared* (ou */Views/Shared*). Les versions de ces fichiers qui sont au niveau de l’application doivent être placées directement dans le dossier */Pages* (ou */Views*).
