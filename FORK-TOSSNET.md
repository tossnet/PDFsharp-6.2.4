# Fork tossnet.PdfSharp (basé sur PDFsharp 6.2.4)

Documentation technique du fork privé de PDFsharp / MigraDoc maintenu pour des applications .NET internes.

Dernière mise à jour : 14 novembre 2026.

## 1. Pourquoi ce fork

- PDFsharp 6.2.4 (publiée le 6 janvier 2026) est la **dernière version sous licence MIT**.
- Le dépôt amont `empira/PDFsharp` (branche `master`, tag `v6.5.0`, branche `release/6.5.0`, tags `v7.x`) est passé en **"AGPL Dual License"** : AGPL v3, ou licence commerciale payante pour un usage en projet fermé. Constaté avec `git diff v6.2.4 upstream/master -- LICENSE` et `git show v6.5.0:LICENSE`.
- Une licence MIT déjà accordée sur du code publié n'est pas retirée : le code du tag `v6.2.4` reste utilisable, modifiable et redistribuable, à condition de conserver le fichier `LICENSE` et le copyright empira.
- Objectif : garder une base MIT maintenable (correctifs de sécurité des dépendances), sans dépendre d'un package externe qui change de licence.

Ce document n'est pas un avis juridique.

## 2. Règles à ne jamais enfreindre

1. Ne **jamais** fusionner, rebaser ni cherry-picker depuis `upstream/master`, `release/6.5.0`, `v6.5.x` ou `v7.x`.
2. Ne **jamais** cliquer sur "Sync fork" sur GitHub (il ramènerait le code AGPL).
3. Ne jamais copier de code venant de ces versions, ni d'un autre projet sans vérifier sa licence.
4. Ne jamais modifier `LICENSE` ni les mentions de copyright empira.
5. Avant toute mise à jour de dépendance : vérifier sa licence.
6. Ne jamais référencer en parallèle le package officiel `PDFsharp` et `tossnet.PdfSharp` (doublon de types, y compris par dépendance indirecte).

Ces règles sont aussi dans `CLAUDE.md` à la racine du dépôt.

## 3. Dépôt

| Élément | Valeur |
|---|---|
| Fork GitHub | `https://github.com/tossnet/PDFsharp-6.2.4` |
| Branche par défaut | `maintenance/6.2.4` |
| Base | tag `v6.2.4` |
| Remote d'origine | `upstream` = `https://github.com/empira/PDFsharp.git` (pour les tags uniquement) |
| Tags du fork | `v6.2.5` |

Historique des commits propres au fork :

| Commit | Contenu |
|---|---|
| `502ee3d` | Mise à jour `System.Security.Cryptography.Pkcs` 8.0.1 vers 8.0.2 (2 CVE de déni de service) |
| `eadece9` | Ajout de `CLAUDE.md` (garde-fous licence et mises à jour) |
| `de6058d` | Renommage des packages NuGet (tag `v6.2.5`) |

### Mise à jour de sécurité (commit 502ee3d)

Un seul paquet vulnérable détecté par `dotnet list package --vulnerable --include-transitive` : `System.Security.Cryptography.Pkcs` 8.0.1.

| Avis | CVE | Gravité | Problème | Corrigé en |
|---|---|---|---|---|
| GHSA-846f-f764-76mx | CVE-2026-84008 | Élevée (7,5) | Déni de service : `SignedCms.CheckSignature` peut épuiser le CPU | 8.0.2 |
| GHSA-2fmv-rv2c-59x7 | CVE-2026-70162 | Moyenne (5,9) | Déni de service sous Windows : `EnvelopedCms.Decode` peut consommer trop de CPU | 8.0.2 |

Changement : une ligne dans `src/Directory.Packages.props` (`Cryptography_PackageVersion` passe à 8.0.2). Licence du paquet : MIT (Microsoft). Ces failles concernent la signature et le déchiffrement de PDF.

Tests (état au moment de la mise à jour) : tous les tests PdfSharp passent. Échecs préexistants, sans lien avec la mise à jour (vérifié sur 8.0.1) :
- `MigraDoc.Tests.TextTests.DecimalTabulatorTest` (position attendue 130,88, obtenue 131,66), probablement lié à la culture fr-FR ou aux polices ;
- suites `GrammarByExample` et `GBE-Runner` : écrivent dans `D:\GBE-Output\` (chemin codé en dur).

## 4. Packages publiés

| Package | Version | Contenu |
|---|---|---|
| `tossnet.PdfSharp` | 6.2.5 | Moteur PDFsharp (build Core) |
| `tossnet.PdfSharp.MigraDoc` | 6.2.5 | MigraDoc (build Core), dépend de `tossnet.PdfSharp` |

Choix de conception :

- **Seul le build Core est publié** (net8.0, net9.0, net10.0, netstandard2.0). Pas de variantes `-gdi` ni `-wpf`. Les projets `-wpf` et `-gdi` du dépôt ne sont pas dans les packages. Ne pas supprimer `PdfSharp.WPFonts` : ce n'est pas du WPF, c'est un projet de polices inclus dans le package.
- **Seul l'identifiant du package change.** Les noms d'assembly et les namespaces (`using PdfSharp...`, `using MigraDoc...`) sont inchangés : le code consommateur ne change pas. Conséquence : ne jamais référencer les deux packages (officiel et fork) ensemble.
- Licence `MIT` et copyright empira conservés dans les `.nuspec`.

Fichiers modifiés (dans `src/foundation/nuget/src/`) :

| Fichier | Modification |
|---|---|
| `PDFsharp.NuGet/PDFsharp.NuGet.csproj` | `NuGetId` = `tossnet.PdfSharp`, `NuGetTitle` |
| `MigraDoc.NuGet/MigraDoc.NuGet.csproj` | `NuGetId` = `tossnet.PdfSharp.MigraDoc`, `NuGetTitle` |
| `PDFsharp.NuGet/PDFsharp.NuGet.nuspec` | URL et branche du dépôt |
| `MigraDoc.NuGet/MigraDoc.NuGet.nuspec` | URL et branche du dépôt, 4 dépendances `PDFsharp` remplacées par `tossnet.PdfSharp` (une par groupe de framework) |

Point d'attention : dans le nuspec de MigraDoc, la dépendance `<dependency id="tossnet.PdfSharp" version="$version$" />` signifie "cette version ou plus". Une dépendance restée sur `PDFsharp` aurait pu attirer le package officiel, donc potentiellement une version AGPL.

Remarque cosmétique connue : le titre NuGet affiche "6.2.4 (fork tossnet)" alors que la version est 6.2.5. Sans effet technique, à corriger lors de la prochaine version.

## 5. Construire une version

Prérequis : PowerShell 7, SDK .NET récent, et le dossier `assets` (polices, utile aux tests), obtenu avec `dev/download-assets.ps1` (script officiel du projet, le dossier est ignoré par git).

Le numéro de version est calculé par **GitVersion** à partir des tags. La branche `maintenance/6.2.4` n'est pas reconnue par les règles de `gitversion.yml` : le build produit un numéro inattendu (observé : `6.2.6-maintenance-6-2-4.3` au lieu de `6.2.5`). Forcer le numéro avec `-p:Version=...` ne fonctionne pas non plus : `-p:DisableGitVersionTask=true` désactive la génération de la classe `GitVersionInformation` dont `PdfSharp.Shared` a besoin pour compiler.

Méthode qui fonctionne : construire depuis une **branche locale nommée `release/x.y.z`** posée sur le commit tagué (leurs règles font alors prendre la version uniquement du tag).

```powershell
# 1. Commiter les changements sur maintenance/6.2.4, puis poser le tag
git tag v6.2.5

# 2. Branche de build locale (ne jamais la pousser)
git switch -c release/6.2.5

# 3. Construire les deux packages
dotnet build src\foundation\nuget\src\PDFsharp.NuGet\PDFsharp.NuGet.csproj -c Release
dotnet build src\foundation\nuget\src\MigraDoc.NuGet\MigraDoc.NuGet.csproj -c Release

# 4. Sauvegarder le code et le tag, puis revenir à la branche normale
git switch maintenance/6.2.4
git push
git push origin v6.2.5
```

Fichiers produits (ignorer les `.nupkg` des dossiers `Debug`, ce sont des restes de builds de test aux noms officiels) :

- `src\foundation\nuget\src\PDFsharp.NuGet\bin\Release\tossnet.PdfSharp.6.2.5.nupkg`
- `src\foundation\nuget\src\MigraDoc.NuGet\bin\Release\tossnet.PdfSharp.MigraDoc.6.2.5.nupkg`

### Contrôle avant publication

Un `.nupkg` est un zip : le copier en `.zip`, l'extraire, lire le `.nuspec`. Vérifier :

1. `id` et `version` corrects ;
2. licence `MIT` et copyright empira ;
3. `System.Security.Cryptography.Pkcs` en 8.0.2 dans tous les groupes ;
4. URL du dépôt = le fork ;
5. DLL `PdfSharp.dll` présentes pour net8.0, net9.0, net10.0 et netstandard2.0 ;
6. pour MigraDoc : dépendance `tossnet.PdfSharp` en `6.2.5` (aucune dépendance sur `PDFsharp`) et uniquement des DLL `MigraDoc.*` dans `lib`.

Ces contrôles ont été faits pour la 6.2.5 : tous conformes.

## 6. Numérotation

- Les numéros du fork sont indépendants de ceux du PDFsharp officiel (l'identifiant du package est différent).
- Règle : n'incrémenter que le **dernier chiffre** (6.2.5, 6.2.6, 6.2.7...), même pour des nouveautés, pour ne jamais ressembler à la 6.5.0 officielle (AGPL).
- Si un gros changement doit être marqué : 6.3.0, 6.4.0, puis sauter la 6.5 (6.6.0).

## 7. Hébergement : feed NuGet privé (Azure Artifacts)

| Élément | Valeur |
|---|---|
| Feed | `<NOM-DU-FEED>` (portée organisation) |
| Source NuGet | `https://pkgs.dev.azure.com/<ORGANISATION>/_packaging/<NOM-DU-FEED>/nuget/v3/index.json` |
| Visibilité | privée (personnes de l'organisation) |
| Coût | 2 GiB gratuits par organisation, puis environ 2 $ par GiB. Un jeu de deux packages pèse environ 6 Mo. Au-delà de 2 GiB sans facturation activée : lecture seule |

Prérequis sur le PC : **Azure Artifacts Credential Provider** (script d'installation officiel Microsoft, puis vérifier le dossier `%UserProfile%\.nuget\plugins\netcore`).

Publication :

```powershell
dotnet nuget push <chemin>\tossnet.PdfSharp.6.2.5.nupkg --source "<URL-DU-FEED>" --api-key az --interactive
dotnet nuget push <chemin>\tossnet.PdfSharp.MigraDoc.6.2.5.nupkg --source "<URL-DU-FEED>" --api-key az --interactive
```

`--api-key az` est une valeur quelconque exigée par l'outil, pas un secret. Publier **PdfSharp avant MigraDoc**. Une version publiée ne se modifie pas : toute correction est une nouvelle version.

Publier sur nuget.org n'a pas été retenu : irréversible (un package ne peut qu'être masqué), public, et incertitude sur l'usage du nom "PdfSharp" dans un identifiant public.

## 8. Utilisation dans un projet consommateur

### nuget.config (à la racine, à côté de la solution)

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="prive" value="<URL-DU-FEED>" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="nuget.org"><package pattern="*" /></packageSource>
    <packageSource key="prive"><package pattern="tossnet.*" /></packageSource>
  </packageSourceMapping>
</configuration>
```

Ne pas copier tel quel le `nuget.config` proposé par la page du feed : son `<clear />` retirerait nuget.org. Le mapping force `tossnet.*` à venir uniquement du feed privé.

### Références

```xml
<PackageReference Include="tossnet.PdfSharp" Version="[6.2.5]" />
<!-- Dans un projet d'export PDF dédié, ajouter PrivateAssets="compile" pour interdire "using PdfSharp" dans le reste de la solution -->
```

Versions épinglées entre crochets (version exacte). Ne jamais utiliser de plage (`6.*`) : risque de retomber sur une version AGPL.

### Pipeline d'intégration continue

Si le pipeline recompile tout sur un agent, il doit lire le feed.

1. Ajouter `- task: NuGetAuthenticate@1` après `UseDotNet@2`, avant toute commande `dotnet` de restauration ou de build, dans chaque stage.
2. Donner le droit de lecture du feed (Feed settings, Permissions, rôle *Feed and Upstream Reader (Collaborator)*) aux identités de build. Observé : l'identité de build globale de l'organisation ne suffisait pas, l'erreur `403 ... You need to have 'ReadPackages'` désignait l'identité de build propre au projet, qu'il a fallu ajouter.
3. Protéger la production : `condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master'))` sur le stage de déploiement (sinon un lancement manuel sur une autre branche déploie en production).

## 9. État actuel

| Projet | État |
|---|---|
| Projet A (Blazor WebAssembly hébergé) | Migré et déployé en production avec `tossnet.PdfSharp` 6.2.5. Export PDF isolé derrière une interface (un seul projet référence PdfSharp). 364 tests au moment de la migration : 363 réussis, 1 ignoré (générateur des PDF de référence), 0 échec. Un PDF exporté indique "PDFsharp 6.2.5" comme créateur et producteur |
| Projet B (Blazor Server) | À faire. Utilise MigraDoc : il faudra les deux packages |

Remarque : le champ "Créateur" des PDF est rempli automatiquement par la bibliothèque ("PDFsharp 6.2.5 (www.pdfsharp.com)"). La licence MIT impose de conserver son texte et le copyright dans le dépôt et les packages, pas de l'afficher dans les PDF.

## 10. Procédure pour une prochaine version (ex. 6.2.6)

1. Travailler sur `maintenance/6.2.4`.
2. Détecter : `dotnet list package --vulnerable --include-transitive` et `dotnet list package --outdated` (versions centralisées dans `src/Directory.Packages.props`).
3. Pour chaque mise à jour : vérifier la licence, appliquer, `dotnet build`, `dotnet test`.
4. Commit avec un message précis (quoi, de quelle version à quelle version, pourquoi).
5. Tag `v6.2.6`, branche locale `release/6.2.6`, build des deux packages, contrôle du contenu (section 5).
6. Pousser branche et tag sur le fork, puis publier les deux packages (PdfSharp d'abord).
7. Dans chaque projet consommateur : changer `[6.2.5]` en `[6.2.6]`, restaurer avec `--interactive`, lancer les tests, tester un export réel.

## 11. Pistes écartées ou reportées

- Retirer les cibles `netstandard2.0` et `net8.0` du package : gain nul sur la taille d'une application (une seule cible est utilisée), mais casse la restauration pour les projets sur ces cibles et alourdit le diff avec l'original.
- Supprimer les projets `-wpf` et `-gdi` du dépôt : sans effet sur les packages, léger gain de lisibilité et d'audit. À faire dans un commit séparé, en reconstruisant les packages pour vérifier un contenu identique.
- Écrire un générateur PDF maison : coût élevé (polices, images, compatibilité) sans besoin actuel.
- Alternatives étudiées : QuestPDF (licence Community gratuite sous 1 M USD de chiffre d'affaires, natif Skia donc risqué en WebAssembly), OfficeIMO (MIT, PDF sans dépendance tierce, non évalué en WebAssembly), PdfPinata (fork de PdfSharpCore, versions 0.x). L'interface d'export PDF du projet A permet de les essayer plus tard sans refonte.