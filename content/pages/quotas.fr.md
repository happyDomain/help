---
date: 2026-06-10T12:00:00+02:00
title: Quotas et limites
weight: 2400
description: "Comprendre les limites qu'un administrateur peut appliquer à votre compte, et ce qu'il se passe lorsque l'une d'elles est atteinte"
---

Une instance happyDomain peut appliquer des **quotas** à chaque compte. Les quotas sont des limites définies par la personne qui administre l'instance (l'administrateur), et non par vous. Ils servent principalement à préserver l'équité sur une instance partagée et à éviter de surcharger les serveurs ; ils concernent avant tout le système de [supervision et contrôles]({{% relref "checks" %}}).

{{% notice style="info" title="Les quotas sont gérés par l'administrateur" icon="circle-info" %}}
Les quotas sont en **lecture seule** pour les utilisateurs ordinaires. Il n'existe aucun écran dans votre compte permettant de consulter ou de modifier votre propre quota : ils sont configurés côté serveur et ne sont ajustés qu'au travers de l'interface d'administration. Si une limite vous gêne, la bonne démarche consiste à contacter l'administrateur de votre instance.
{{% /notice %}}


## Ce que les quotas peuvent couvrir

Sur une instance happyDomain par défaut, les quotas portent sur le fonctionnement du planificateur de supervision pour votre compte. Un administrateur peut définir, par compte ou pour l'ensemble de l'instance :

- **Nombre maximal de contrôles par jour** : un plafond sur le nombre d'exécutions de contrôles lancées automatiquement pour vous chaque jour. Une fois ce plafond atteint, les contrôles planifiés suivants attendent le lendemain.
- **Conservation des résultats** : la durée pendant laquelle les résultats de vos contrôles sont conservés avant que les anciennes exécutions ne soient automatiquement nettoyées.
- **Pause pour inactivité** : après une période sans connexion, le planificateur peut cesser d'exécuter vos contrôles automatiques jusqu'à votre prochaine connexion. Cela évite que l'instance ne dépense des ressources pour des comptes abandonnés.
- **Pause de planification** : un administrateur peut entièrement suspendre la planification automatique d'un compte.

Chacun de ces réglages peut rester à la valeur par défaut de l'instance, être fixé à une valeur particulière pour votre compte, ou être explicitement rendu illimité, à la discrétion de l'administrateur.

{{% notice style="tip" title="Pas de limite fixe sur le nombre de domaines" icon="lightbulb" %}}
Sur une instance happyDomain standard, le système de quotas n'impose pas de plafond intégré sur le nombre de domaines que vous pouvez gérer. Si une instance particulière restreint ce point, il s'agit d'une politique propre à cette instance ; demandez à son administrateur les détails qui vous concernent.
{{% /notice %}}


## Ce que vous voyez lorsqu'une limite est atteinte

Les quotas agissent surtout en arrière-plan. Vous ne verrez généralement pas d'écran de quota ; vous en constatez plutôt les effets :

- **Les contrôles automatiques cessent de s'exécuter** : si votre plafond quotidien de contrôles est atteint, ou si votre compte est en pause pour inactivité ou de planification, les contrôles planifiés ne s'exécutent simplement plus tant que la condition persiste. Vous pouvez toujours déclencher un contrôle **manuellement** depuis l'interface des [contrôles]({{% relref "checks#lancer-une-vérification-manuellement" %}}).
- **Les anciens résultats disparaissent** : avec une limite de conservation, les exécutions plus anciennes que la durée autorisée sont supprimées lors du nettoyage régulier, si bien que l'historique ne remonte que jusqu'à un certain point.
- **Une action est refusée** : lorsqu'une opération dépasserait une limite, l'interface affiche un message d'erreur expliquant ce qui n'a pas fonctionné.

Si vous ne savez pas si un comportement est dû à un quota, ou si vous avez besoin qu'une limite soit relevée, contactez l'administrateur de votre instance.
