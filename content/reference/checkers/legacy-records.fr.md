---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Enregistrements obsolètes
description: "Analyse une zone à la recherche de types d'enregistrements DNS dépréciés par l'IETF et signale chacun avec sa référence RFC et une suggestion de migration."
weight: 330
---

Le vérificateur **Types d'enregistrements DNS obsolètes** analyse une zone à la recherche de types d'enregistrements dépréciés par l'IETF, et signale chaque occurrence avec la référence RFC pertinente et une suggestion de migration concrète. Les types dépréciés encombrent une zone et, dans quelques cas, sont silencieusement ignorés par les résolveurs modernes : les nettoyer garde la zone à la fois propre et sans ambiguïté.

Il s'agit d'un vérificateur de **niveau zone** : il parcourt chaque service de la zone de travail en une seule passe et consolide les constats par type d'enregistrement, afin que le rapport présente une vue d'ensemble plutôt qu'un résultat par enregistrement.

## Ce qui est vérifié

Une seule règle, `legacy_records`, inspecte le corps de chaque enregistrement orphelin à la recherche d'un type déprécié et regroupe les constats par sévérité. Le statut par défaut de la règle est critique (la pire sévérité présente remonte en tête du rapport).

| Sévérité | Types d'enregistrement | Pourquoi |
|----------|------------------------|----------|
| Critique | `KEY`, `SIG`, `NXT` | RFC 3755 : remplacés par DNSKEY / RRSIG / NSEC ; les validateurs modernes les ignorent. |
| Avertissement | `SPF`, `A6`, `MD`, `MF` | RFC 7208 / RFC 6563 / RFC 973 : remplacés par TXT, AAAA, MX. |
| Info | `WKS`, `MB`, `MG`, `MR`, `MINFO`, `NULL`, `GPOS`, `NSAP`, `NSAP-PTR`, `X25`, `ISDN`, `RT`, `ATMA`, `EID`, `NIMLOC`, `SINK`, `NINFO`, `RKEY` | Expérimentaux ou historiques (RFC 1035, 1183, 1706, 1712, etc.) ; suppression sans danger. |

Pour chaque type détecté, le rapport nomme chaque propriétaire où il apparaît, la raison RFC, le remplacement suggéré et une instruction concrète de correction. Une zone propre produit un unique état OK avec le décompte de l'analyse ; les erreurs d'analyse rencontrées pendant le balayage sont exposées dans une section « ignorés » distincte, afin qu'un saut silencieux ne se fasse jamais passer pour un passage propre.

{{% notice style="info" title="Le type d'enregistrement SPF, pas la politique SPF" %}}
Le *type d'enregistrement* `SPF` (RFC 4408) a été déprécié par la RFC 7208 au profit de la publication de la politique SPF dans un enregistrement `TXT`. Ce vérificateur signale le type d'enregistrement `SPF` obsolète, et non votre politique SPF elle-même, qui reste valide et nécessaire lorsqu'elle est publiée dans un enregistrement `TXT`.
{{% /notice %}}

## Options

Ce vérificateur n'a aucune option réglable par l'utilisateur. Le nom de domaine et le contenu de la zone sont renseignés automatiquement.

## Dans happyDomain

Activez ce vérificateur sur le domaine depuis la vue {{< relref "/pages/checks" >}} ; il s'exécute sur l'ensemble de la zone en une seule passe et ne nécessite aucune configuration. Utilisez ses constats comme une liste de nettoyage : chaque carte vous indique le type d'enregistrement à supprimer et ce qu'il convient, le cas échéant, de publier à la place.
