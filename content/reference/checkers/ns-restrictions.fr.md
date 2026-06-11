---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Restrictions des serveurs de noms
description: "Sonde chaque serveur faisant autorité d'une zone pour confirmer qu'il est correctement verrouillé : transferts de zone refusés, pas de récursion ouverte, gestion d'ANY conforme à la RFC 8482."
weight: 90
---

Le vérificateur **Restrictions des serveurs de noms** vérifie que les serveurs faisant autorité d'une zone sont correctement verrouillés. Pour chaque serveur de noms déclaré, il résout le nom d'hôte puis lance une série de sondes DNS contre chaque adresse IPv4 et IPv6 renvoyée (les cibles IPv6 sont ignorées sans erreur lorsque l'hôte n'a pas de connectivité IPv6). L'objectif est de détecter les erreurs de configuration courantes qui divulguent des données ou transforment un serveur de noms en vecteur d'abus : transferts de zone ouverts, récursion ouverte et réponses `ANY` non bornées.

Ce vérificateur s'applique au niveau du **service** : il cible un service d'origine ou d'origine NS-seule (`abstract.Origin`, `abstract.NSOnlyOrigin`) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

Chaque règle émet un constat par adresse de serveur de noms sondée, avec un `code` stable.

| Règle | Vérifie | Sévérité en cas d'échec |
|---|---|---|
| `ns_resolution` | Chaque nom d'hôte NS déclaré dans la délégation se résout en au moins une adresse IP. | Critique |
| `ns_axfr_refused` | Les transferts de zone `AXFR` sont refusés par chaque serveur faisant autorité. | Critique |
| `ns_ixfr_refused` | Les transferts de zone `IXFR` sont refusés par chaque serveur faisant autorité. | Avertissement |
| `ns_no_recursion` | Les serveurs faisant autorité n'annoncent pas la récursion (bit RA non positionné). | Avertissement |
| `ns_any_handled` | Les requêtes `ANY` sont traitées conformément à la RFC 8482 (HINFO ou réponse minimale plutôt que le contenu complet de la zone). | Avertissement |
| `ns_is_authoritative` | Les serveurs de noms répondent de façon autoritaire (bit AA positionné) pour la zone. | Info |

{{% notice style="info" title="Pourquoi c'est important" %}}
Un `AXFR` ouvert permet à quiconque de télécharger la zone entière, exposant votre nommage interne. La récursion ouverte transforme votre serveur faisant autorité en relais d'amplification et en cible d'empoisonnement de cache. Les réponses `ANY` non bornées constituent un vecteur d'amplification classique que la RFC 8482 a été écrite pour neutraliser.
{{% /notice %}}

## Options

Ce vérificateur n'expose aucune option réglable : il exécute un ensemble fixe de sondes contre chaque adresse de serveur de noms résolue.

## Dans happyDomain

Activez le vérificateur Restrictions des serveurs de noms depuis l'onglet **Vérifications** d'un service d'origine. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet. Pour la santé et l'accord plus larges de ces mêmes serveurs faisant autorité, voyez {{< relref "/reference/checkers/authoritative-consistency" >}}.
