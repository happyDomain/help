---
date: 2026-06-10T12:00:00+02:00
title: "Importer et exporter une zone"
author: nemunaire
weight: 1600
description: "Importer un fichier de zone existant dans happyDomain, ou exporter votre zone au format BIND standard"
---

happyDomain peut échanger votre zone avec le reste du monde DNS en utilisant le format de fichier de zone standard (le célèbre format BIND). Vous pouvez **importer** un fichier de zone pour alimenter votre copie de travail, et **exporter** la zone actuelle pour la lire, la copier ou la conserver ailleurs.

Ces actions sont accessibles depuis le menu en forme d'engrenage situé en haut de la barre latérale de l'éditeur de zone.

## Réimporter la zone en ligne

Avant de travailler avec des fichiers, il est utile de savoir que vous pouvez toujours récupérer la zone **actuelle** auprès de votre hébergeur. Dans le menu engrenage, choisissez « Récupérer la zone actuelle ».

Cette action contacte votre hébergeur, lit la zone telle qu'elle est en ce moment, et rafraîchit votre copie de travail à partir d'elle. Utilisez-la lorsque la zone a pu être modifiée en dehors de happyDomain, ou pour repartir d'un état propre.

{{% notice style="warning" title="Cela remplace les modifications non publiées" icon="triangle-exclamation" %}}
Réimporter la zone en ligne récupère la version de l'hébergeur. Tout changement local que vous n'aviez pas encore publié peut être écrasé : examinez donc d'abord [vos changements en attente]({{% relref "publish-changes" %}}) si vous souhaitez les conserver.
{{% /notice %}}

## Importer un fichier de zone

Pour charger une zone à partir d'un fichier de zone standard :

1. Ouvrez le menu engrenage de la barre latérale et choisissez l'action d'import de zone.
2. Une fenêtre s'ouvre avec deux onglets :
   - **Depuis du texte** : collez le contenu d'un fichier de zone directement dans la zone de saisie.
   - **Depuis un fichier de zone** : sélectionnez un fichier sur votre ordinateur.
3. Cliquez sur le bouton d'envoi pour le transmettre.

![La fenêtre d'import de zone, avec les onglets « Depuis du texte » et « Depuis un fichier de zone »](happydomain-modal-import-zone.webp)

happyDomain analyse le fichier de zone, reconnaît les enregistrements et les regroupe à nouveau en services dans votre copie de travail. Comme pour toute autre modification, le contenu importé reste local tant que vous ne l'avez pas publié.

{{% notice style="info" title="Format standard" icon="circle-info" %}}
Le format attendu est le fichier de zone textuel standard, le même que celui qu'utilisent BIND et la plupart des outils DNS. Une ligne telle que `@ 4269 IN SOA root ns 2042070136 ...` est un exemple typique de ce que vous pouvez coller.
{{% /notice %}}

## Exporter / visualiser la zone

Pour obtenir votre zone sous forme de fichier de zone standard :

1. Ouvrez le menu engrenage et choisissez « Voir ma zone ».
2. happyDomain affiche la zone actuelle au format BIND standard, avec coloration syntaxique.
3. Utilisez le bouton **Copier dans le presse-papier** pour récupérer le fichier entier en un clic.

![La zone affichée au format BIND standard, avec coloration syntaxique et un bouton de copie dans le presse-papier](happydomain-export-zone.webp)

Cette vue reflète toujours la zone que vous consultez actuellement. Si vous parcourez une version passée depuis l'[historique]({{% relref "domain-history" %}}), l'export montre cette version historique, ce qui est pratique pour comparer ou restaurer un état antérieur.

## Cas d'usage typiques

- **Migrer depuis un autre outil** : exportez la zone de votre outil précédent sous forme de fichier de zone, puis importez-la ici avec « Depuis un fichier de zone ».
- **Conserver une sauvegarde** : ouvrez « Voir ma zone » et copiez le contenu dans un endroit sûr.
- **Édition en masse** : exportez, modifiez le fichier dans votre éditeur favori, puis réimportez le résultat.

## Après un import

Un import ne modifie que votre copie de travail. Pour le rendre effectif chez votre hébergeur :

1. Examinez le différentiel obtenu, puis [publiez les changements]({{% relref "publish-changes" %}}).
