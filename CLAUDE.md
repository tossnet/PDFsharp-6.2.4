# PDFsharp 6.2.4 (fork privé)

## Contexte
Fork du tag v6.2.4 de empira/PDFsharp, la dernière version sous licence MIT.
Les versions 6.5 et 7.x sont sous AGPL ou licence commerciale : interdites ici.
Usage : package NuGet privé

## Règles absolues
- Ne jamais fusionner, rebaser ni cherry-picker depuis upstream/master, release/6.5.0 ni les tags v6.5.x et v7.x.
- Ne jamais copier de code venant de ces versions, ni d'un autre projet sans en vérifier la licence.
- Ne jamais modifier le fichier LICENSE ni les mentions de copyright empira.
- Avant toute mise à jour de dépendance, vérifier sa licence et me la signaler.
- Ne rien commiter ni pousser sans mon accord.

## Travail
- Versions centralisées dans src/Directory.Packages.props.
- Après chaque changement : dotnet build PdfSharp.sln puis dotnet test.
- Garder les modifications petites et isolées.
- Le package publié doit avoir un identifiant différent de PDFsharp.