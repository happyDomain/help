---
date: 2026-06-10T12:00:00+02:00
title: Votre compte et vos réglages
weight: 2000
description: "Gérez votre compte happyDomain et personnalisez l'interface : langue, aide des champs, confirmation, affichage des zones, mot de passe et suppression du compte"
aliases:
    me
---

happyDomain vous permet de gérer votre compte et d'ajuster le comportement de l'interface. Vos préférences sont enregistrées avec votre compte ; vous les retrouvez ainsi sur chaque appareil depuis lequel vous vous connectez.

Vous accédez à cette page en cliquant sur le lien « Mon compte » dans le menu en haut. Chaque option est décrite ci-dessous.

<!-- TODO: screenshot of the settings form -->

## Langue

Choisissez la langue utilisée dans toute l'interface. La liste contient toutes les langues actuellement disponibles sur votre instance happyDomain. Le changement prend effet dès l'enregistrement.

## Aide des champs

Les champs des formulaires sont souvent accompagnés d'un court texte d'aide. Ce réglage détermine la façon dont cette aide s'affiche :

- **Masquer** : aucun texte d'aide n'est affiché.
- **Infobulle près du champ** : l'aide apparaît sous forme d'infobulle au survol du champ.
- **Sous le champ lors de la saisie** : l'aide apparaît sous le champ, mais uniquement pendant que vous le modifiez.
- **Sous le champ, en permanence** : l'aide reste affichée sous chaque champ.

Choisissez le niveau d'accompagnement adapté à votre aisance avec le DNS.

## Confirmation avant application

Lorsque vous publiez des modifications chez votre fournisseur DNS, happyDomain peut afficher une étape de confirmation au préalable. Ce réglage décide du moment :

- **Demander en cas de différences inattendues** : la confirmation n'apparaît que lorsque les modifications diffèrent de ce qui était attendu.
- **Toujours demander** : une confirmation est systématiquement affichée avant l'application.
- **Ne jamais demander, écraser à la publication** : les modifications sont appliquées directement, sans étape de confirmation.

Consultez [Publier ses modifications]({{% relref "publish-changes" %}}) pour le déroulé complet de la publication.

## Affichage des zones

Choisissez la façon dont la zone d'un domaine est présentée :

- **Vue en grille (le plus simple)** : la présentation la plus visuelle et la plus accessible.
- **Vue en liste (le plus rapide)** : une liste compacte, plus rapide à parcourir.
- **Liste avec enregistrements (avancé)** : une liste qui expose également les enregistrements DNS sous-jacents, pour les utilisateurs avancés.

## Afficher les types d'enregistrements DNS

Cet interrupteur affiche le type d'enregistrement (A, MX, CNAME, etc.) associé à chaque service. Il s'adresse aux personnes déjà familières du DNS qui souhaitent voir les types techniques derrière les noms de services simplifiés.

## Lettre d'information

L'option permettant de recevoir les nouvelles des prochaines améliorations vous est proposée lors de la [création de votre compte]({{% relref "signup" %}}). Pour modifier ensuite votre abonnement, référez-vous au lien de désinscription présent dans les messages que vous recevez.

## Enregistrement

Une fois vos préférences définies, cliquez sur « Enregistrer les réglages ». Un message de confirmation apparaît et les nouveaux réglages prennent effet immédiatement, y compris un changement de langue si vous en avez effectué un.

## Changer de mot de passe

Une partie dédiée de la page vous permet de changer le mot de passe de votre compte.

## Changer l'adresse électronique de compte

Il n'est pour l'instant pas possible de changer l'adresse électronique de votre compte. Nous vous invitons à nous contacter si vous souhaitez la changer.

## Supprimer son compte

La dernière partie de la page vous permet de supprimer votre compte happyDomain.

Une fois la suppression validée, votre compte ne sera plus accessible et l'ensemble des données vous appartenant sera supprimé peu de temps après, lors d'un nettoyage régulier de la base de données.

À partir du moment où vous supprimez votre compte, vos domaines continueront de répondre selon la dernière mise à jour que vous avez effectué sur happyDomain. La suppression n'affectera pas les données distribuées.
