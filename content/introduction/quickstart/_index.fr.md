---
date: 2025-06-15T11:44:00+02:00
title: Démarrer rapidement
author: nemunaire
weight: 1
---

happyDomain est un service qui centralise la gestion de vos noms de domaines depuis différents bureaux d'enregistrement, hébergeurs ou serveurs DNS faisant autorité.
Il s'agit d'une interface web et d'une API REST qui vous proposent d'avoir une expérience des domaines plus simple que la plupart des interfaces que l'on peut voir habituellement pour gérer ses domaines, avec notamment des fonctionnalités que l'on attendrait en 2025.

Avec happyDomain, nous voulons faire en sorte que les noms de domaine et le DNS ne soient plus une expérience rebutante, mais au contraire vous redonner toutes les cartes en main pour comprendre et faire des modifications serainement.

Vous avez hâtes de commencer ? suivez le guide !


## 1. En ligne ou chez soi

happyDomain est un logiciel libre.
Cela signifie entre-autre que vous pouvez l'installer chez vous.

Si vous connaissez Docker, vous pouvez suivre le [guide d'installation dédié]({{% relref "../deploy/docker" %}}).

Si vous n'êtes pas un habitué de la ligne de commande ou que vous souhaitez évaluer rapidement le logiciel, nous vous recommandons de vous créer un compte sur notre service en ligne.

Rendez-vous sur : <https://app.happydomain.org/join>


## 2. Ajouter un domaine à gérer

happyDomain va se connecter à votre hébergeur (ou à votre serveur local faisant autorité).
Votre domaine reste hébergé là où il est aujourd'hui, utiliser happyDomain n'implique aucun transfert ou changement de propriété.

{{% notice style="info" title="Je n'ai pas encore de nom de domaine" icon="question" %}}

Nous ne vendons pas de noms de domaine, il faut que vous en disposiez déjà d'un [chez un hébergeur supporté](https://app.happydomain.org/providers/features).

{{% /notice %}}

Lorsque vous vous connectez pour la première fois à happyDomain, un assistant vous guide pour relier votre premier domaine.

Selon votre hébergeur, la procédure sera différente.
Mais pour la plupart, vous devrez vous rendre sur le compte client de votre hébergeur, et demander une clef d'API.

Pour OVH, la procédure est simplifiée car il n'y a qu'à suivre les instructions et à vous autoriser happyDomain à accéder à la partie liée aux domaines de votre compte.

Si vous avez votre propre serveur faisant autorité, vous devrez récupérer soit auprès de l'administrateur, soit en regardant dans la configuration, les clefs permettant d'interagir avec le serveur.

Une fois la connexion entre happyDomain et votre premier hébergeur ou serveur établi, vous n'avez plus qu'à sélectionner les domaines que vous souhaitez voir gérer dans happyDomain.


## 3. Consulter la zone DNS

On parle de zone DNS pour faire référence au contenu technique de votre domaine.

Pour afficher la zone correspondant à un domaine, cliquez sur le nom de domaine qui apparaît sur la page d'accueil d'happyDomain.

Après quelques secondes d'import et d'analyse entièrement automatique, vous verrez tout de suite une liste d'enregistrements ou de services tels qu'actuellement diffusés auprès de vos visiteurs.

{{% notice style="primary" title="À propos des « services »" %}}

La complexité du DNS vient en partie d'un décalage entre les contraintes purement techniques et l'usage concret des enregistrements.

happyDomain essai de simplifier cela en regroupant les enregistrements techniques sous leurs usages concrets. C'est cela que l'on appel « service ».

{{% /notice %}}


## 4. Modifier un enregistrement

Dans l'écran montrant la zone, appuyer sur un enregistrement pour afficher les détails.

Chaque enregistrement DNS a des caractéristiques et des contraintes particulières.
Les aides contextuelles essaient autant que possible de vous donner les informations essentielles pour vous guider dans les modifications dont vous avez besoin.

Lorsque vous faites une modification sur un enregistrement, celle-ci n'est pas directement publiée auprès de votre hébergeur ou de votre serveur.


## 5. Propager vos modifications

Une fois que vous avez fait toutes les modifications dont vous avez besoin, cliquez sur le bouton « Diffuser mes changements ».

Une boîte de dialogue va s'afficher pour vous montrer les changements exacts qui seront appliqués auprès de votre hébergeur.
À ce stade, il est toujours temps de sélectionner les changements que vous ne souhaitez pas/plus voir appliqués.

Avant de valider la fenêtre, vous pouvez ajouter un message qui sera enregistré dans le journal, ce qui vous permettra de vous rappeler rapidement quel était la raison de ce changement.

Car l'un des avantages d'happyDomain, c'est que vous pouvez très facilement revenir en arrière, annuler une modification, s'il s'avère que vous voyez une erreur.
[L'historique fait l'objet d'une page d'aide dédiée.]({{% relref "../../pages/domain-history" %}})

---

Eh voilà, vous savez maintenant utilisé les principales fonctionnalités d'happyDomain.
Si vous rencontrez des problèmes ou si vous avez des idées d'amélioration, [pensez à nous le faire savoir](https://github.com/happyDomain/happyDomain/issues/new).
