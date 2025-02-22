Le tableau suivant détaille les paramètres du générateur de code ASP.NET Core :

| Paramètre               | Description|
| ----------------- | ------------ |
| -m  | Nom du modèle. |
| -dc  | Contexte de données. |
| -udl | Utiliser la disposition par défaut. |
| --relativeFolderPath | Chemin du dossier de sortie relatif pour créer les fichiers. |
| --useDefaultLayout | La disposition par défaut doit être utilisée pour les vues. |
| --referenceScriptLibraries | Ajoute `_ValidationScriptsPartial` aux pages Modifier et Créer. |

Utilisez le commutateur `h` pour obtenir de l’aide sur la commande `aspnet-codegenerator controller` :

```dotnetcli
dotnet aspnet-codegenerator controller -h
```

Pour plus d’informations, consultez [dotnet aspnet-codegenerator](xref:fundamentals/tools/dotnet-aspnet-codegenerator).
