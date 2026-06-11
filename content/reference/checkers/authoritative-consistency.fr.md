---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Cohérence des serveurs faisant autorité
description: "Sonde chaque serveur faisant autorité d'une zone et vérifie qu'ils s'accordent entre eux et avec le parent sur les NS, le SOA, la joignabilité, EDNS0 et l'autorité."
weight: 80
---

Le vérificateur **Cohérence des serveurs faisant autorité** sonde chaque serveur faisant autorité d'une zone et vérifie qu'ils s'accordent, entre eux et avec la délégation parente. Là où le vérificateur {{< relref "/reference/checkers/delegation" >}} se concentre sur le passage de relais parent/enfant, celui-ci porte sur l'*apex lui-même* : tous les serveurs servent-ils les mêmes `NS` et `SOA`, sont-ils tous joignables en UDP et en TCP, prennent-ils en charge EDNS0, répondent-ils de façon autoritaire, et à quelle vitesse répondent-ils ?

Ce vérificateur s'applique au niveau du **service** : il cible un service d'origine ou d'origine NS-seule (`abstract.Origin`, `abstract.NSOnlyOrigin`) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

Chaque règle émet un code de constat. Plusieurs sévérités dépendent des options ci-dessous.

| Code de constat | Sévérité par défaut | Condition |
|---|---|---|
| `authoritative_consistency_no_ns` | Critique | Aucun serveur de noms n'a pu être découvert (liste déclarée vide et requête parente sans réponse). |
| `authoritative_consistency_too_few_ns` | Avertissement | Moins de serveurs déclarés que `minNameServers` (la RFC 1034 recommande au moins 2). |
| `authoritative_consistency_parent_query_failed` | Avertissement | La requête de délégation parente a échoué (erreur réseau, REFUSED, etc.). |
| `authoritative_consistency_parent_drift` | Avertissement | Le RRset `NS` du parent ne correspond pas aux `NS` déclarés dans le service. |
| `authoritative_consistency_ns_unresolvable` | Critique | Un serveur de noms déclaré n'a aucun enregistrement `A` ni `AAAA`. |
| `authoritative_consistency_ns_udp_failed` | Critique | Un serveur de noms n'a répondu à aucune requête SOA en UDP/53. |
| `authoritative_consistency_ns_tcp_failed` | Critique avec `requireTCP`, sinon avertissement | Un serveur de noms n'a pas répondu en TCP/53 (exigé par la RFC 7766 et par DNSSEC). |
| `authoritative_consistency_lame` | Critique | Un serveur de noms a répondu sans le bit AA pour la zone (délégation boiteuse). |
| `authoritative_consistency_no_soa` | Critique | Un serveur fait autorité mais n'a renvoyé aucun `SOA`. |
| `authoritative_consistency_edns_unsupported` | Avertissement | Un serveur ignore ou gère mal les requêtes EDNS0 (RFC 6891). |
| `authoritative_consistency_slow_ns` | Info | Le temps de réponse d'un serveur a dépassé `latencyThresholdMs`. |
| `authoritative_consistency_serial_drift` | Avertissement | Les serveurs divergent sur le numéro de série `SOA` (zone non entièrement propagée). |
| `authoritative_consistency_serial_stale_vs_saved` | Avertissement | Le numéro de série enregistré dans happyDomain est plus récent que celui publié (changement probablement non poussé). |
| `authoritative_consistency_serial_ahead_of_saved` | Info | Les serveurs publient un numéro de série plus récent que celui enregistré (changement hors bande). |
| `authoritative_consistency_soa_fields_drift` | Avertissement | Les serveurs divergent sur les champs `SOA` (MNAME, RNAME, refresh, retry, expire, minimum). |
| `authoritative_consistency_ns_rrset_drift` | Avertissement | Les serveurs divergent sur le RRset `NS` qu'ils publient à l'apex. |
| `authoritative_consistency_ns_rrset_mismatch_config` | Avertissement | Le RRset `NS` publié ne correspond pas aux `NS` déclarés dans le service. |

## Options

| Option | Signification | Défaut |
|---|---|---|
| `requireTCP` | Si activé, un serveur défaillant en TCP est critique (sinon avertissement). TCP/53 est exigé par la RFC 7766 et par DNSSEC. | `true` |
| `checkEDNS` | Sonde chaque serveur de noms pour EDNS0 (RFC 6891). Les serveurs qui gèrent mal EDNS0 cassent DNSSEC et les grandes réponses. | `true` |
| `checkLatency` | Mesure le temps de réponse de chaque serveur et avertit en cas de lenteur. | `true` |
| `latencyThresholdMs` | Les temps de réponse au-delà de cette valeur déclenchent un avertissement de lenteur. | `500` |
| `useParentNS` | Interroge le parent pour le RRset `NS` de délégation et le compare aux serveurs de noms déclarés dans le service. | `true` |
| `warnOnStaleSaved` | Avertit lorsque le numéro de série `SOA` enregistré est plus récent que celui publié par les serveurs faisant autorité. | `true` |
| `minNameServers` | En dessous de ce nombre, un avertissement est émis (la RFC 1034 recommande au moins 2). | `2` |

## Dans happyDomain

Activez le vérificateur Cohérence des serveurs faisant autorité depuis l'onglet **Vérifications** d'un service d'origine. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet. Pour comparer ce que voient les *résolveurs récursifs à travers le monde* à la réponse faisant autorité, associez-le à {{< relref "/reference/checkers/resolver-propagation" >}}.
