---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Zone inverse
description: "Inspecte les enregistrements PTR d'une zone inverse in-addr.arpa ou ip6.arpa : FCrDNS, résolvabilité des cibles, syntaxe des noms, noms génériques et hygiène des TTL."
weight: 110
---

Le vérificateur **Zone inverse** inspecte les enregistrements `PTR` d'une zone DNS inverse (`in-addr.arpa` ou `ip6.arpa`) et valide qu'ils sont bien formés et cohérents avec le DNS direct. Il vérifie le Forward-Confirmed Reverse DNS (FCrDNS), s'assure que les cibles se résolvent et sont des noms d'hôtes syntaxiquement valides, signale les noms génériques ou auto-générés et les TTL trop courts, et détecte les violations de plusieurs `PTR` par IP (RFC 1912 §2.1). Un DNS inverse correct compte en pratique : les serveurs de courrier et les points d'accès SSH rejettent ou dégradent régulièrement les connexions provenant d'IP sans FCrDNS valide.

Ce vérificateur s'applique au niveau de la **zone** : il opère sur le contenu complet d'une zone inverse (il s'applique au domaine et lit l'intégralité de la zone).

## Ce qu'il vérifie

| Code de constat | Ce qui est vérifié | Sévérité |
|---|---|---|
| `reverse_zone.is_reverse_arpa` | La zone est sous `in-addr.arpa` ou `ip6.arpa`. | Critique |
| `reverse_zone.has_ptrs` | La zone inverse déclare au moins un enregistrement `PTR`. | Avertissement |
| `reverse_zone.fcrdns` | Le `A`/`AAAA` de chaque cible `PTR` revient bien à l'IP d'origine (Forward-Confirmed Reverse DNS). | Critique |
| `reverse_zone.target_resolves` | Chaque cible `PTR` se résout en au moins un enregistrement `A` ou `AAAA`. | Critique |
| `reverse_zone.single_ptr_per_ip` | Signale les IP ayant plusieurs `PTR` (la RFC 1912 §2.1 en recommande exactement un). | Avertissement |
| `reverse_zone.target_syntax` | Chaque cible `PTR` est un nom d'hôte syntaxiquement valide. | Critique |
| `reverse_zone.generic_hostname` | Signale les cibles `PTR` qui intègrent l'IP ou correspondent aux motifs auto-générés courants des FAI. | Avertissement |
| `reverse_zone.ttl_hygiene` | Signale les `PTR` dont le TTL est inférieur au minimum configuré. | Avertissement |
| `reverse_zone.truncated` | Signale lorsque la zone compte plus de `PTR` que le plafond configuré ne permet d'inspecter. | Info |

## Options

| Option | Signification | Défaut |
|---|---|---|
| `requireForwardMatch` | Si activé, un `PTR` dont la cible ne revient pas à l'IP d'origine est critique (sinon avertissement). Les serveurs de courrier et SSH exigent le FCrDNS. | `true` |
| `allowMultiplePTR` | Si désactivé, plus d'un `PTR` au même propriétaire est signalé en avertissement (la RFC 1912 §2.1 recommande un seul `PTR` par IP). | `false` |
| `minTTL` | Les `PTR` dont le TTL est inférieur à ce seuil (en secondes) sont signalés en avertissement. | `300` |
| `flagGenericPTR` | Si activé, les cibles `PTR` qui intègrent l'IP pointée ou correspondent aux motifs auto-générés courants des FAI sont signalées en avertissement. | `true` |
| `maxPTRsToCheck` | Plafonne le nombre de `PTR` inspectés par exécution, protégeant le vérificateur contre les très grandes zones inverses. | `1024` |

{{% notice style="info" title="Forward-Confirmed Reverse DNS" %}}
Le FCrDNS signifie que la cible du `PTR`, résolue en direct, revient bien à l'adresse IP d'origine. Un `PTR` pointant vers un hôte dont le `A`/`AAAA` n'inclut pas cette IP échoue à l'aller-retour et est considéré comme une mauvaise configuration par de nombreux serveurs de courrier et SSH.
{{% /notice %}}

## Dans happyDomain

Activez le vérificateur Zone inverse sur un domaine inverse (`in-addr.arpa` / `ip6.arpa`) depuis sa vue **Vérifications**. Consultez {{< relref "/pages/checks" >}} pour le déroulé complet.
