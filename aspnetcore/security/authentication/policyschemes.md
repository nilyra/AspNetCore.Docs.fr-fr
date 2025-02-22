---
title: Jeux de stratégie dans ASP.NET Core
author: rick-anderson
description: Modèles de stratégie d’authentification facilitent l’ont un schéma d’authentification logique unique
ms.author: riande
ms.date: 02/28/2019
uid: security/authentication/policyschemes
ms.openlocfilehash: be03f349455c673b0739935ad20e596325c8cb74
ms.sourcegitcommit: 8516b586541e6ba402e57228e356639b85dfb2b9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/11/2019
ms.locfileid: "67815282"
---
# <a name="policy-schemes-in-aspnet-core"></a>Jeux de stratégie dans ASP.NET Core

Modèles de stratégie d’authentification facilitent l’ont un schéma d’authentification logiques seule potentiellement utiliser plusieurs approches. Par exemple, un jeu de stratégie peut utiliser l’authentification Google pour défis et l’authentification des cookies pour tout le reste. Les jeux de stratégie d’authentification permettent de :

* Très simple de transférer une action de l’authentification vers un autre schéma.
* Avant de façon dynamique en fonction de la demande.

Tous les schémas d’authentification qui utilisez dérivés <xref:Microsoft.AspNetCore.Authentication.AuthenticationSchemeOptions> et associé [ `AuthenticationHandler<TOptions>` ](/dotnet/api/microsoft.aspnetcore.authentication.authenticationhandler-1):

* Sont automatiquement des jeux de stratégie dans ASP.NET Core 2.1 et versions ultérieures.
* Peut être activée via la configuration des options de ce modèle.

[!code-csharp[sample](policyschemes/samples/AuthenticationSchemeOptions.cs?name=snippet)]

## <a name="examples"></a>Exemples

L’exemple suivant montre un schéma de niveau supérieur qui combine les schémas de niveau inférieur. L’authentification Google est utilisée pour les défis, et l’authentification des cookies est utilisée pour tout le reste :

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet1)]

L’exemple suivant active la sélection dynamique de schémas de base de chaque demande. Autrement dit, comment combiner les cookies et l’authentification d’API :

 <!-- REVIEW, missing If set in public Func<HttpContext, string> ForwardDefaultSelector -->

[!code-csharp[sample](policyschemes/samples/Startup.cs?name=snippet2)]
