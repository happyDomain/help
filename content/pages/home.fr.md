---
date: 2020-12-09T18:12:45+01:00
title: La page d'accueil
aliases:
    domains
weight: 10
---

## Vos domaines

La page d'accueil présente la liste de l'ensemble des domaines gérés par happyDomain, quelque soit leur hébergeur :

![Les domaines gérés par happyDomain](domain-list.png)

Cliquez sur l'un des domaines pour commencer à [y apporter des modifications]({{% relref "domain-abstract" %}}) (ajouter un sous-domaine, ajouter un service, ...).


## Vos registres et hébergeurs de domaines

Sur la droite, vous voyez la liste des différents hébergeurs de vos domaines :

![Les hébergeurs de vos domaines](hosters-list.png)

Vous pouvez [ajouter un nouvel hébergeur]({{% relref "source-new-choice" %}}) en cliquant sur le bouton +, présent dans l'en-tête du tableau.

En cliquant sur une ligne de ce tableau, vous filtrerez la liste des domaines pour n'afficher que les domaines gérés par cet hébergeur.

Vous verrez aussi, si l'hébergeur permet de lister les domaines qui vous appartiennent, les domaines que vous pouvez ajouter à happyDomain :

![Filtrage des domaines en fonction de l'hébergeur](hoster-ovh.png)

Pour afficher à nouveau la liste dans son intégralité, recliquez simplement sur l'hébergeur qui est sélectionné.


### Modifier ou supprimer un hébergeur

Si vous constatez une erreur ou n'avez plus besoin d'un hébergeur, cliquez sur les ... sur la ligne de l'hébergeur concerné. Vous aurez alors la possibilité de choisir entre [mettre à jour les informations]({{% relref "source-update" %}}) ou supprimer l'hébergeur :

![Modification ou suppression d'un hébergeur](hoster-edit.png)

Notez que vous ne pourrez pas supprimer l'hébergeur tant que des domaines y faisant référence, existeront dans la liste de gauche.


## Ajouter un domaine

Vous avez un nouveau domaine que vous souhaitez gérer dans happyDomain ? Commencez par entrer son nom dans le champ présent sous la liste. Vous serez ensuite guidé vers l'écran [permettant de choisir l'hébergeur]({{% relref "domain-new" %}}).

![Emplacement pour ajouter un domaine qui n'est pas listé](new-domain.png)

Le champ n'apparaît pas lorsqu'un hébergeur est sélectionné à droite. Sauf si cet hébergeur ne permet pas de lister les domaines :

![Cas particulier d'ajout pour les serveurs DNS autoritaire](hoster-self.png)

Dans ce cas, la validation du champ recherchera automatiquement le nouveau domaine auprès de l'hébergeur sélectionné, comme l'indique le message juste au dessus du champ.
