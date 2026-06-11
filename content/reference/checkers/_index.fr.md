---
date: 2026-06-11T09:00:00+02:00
title: Vérificateurs
author: nemunaire
archetype: chapter
weight: 35
aliases:
    checkers
---

Les vérificateurs sont les briques de base du système de surveillance de happyDomain. Chacun collecte des données sur un domaine, une zone ou un service, les évalue selon un ensemble de règles, puis rapporte un état clair (OK, Avertissement, Critique ou Erreur).

Ce chapitre documente chaque vérificateur fourni avec happyDomain : ce qu'il contrôle, la portée à laquelle il s'applique, les options que vous pouvez régler et les règles qu'il évalue. Pour le fonctionnement au quotidien (configuration, planification et lecture des résultats dans l'interface), consultez {{< relref "/pages/checks" >}}.

## Portées

Chaque vérificateur déclare la portée sur laquelle il opère :

- **Niveau domaine** : concerne le domaine lui-même, indépendamment de ses enregistrements DNS (statut d'enregistrement, expiration, verrou de transfert...).
- **Niveau zone** : nécessite le contenu complet de la zone (validation DNSSEC, cohérence de la délégation...).
- **Niveau service** : cible un service précis publié sur un sous-domaine (HTTP, TLS, ping...), et se configure depuis l'onglet **Vérifications** de ce service.

## États

Les vérificateurs rapportent l'un des états suivants, par ordre de gravité :

| État | Signification |
|------|---------------|
| **OK** | Tout se situe dans les paramètres acceptables |
| **Info** | Information utile, aucune action requise |
| **Avertissement** | Un seuil est sur le point d'être atteint ; attention recommandée |
| **Critique** | Un seuil a été dépassé ; action requise |
| **Erreur** | La vérification elle-même a échoué (erreur de collecte, mauvaise configuration) |
| **Inconnu** | La vérification n'a pas pu déterminer de résultat |

{{% children %}}
