---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Validation DNSSEC
description: "Audite l'hygiène opérationnelle d'une zone signée : algorithmes, taille des clés, fraîcheur des signatures, paramètres NSEC/NSEC3 et cohérence entre serveurs."
weight: 60
---

Le vérificateur **DNSSEC** audite l'*hygiène opérationnelle* d'une zone signée. Il ne revalide pas de bout en bout la chaîne de confiance cryptographique (cette tâche est confiée à un vérificateur dédié fondé sur DNSViz) : il se concentre sur les aspects de politique et d'exploitation au quotidien de la signature, à savoir les algorithmes et tailles de clés utilisés, la validité et la fraîcheur des signatures, le mode de déni d'existence (NSEC ou NSEC3) et la cohérence de la vue servie par chaque serveur faisant autorité.

Ce vérificateur s'applique au niveau du **domaine** : il opère sur l'apex de la zone. À partir d'un résolveur récursif d'amorçage, il découvre les serveurs de noms de l'apex et interroge le `DS` parent, puis questionne directement chaque serveur faisant autorité.

## Ce qu'il vérifie

Le vérificateur évalue les règles suivantes. Plusieurs d'entre elles sont pilotées par les options décrites plus bas.

### Chaîne et matériel de clés

| Règle | Vérifie | Sévérité |
|---|---|---|
| `dnssec_zone_signed` | Le parent annonce un `DS` mais l'apex ne sert aucun `DNSKEY` (zone signée cassée). | Critique |
| `dnssec_dnskey_consistent` | Tous les serveurs faisant autorité renvoient le même RRset `DNSKEY`. | Critique |
| `dnssec_dnskey_query_ok` | Tous les serveurs faisant autorité ont répondu à la requête `DNSKEY`. | Critique |
| `dnssec_ksk_present` | Au moins un `DNSKEY` porte le bit SEP (clé de signature de clé, KSK). | Critique |
| `dnssec_dnskey_count` | Avertit lorsque trop de `DNSKEY` sont publiés, ce qui gonfle les réponses et le potentiel d'amplification. | Avertissement |

### Algorithmes et robustesse des clés

| Règle | Vérifie | Sévérité |
|---|---|---|
| `dnssec_algorithm_allowed` | Aucun `DNSKEY` n'utilise un algorithme interdit ou hors de la liste autorisée. | Critique |
| `dnssec_algorithm_modern` | Recommande ECDSAP256SHA256 (13) ou Ed25519 (15) plutôt que RSA. | Avertissement |
| `dnssec_rsa_keysize` | Les `DNSKEY` RSA atteignent la taille de module minimale. | Critique |

### Signatures

| Règle | Vérifie | Sévérité |
|---|---|---|
| `dnssec_rrsig_present_dnskey` | Le RRset `DNSKEY` est signé. | Critique |
| `dnssec_rrsig_present_soa` | Le RRset `SOA` est signé. | Critique |
| `dnssec_rrsig_validity_window` | Chaque `RRSIG` observé est dans sa fenêtre d'inception/expiration. | Critique |
| `dnssec_rrsig_freshness` | Alerte anticipée lorsque les `RRSIG` approchent de l'expiration (détecte les signeurs bloqués). | Critique |

### Déni d'existence (NSEC / NSEC3)

| Règle | Vérifie | Sévérité |
|---|---|---|
| `dnssec_denial_uses_nsec3` | Avertit lorsque la zone utilise NSEC nu, ce qui la rend parcourable (RFC 5155 / RFC 7129). | Avertissement |
| `dnssec_nsec3_iterations` | `NSEC3PARAM.Iterations` ne dépasse pas le plafond configuré (la RFC 9276 §3.1 recommande 0). | Critique |
| `dnssec_nsec3_salt_empty` | `NSEC3PARAM.SaltLength` vaut 0 (RFC 9276 §3.1 : un sel n'apporte aucune protection mesurable). | Avertissement |
| `dnssec_nsec3_optout_only_when_signed_delegations` | Note informative lorsque le drapeau OPT-OUT est positionné dans une zone feuille. | Info |
| `dnssec_denial_consistent` | Tous les serveurs faisant autorité utilisent le même schéma de déni d'existence. | Avertissement |

### Hygiène des TTL

| Règle | Vérifie | Sévérité |
|---|---|---|
| `dnssec_dnskey_ttl_min` | Avertit lorsque le TTL des `DNSKEY` est trop court pour être utile au cache. | Avertissement |

## Options

| Option | Signification | Défaut |
|---|---|---|
| `nsec3IterationsMax` | Plafond RFC 9276 §3.1 sur `NSEC3PARAM.Iterations`. À relever uniquement si votre signeur ne sait pas encore publier 0. | `0` |
| `nsec3IterationsSeverity` | Sévérité lorsque les itérations dépassent le plafond. Mettre `crit` pour appliquer strictement la RFC 9276. | `warn` |
| `signatureFreshness` | Avertit lorsque le `RRSIG` le plus proche expire dans moins de ce nombre de jours. | `7` |
| `signatureFreshnessCrit` | Critique lorsque le `RRSIG` le plus proche expire dans moins de ce nombre de jours. | `1` |
| `minRSAKeySize` | Taille de module RSA minimale acceptable, en bits. | `2048` |
| `requireSEP` | Exige au moins un `DNSKEY` portant le bit SEP (KSK). | `true` |
| `dnskeyTTLMin` | TTL minimal des `DNSKEY`, en secondes ; des TTL plus courts nuisent à la mise en cache. | `3600` |

L'administrateur peut également définir un résolveur d'amorçage (`resolver`, au format `host:port`) servant à découvrir les serveurs de noms de l'apex et à interroger le `DS` parent ; sa valeur par défaut est `/etc/resolv.conf`.

{{% notice style="info" title="Ce que ce vérificateur ne fait pas" %}}
Le vérificateur DNSSEC ne vérifie pas l'intégralité de la chaîne de confiance cryptographique depuis la racine. Pour cette validation de bout en bout, utilisez le vérificateur fondé sur DNSViz. Le présent vérificateur le complète en détectant les problèmes de politique et d'exploitation (algorithmes faibles, signatures expirantes, NSEC parcourable) qu'une validation de chaîne seule ne révélerait pas.
{{% /notice %}}

## Dans happyDomain

Activez le vérificateur DNSSEC sur un domaine depuis sa vue **Vérifications**. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet de l'ajout, de la planification et de la lecture des vérifications. Pour le passage de relais `DS`/`DNSKEY` côté délégation, voyez le vérificateur {{< relref "/reference/checkers/delegation" >}}.
