---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Délégation
description: "Audite la délégation d'une zone : cohérence des NS entre parent et enfant, exactitude des glue, passage de relais DS/DNSKEY, joignabilité et autorité de chaque serveur délégué."
weight: 70
---

Le vérificateur **Délégation** audite la façon dont une zone est déléguée depuis son parent. Il confronte la zone parente et les serveurs de noms enfants pour confirmer la cohérence du passage de relais : le parent et l'enfant s'accordent sur l'ensemble des `NS`, les enregistrements de glue sont corrects, la chaîne DNSSEC `DS`/`DNSKEY` est alignée, et chaque serveur délégué est joignable et réellement faisant autorité pour la zone.

Ce vérificateur s'applique au niveau du **service** : il cible un service de délégation (`abstract.Delegation`) publié sur un sous-domaine, et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

Chaque règle émet un `code` de constat stable, afin que les résultats puissent être appariés de façon déterministe.

### Nombre de serveurs de noms et découverte du parent

| Code de constat | Ce qui est vérifié |
|---|---|
| `delegation_too_few_ns` | La zone déclare au moins `minNameServers` enregistrements `NS` (la RFC 1034 recommande ≥ 2). |
| `delegation_no_parent_ns` | La zone parente et ses serveurs faisant autorité peuvent être découverts. |
| `delegation_parent_query_failed` | Chaque serveur de noms parent répond à la requête `NS` pour la zone déléguée. |
| `delegation_parent_tcp_failed` | Chaque serveur de noms parent est joignable en TCP (RFC 7766). |

### NS et glue au parent

| Code de constat | Ce qui est vérifié |
|---|---|
| `delegation_ns_mismatch` | Le RRset `NS` au parent correspond à l'ensemble `NS` déclaré par le service. |
| `delegation_missing_glue` | Les serveurs de noms dans le bailliage disposent de glue (`A`/`AAAA`) au parent. |
| `delegation_unnecessary_glue` | Les serveurs de noms hors bailliage ne portent pas de glue superflue. |

### Passage de relais DNSSEC

| Code de constat | Ce qui est vérifié |
|---|---|
| `delegation_ds_query_failed` | Le RRset `DS` peut être interrogé auprès des serveurs de noms parents. |
| `delegation_ds_mismatch` | Le RRset `DS` au parent correspond à l'ensemble `DS` déclaré par le service. |
| `delegation_ds_missing` | Des enregistrements `DS` sont présents au parent lorsque DNSSEC est attendu (conditionné par `requireDS`). |
| `delegation_ds_rrsig_invalid` | Le RRset `DS` est couvert par un `RRSIG` valide au parent. |
| `delegation_dnskey_query_failed` | Le RRset `DNSKEY` peut être interrogé auprès de chaque serveur de noms enfant. |
| `delegation_dnskey_no_match` | Au moins un `DNSKEY` enfant correspond à un condensat `DS` publié au parent. |

### Joignabilité et autorité des serveurs enfants

| Code de constat | Ce qui est vérifié |
|---|---|
| `delegation_ns_unresolvable` | Chaque nom de serveur déclaré se résout en au moins une adresse. |
| `delegation_unreachable` | Chaque serveur de noms enfant répond aux requêtes DNS sur ses adresses annoncées. |
| `delegation_lame` | Chaque serveur de noms enfant fait autorité pour la zone (pas de délégation boiteuse). |
| `delegation_no_authoritative_answer` | Chaque serveur de noms enfant positionne le drapeau AA dans ses réponses pour la zone. |
| `delegation_tcp_failed` | Chaque serveur de noms enfant répond en TCP (conditionné par `requireTCP`). |
| `delegation_soa_serial_drift` | Le numéro de série `SOA` est cohérent entre tous les serveurs de noms enfants. |
| `delegation_ns_drift` | Le RRset `NS` renvoyé par chaque enfant correspond au RRset `NS` au parent. |
| `delegation_glue_mismatch` | Les adresses de glue chez l'enfant correspondent à celles du parent (conditionné par `allowGlueMismatch`). |

## Options

| Option | Signification | Défaut |
|---|---|---|
| `requireDS` | Si activé, l'absence de `DS` au parent est traitée comme critique (sinon informative). | `false` |
| `requireTCP` | Si activé, les serveurs de noms qui ne répondent pas en TCP sont signalés comme critiques (sinon en avertissement). | `true` |
| `minNameServers` | En dessous de ce nombre, la délégation est signalée en avertissement (la RFC 1034 recommande au moins 2). | `2` |
| `allowGlueMismatch` | Si désactivé, les divergences de glue/adresses entre parent et enfant sont signalées comme critiques. | `false` |

## Dans happyDomain

Activez le vérificateur Délégation depuis l'onglet **Vérifications** d'un service de délégation. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet. Pour la cohérence *entre* les serveurs faisant autorité de l'apex lui-même (plutôt que le passage de relais parent/enfant), voyez {{< relref "/reference/checkers/authoritative-consistency" >}} ; pour l'hygiène DNSSEC de la zone signée, voyez {{< relref "/reference/checkers/dnssec" >}}.
