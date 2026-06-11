---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Propagation chez les résolveurs
description: "Sonde un catalogue mondial de résolveurs récursifs publics sur plusieurs transports et régions, puis compare leurs réponses aux serveurs faisant autorité de la zone."
weight: 100
---

Le vérificateur **Propagation chez les résolveurs** mesure la façon dont une zone est vue depuis l'*extérieur*, à travers l'internet public. Il sonde un catalogue choisi de résolveurs récursifs publics (Cloudflare, Google, Quad9, OpenDNS, Yandex, FAI régionaux et autres) sur plusieurs transports (UDP, TCP, DoT, DoH) et depuis plusieurs régions, puis compare leurs réponses entre elles et avec celles des serveurs faisant autorité de la zone. Cela révèle les retards de propagation, les divergences régionales, les écarts de numéro de série `SOA`, les caches périmés, les échecs de validation DNSSEC, les incohérences `SERVFAIL`/`NXDOMAIN` et le filtrage par les résolveurs.

Ce vérificateur s'applique au niveau du **service** : il cible un service d'origine ou d'origine NS-seule (`abstract.Origin`, `abstract.NSOnlyOrigin`) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

| Code de constat | Ce qui est vérifié | Sévérité |
|---|---|---|
| `resolver_propagation.selection` | Le jeu d'options courant sélectionne au moins un résolveur public. | Critique |
| `resolver_propagation.reachable` | Au moins un résolveur sélectionné a répondu à une requête. | Critique |
| `resolver_propagation.latency` | Résolveurs injoignables ou dont le temps de réponse moyen dépasse le seuil. | Avertissement |
| `resolver_propagation.filtered_hit` | Résolveurs filtrants renvoyant une réponse différente du consensus (comportement typique d'une liste de blocage). | Info |
| `resolver_propagation.consensus` | Les résolveurs publics s'accordent sur une réponse unique pour chaque RRset sondé. | Avertissement |
| `resolver_propagation.matches_authoritative` | Le consensus public correspond à la réponse servie par les serveurs faisant autorité de la zone. | Critique |
| `resolver_propagation.nxdomain` | RRset pour lesquels certains résolveurs renvoient `NXDOMAIN` quand d'autres renvoient `NOERROR`. | Critique |
| `resolver_propagation.servfail` | RRset pour lesquels un résolveur renvoie `SERVFAIL` (généralement échec DNSSEC ou de joignabilité). | Critique |
| `resolver_propagation.regional_split` | Régions où tous les résolveurs s'accordent sur une réponse différente du consensus mondial. | Avertissement |
| `resolver_propagation.serial_drift` | Désaccord sur le numéro de série `SOA` parmi les résolveurs non filtrants. | Avertissement |
| `resolver_propagation.stale_cache` | Résolveurs servant encore un numéro de série `SOA` inférieur à celui enregistré par happyDomain. | Info |
| `resolver_propagation.dnssec` | Les résolveurs validants valident avec succès la chaîne DNSSEC de la zone. | Critique |

## Options

| Option | Signification | Défaut |
|---|---|---|
| `recordTypes` | Types de RR à sonder à chaque propriétaire, séparés par des virgules. Vide : dérivés de la zone de travail (SOA/NS à l'apex plus les types de RR réellement définis sur chaque propriétaire). | _dérivé de la zone_ |
| `subdomains` | Noms de propriétaires à sonder en plus de l'apex, séparés par des virgules (par exemple `www,mail,@`). Vide : apex seul. | `www` |
| `includeFiltered` | Sonde les résolveurs filtrants (malware/famille/anti-pub). Leurs réponses divergent par conception ; n'activer que pour diagnostiquer un blocage. | `false` |
| `region` | Restreint à une région : `all`, `global`, `na`, `eu`, `asia`, `ru`, `me`. | `all` |
| `transports` | Transports à sonder, séparés par des virgules : `udp`, `tcp`, `dot`, `doh`. Les transports chiffrés ne sont utilisés que là où ils sont publiés. | `udp` |
| `resolverAllowlist` | Identifiants ou IP de résolveurs à sonder exclusivement, séparés par des virgules (par exemple `cloudflare,google,9.9.9.9`). Vide : sélection du catalogue. | _(vide)_ |
| `latencyThresholdMs` | Les résolveurs dont la moyenne dépasse cette valeur émettent un constat d'information. | `500` |
| `runTimeoutSeconds` | Budget temps absolu pour une exécution de propagation. Les résolveurs plus lents sont signalés comme injoignables. | `30` |

## Dans happyDomain

Activez le vérificateur Propagation chez les résolveurs depuis l'onglet **Vérifications** d'un service d'origine. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet. Ce vérificateur est le pendant tourné vers l'extérieur de {{< relref "/reference/checkers/authoritative-consistency" >}}, qui examine directement les serveurs faisant autorité ; exécuter les deux donne la vue depuis l'origine et depuis les résolveurs qui l'interrogent.
