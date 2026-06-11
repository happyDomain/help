---
date: 2020-12-09T18:12:45+01:00
title: "Utiliser l'éditeur de zone"
weight: 900
description: "Visualiser, ajouter, modifier et supprimer les enregistrements et services de votre zone, regroupés par sous-domaine"
---

L'éditeur de zone est l'écran principal pour travailler sur un domaine. Il présente le contenu de votre zone regroupé par sous-domaine, et vous permet d'ajouter, modifier et supprimer les services et enregistrements qui composent votre zone, sans rien changer chez votre hébergeur tant que vous n'avez pas décidé de [publier vos changements]({{% relref "publish-changes" %}}).

## Disposition de l'éditeur

Lorsque vous ouvrez un domaine, l'écran est divisé en deux parties :

- À gauche, une **barre latérale** liste tous les sous-domaines de la zone. Elle donne aussi accès aux actions concernant l'ensemble du domaine (historique, journal d'audit, contrôles, WHOIS, import/export, etc.).
- À droite, le **Visualiseur de zone** affiche le contenu de la zone, un bloc par sous-domaine.

![L'éditeur de zone d'happyDomain, avec la barre latérale des sous-domaines et le visualiseur de zone](happydomain-abstract-zone-records.webp)

Tout en haut de la barre latérale se trouvent le bouton **Ajouter un sous-domaine** et un menu en forme d'engrenage regroupant les autres actions. Le bouton servant à transmettre vos changements à votre hébergeur (« Diffuser mes changements ») est également accessible depuis cet écran ; voyez [cette page]({{% relref "publish-changes" %}}) pour les détails.

{{% notice style="info" title="Rien n'est envoyé automatiquement" icon="cloud" %}}
Chaque modification que vous faites dans l'éditeur est conservée localement dans happyDomain. Elle n'est transmise à votre hébergeur que lorsque vous la publiez explicitement.
{{% /notice %}}

## Parcourir la zone

La zone est organisée **par sous-domaine**. La racine du domaine apparaît en premier (affichée avec le nom de domaine seul), suivie de chaque sous-domaine. Les sous-domaines intermédiaires qui ne portent aucun service sont tout de même affichés, signalés par une icône en pointillés, afin que vous puissiez toujours voir l'arborescence complète.

- Cliquez sur le titre d'un sous-domaine pour le **déplier ou le replier** et révéler les services qu'il contient.
- Lorsqu'un bloc est replié, un badge indique combien de services il contient ; survolez-le pour obtenir un aperçu rapide.
- Utilisez la **barre latérale** pour accéder directement à un sous-domaine : elle reflète la liste et fait défiler le visualiseur jusqu'au bloc correspondant.

Les alias pointant vers un sous-domaine apparaissent à côté de son titre, sous forme d'un badge « + N alias ».

<!-- TODO: screenshot d'un bloc de sous-domaine déplié, montrant ses services -->

## Enregistrements et services

Par défaut, happyDomain ne vous présente pas une liste brute d'enregistrements DNS. Il regroupe au contraire les enregistrements liés en **services**, des objets de plus haut niveau plus simples à appréhender (un serveur de courrier, un site web, une délégation, etc.). Chaque service se déploie en montrant les enregistrements concrets qu'il génère.

Si vous préférez travailler directement avec les enregistrements individuels, vous pouvez changer le mode d'affichage de la zone dans [vos préférences]({{% relref "settings" %}}). L'éditeur propose alors un bouton **Ajouter un enregistrement** au lieu d'**Ajouter un service**.

## Ajouter un sous-domaine

1. Cliquez sur **Ajouter un sous-domaine** en haut de la barre latérale.
2. Saisissez le nom du sous-domaine à créer (relatif à votre domaine).
3. happyDomain vous propose ensuite d'ajouter immédiatement un premier service sur ce sous-domaine.

Un sous-domaine n'existe réellement qu'à partir du moment où il porte au moins un service : les deux étapes sont donc enchaînées.

![La fenêtre d'ajout d'un sous-domaine](happydomain-modal-new-subdomain.webp)

## Ajouter un service

Pour ajouter un service à un sous-domaine existant :

1. Repérez le bloc du sous-domaine (ou la racine du domaine) dans le visualiseur.
2. Cliquez sur le bouton **+** présent sur le titre du sous-domaine, ou utilisez l'action **Ajouter un service**.
3. Choisissez le type de service dans le sélecteur. La liste s'adapte à ce qui existe déjà sur ce sous-domaine (par exemple, vous ne pouvez pas ajouter deux services en conflit).
4. Remplissez le formulaire du service choisi, puis enregistrez.

![Le sélecteur de type de service](happydomain-modal-service-selector.webp)

## Inspecter un service

Cliquez sur un service pour ouvrir le **panneau de détails** qui apparaît par la droite. Il présente :

- Une description du type de service et le commentaire éventuel que vous avez défini.
- Les enregistrements DNS concrets que le service produit.
- L'état de propagation (date de la dernière publication de la modification).
- Les contrôles de santé éventuellement rattachés à ce service (voyez {{% relref "checks" %}}).

Depuis ce panneau, vous pouvez aussi ajuster le **TTL par défaut** du service, le modifier ou le supprimer.

![Le panneau de détails d'un service](happydomain-offcanva-service-details.webp)

## Modifier un service

1. Ouvrez le panneau de détails du service, puis cliquez sur **Modifier ce service** ; ou utilisez le bouton crayon affiché sur les services simples comme les alias.
2. happyDomain ouvre le formulaire d'édition complet du service.
3. Effectuez vos changements et enregistrez. Le visualiseur se met à jour pour les refléter.

## Supprimer un service

1. Ouvrez le panneau de détails du service.
2. Cliquez sur **Supprimer ce service**.

Le service et tous les enregistrements qu'il générait sont retirés de votre copie de travail de la zone.

{{% notice style="warning" title="Certains services ne peuvent pas être supprimés" icon="triangle-exclamation" %}}
Le service d'origine d'une zone (celui qui porte le SOA et les serveurs de noms faisant autorité) est essentiel et ne peut pas être supprimé depuis l'éditeur.
{{% /notice %}}

Pour les alias (CNAME) et les pointeurs inverses (PTR), un bouton de suppression dédié est disponible directement sur le titre du sous-domaine.

## Alias

Lorsqu'un sous-domaine porte des services, vous pouvez lui rattacher un **alias** à l'aide du bouton en forme de lien présent sur son titre. L'alias fait pointer un autre nom vers ce sous-domaine.

## Et ensuite

Aucune des opérations ci-dessus ne change quoi que ce soit chez votre hébergeur pour l'instant. Lorsque vos modifications vous conviennent :

- Examinez ce qui va changer, puis envoyez les changements ; voyez {{% relref "publish-changes" %}}.

Vous pouvez aussi réimporter la zone en ligne, ou [l'importer / l'exporter sous forme de fichier de zone standard]({{% relref "import-export" %}}).
