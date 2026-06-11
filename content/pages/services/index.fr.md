---
date: 2026-06-10T12:00:00+02:00
title: Services
weight: 1200
description: "Comprendre comment happyDomain transforme les enregistrements DNS bruts en services de haut niveau, et comment en ajouter un à un sous-domaine"
---

happyDomain ne vous demande pas de raisonner en termes d'enregistrements DNS individuels. À la place, il regroupe les enregistrements qui vont ensemble au sein d'un unique **service** porteur de sens : une messagerie, un site web, une délégation, une politique CAA, etc. C'est le fondement de la [vue abstraite]({{% relref "domain-abstract" %}}) de votre zone.

## Qu'est-ce qu'un service ?

Un service est un objet de plus haut niveau qui masque la complexité d'un ou de plusieurs enregistrements DNS derrière un formulaire clair, orienté usage.

Par exemple, au lieu d'éditer séparément un enregistrement `MX`, quelques enregistrements `A`/`AAAA` et un enregistrement `SPF` de type `TXT`, vous remplissez un seul service **messagerie**. happyDomain se charge de générer les bons enregistrements, avec les bons noms et la bonne syntaxe.

Chaque service appartient à une famille :

- **Services** (abstraits) : les objets recommandés, pensés pour les humains (messagerie, site web, CAA, délégation, etc.). Ils se traduisent automatiquement en un ou plusieurs enregistrements.
- **Fournisseurs** : des services liés à un acteur tiers précis qui publie son propre assistant (par exemple un service hébergé nécessitant un ensemble d'enregistrements prédéfini).
- **Ressources DNS brutes** : une solution de repli qui vous permet d'ajouter directement un enregistrement unique (`A`, `TXT`, `SRV`, etc.), lorsqu'aucun service abstrait ne correspond à votre besoin.

{{% notice style="info" title="Pourquoi des services plutôt que des enregistrements ?" icon="lightbulb" %}}
Travailler avec des services vous évite d'avoir à mémoriser les types d'enregistrements exacts, leur ordre ou leur syntaxe. happyDomain valide votre saisie et génère du DNS correct à votre place, ce qui réduit fortement le risque d'erreur de configuration.
{{% /notice %}}

## La vue par services d'un sous-domaine

Lorsque vous ouvrez un domaine, chaque sous-domaine est présenté sous la forme d'une liste des services qui lui sont rattachés. Un sous-domaine peut accueillir plusieurs services à la fois : par exemple, la racine (`@`) d'un domaine porte souvent à la fois un service de messagerie, un service de site web et une politique CAA.

![Un sous-domaine affichant plusieurs services dans l'éditeur de zone](happydomain-abstract-zone-records.webp)

Depuis cette vue, vous pouvez :

- **Ajouter** un nouveau service au sous-domaine ;
- **Modifier** un service existant pour en changer les valeurs ;
- **Supprimer** un service dont vous n'avez plus besoin.

Toutes ces modifications sont préparées localement et ne sont appliquées chez votre hébergeur qu'au moment de leur publication. Consultez la [vue abstraite]({{% relref "domain-abstract" %}}) pour comprendre le fonctionnement de l'édition et de la propagation.

## Ajouter un service à un sous-domaine

Pour rattacher un nouveau service, placez-vous sur le sous-domaine où vous le souhaitez (voir [Sous-domaines]({{% relref "subdomains" %}}) pour naviguer dans la zone), puis suivez ces étapes.

### 1. Ouvrir le sélecteur de services

Cliquez sur l'action **Ajouter un service** du sous-domaine. Un sélecteur s'ouvre, listant tous les types de services que vous pouvez ajouter.

![La fenêtre de sélection des services](happydomain-modal-service-selector.webp)

Le sélecteur est organisé en onglets pour vous aider à restreindre la liste :

- **Tous** : tous les types de services disponibles.
- **Services** : les services abstraits de haut niveau (recommandé).
- **Fournisseurs** : les services spécifiques à un fournisseur.
- **Ressources DNS brutes** : un type d'enregistrement unique, lorsque vous avez besoin d'un contrôle total.

Vous pouvez également saisir un terme dans le champ de recherche, en haut, pour filtrer la liste par nom. Appuyer sur Entrée sélectionne le premier résultat disponible.

{{% notice style="tip" title="Entrées grisées" icon="circle-info" %}}
Certains types de services peuvent apparaître désactivés. Cela se produit lorsque le service ne peut pas être ajouté dans le contexte actuel : par exemple parce que votre hébergeur DNS ne prend pas en charge le type d'enregistrement sous-jacent, ou parce que ce service existe déjà sur ce sous-domaine et qu'une seule instance est autorisée. Survolez une entrée désactivée pour en connaître la raison.
{{% /notice %}}

### 2. Choisir le type de service

Sélectionnez le service correspondant à ce que vous souhaitez publier. happyDomain connaît les types d'enregistrements pris en charge par votre hébergeur : seuls les choix pertinents vous sont donc proposés. Pour savoir ce qu'un hébergeur donné sait gérer, consultez la page [Fonctionnalités des fournisseurs]({{% relref "provider-features" %}}).

### 3. Remplir le formulaire du service

happyDomain présente alors un formulaire adapté au service choisi. Chaque champ correspond à une information porteuse de sens (un hôte cible, une priorité, une clé publique, une valeur de politique, etc.) plutôt qu'à de la syntaxe d'enregistrement brute.

<!-- TODO: capture d'écran d'un formulaire d'édition de service -->

Renseignez les champs, puis validez pour ajouter le service au sous-domaine.

### 4. Vérifier les enregistrements générés

Une fois le formulaire validé, happyDomain convertit votre saisie en enregistrements DNS correspondants et les ajoute à la zone en préparation. Vous pouvez les vérifier dans la [vue abstraite]({{% relref "domain-abstract" %}}) avant de publier vos modifications chez votre hébergeur.

## Essayer les services sans compte

happyDomain propose également un **générateur d'enregistrements** public qui vous permet de construire et de prévisualiser les enregistrements DNS produits par un service sans être connecté. Choisissez un type de service, saisissez un nom de domaine, remplissez le formulaire : les entrées de fichier de zone générées s'affichent instantanément.

C'est un moyen pratique de découvrir comment un service donné se traduit en véritables enregistrements DNS, ou de récupérer un enregistrement à coller ailleurs.

<!-- TODO: capture d'écran du générateur d'enregistrements public -->
