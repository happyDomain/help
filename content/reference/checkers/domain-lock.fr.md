---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Verrou de transfert du domaine
description: "Vérifie qu'un domaine porte les statuts de verrouillage EPP attendus, qui le protègent contre les transferts ou modifications non autorisés."
weight: 20
---

Le vérificateur **Verrou de transfert du domaine** vérifie qu'un domaine porte les codes de statut EPP qui le protègent contre les transferts, modifications ou suppressions non autorisés. Un domaine verrouillé (par exemple portant `clientTransferProhibited`) ne peut pas être transféré sans que le verrou soit d'abord retiré, ce qui constitue une défense essentielle contre le détournement de domaine.

Il s'agit d'un vérificateur de **niveau domaine** : les codes de statut sont lus auprès du registre via une requête WHOIS/RDAP, et non dans les enregistrements DNS de la zone.

## Ce qui est vérifié

Une seule règle, `domain_lock_check`, compare les codes de statut EPP rapportés par le registre à la liste des statuts que vous exigez.

| Statut | Condition |
|--------|-----------|
| **OK** | Tous les statuts requis sont présents sur le domaine |
| **Critique** | Un ou plusieurs statuts requis sont absents (les codes manquants sont listés) |
| **Inconnu** | Aucun statut requis n'est configuré (rien à vérifier) |
| **Erreur** | La requête WHOIS/RDAP a échoué |

La comparaison tolère les différences de format : les espaces, tirets et tirets bas sont ignorés et la casse n'a pas d'importance. Ainsi `clientTransferProhibited`, `client-transfer-prohibited` et `client transfer prohibited` sont tous considérés comme équivalents.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Statuts de verrouillage requis | Liste de codes de statut EPP séparés par des virgules qui doivent être présents sur le domaine (par exemple `clientTransferProhibited`, `clientUpdateProhibited`, `clientDeleteProhibited`). Au moins un code doit être fourni. | `clientTransferProhibited` |

## Dans happyDomain

Activez ce vérificateur depuis la vue **Vérifications** du domaine ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. Le nom de domaine est renseigné automatiquement.

Ce vérificateur se combine naturellement avec {{< relref "/reference/checkers/domain-expiry" >}} et {{< relref "/reference/checkers/domain-contact" >}} pour garder la maîtrise de l'enregistrement d'un domaine.
