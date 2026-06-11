---
date: 2025-06-15T11:06:00+02:00
title: "Voir l'historique des changements"
weight: 1500
---

À chaque fois que vous [publiez des modifications]({{% relref "publish-changes" %}}) avec happyDomain, celles-ci sont enregistrées dans un journal. Ce journal vous permet de retrouver facilement l'état de vos domaines tels qu'ils étaient déployés précédemment, et de voir quand vous avez fait chaque modification.

## Ouvrir l'historique

Depuis la page de votre domaine, ouvrez l'entrée **Historique** dans le menu. happyDomain affiche toutes les versions enregistrées de la zone, de la plus récente en haut à la plus ancienne en bas.

<!-- TODO: capture d'écran de la page d'historique -->

Pour garder la liste lisible, les versions sont regroupées par mois. Un en-tête marque le début de chaque mois, ce qui vous permet de retrouver rapidement une modification d'après sa période.

## Lire une version

Chaque version est identifiée par le moment de sa dernière modification, accompagné de l'avatar et de l'adresse électronique de l'auteur qui l'a effectuée.

Trois dates peuvent être affichées pour une version :

- **Publiée le** : le moment où cette version a été déployée chez votre fournisseur DNS. Une version sans cette date a été enregistrée, mais jamais publiée.
- **Enregistrée le** : le moment où la version a été figée (sauvegardée comme un état définitif de la zone).
- **Dernière modification le** : le moment où la version a été modifiée pour la dernière fois.

Si un message a été associé lors de la publication des changements, il apparaît sous les dates, à la manière d'un message de commit dans un gestionnaire de versions.

## Voir la zone à un instant donné

Pour examiner le contenu complet de la zone tel qu'il était pour une version donnée, cliquez sur le bouton en forme d'œil situé à côté de sa date. happyDomain ouvre cette version en lecture seule, ce qui vous permet de parcourir tous les enregistrements exactement tels qu'ils étaient à ce moment.

## Comparer deux versions

Sous chaque version (sauf la plus ancienne), la section **Voir les changements** vous permet de la comparer avec la version qui la précède immédiatement.

<!-- TODO: capture d'écran de la section des changements dépliée -->

Dépliez la section pour révéler les modifications : les enregistrements ajoutés, supprimés et modifiés sont mis en évidence, ce qui vous permet de voir d'un coup d'œil ce que chaque publication a changé. La comparaison la plus récente est dépliée automatiquement à l'ouverture de la page.

{{% notice style="info" title="Pas encore d'historique ?" %}}
Un domaine ne se constitue un historique qu'à partir du moment où vous commencez à y publier des changements. Si vous venez d'importer un domaine, son historique se remplira au fur et à mesure que vous effectuerez et publierez vos premières modifications.
{{% /notice %}}
