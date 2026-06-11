---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SIP
description: "Sonde le déploiement SIP/VoIP d'un domaine de bout en bout : résolution NAPTR/SRV (RFC 3263), accessibilité des points d'accès en UDP/TCP/TLS et un ping SIP OPTIONS avec inspection des capacités."
weight: 270
---

Le vérificateur **SIP** sonde le déploiement SIP/VoIP d'un domaine à partir de ses enregistrements DNS, en suivant la résolution de la RFC 3263 : NAPTR → SRV (`_sip._udp`, `_sip._tcp`, `_sips._tcp`) → A/AAAA. Il teste l'accessibilité de chaque `cible:port` résolue en UDP, TCP et TLS, puis envoie un ping SIP `OPTIONS` brut et inspecte la réponse (ligne de statut, `Server`/`User-Agent`, méthodes `Allow` annoncées, temps d'aller-retour).

Il s'agit d'un vérificateur de **niveau service** : il cible un service *SIP* (`abstract.SIP`) publié sur un sous-domaine et se configure depuis l'onglet **Vérifications** de ce service. Lorsqu'aucun enregistrement SRV n'est publié, le vérificateur se rabat sur `<domaine>:5060` / `<domaine>:5061`, avec un marqueur d'information visible dans le rapport.

{{% notice style="info" title="La posture TLS est intégrée, pas dupliquée" %}}
Pour la poignée de main TLS, le vérificateur utilise `InsecureSkipVerify` : il confirme uniquement qu'une session TLS peut être établie. Chaque cible `_sips._tcp` est publiée comme entrée de découverte `tls.endpoint.v1` afin que le vérificateur TLS dédié puisse vérifier la chaîne de certificats, la correspondance du nom d'hôte, l'expiration et la posture des algorithmes. Ces constats sont réintégrés sur la page du service SIP via la règle `sip.tls_quality`. Consultez {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qui est vérifié | Gravité |
|---|---|---|
| `sip.srv_present` | Les enregistrements SRV `_sip._udp` / `_sip._tcp` / `_sips._tcp` sont publiés et résolvables. | Critique |
| `sip.transport_diversity` | Des transports SIP modernes (TCP, et idéalement TLS) sont publiés en plus de l'UDP historique. | Avertissement |
| `sip.srv_targets_resolvable` | Chaque cible SRV se résout en au moins une adresse A ou AAAA. | Critique |
| `sip.endpoint_reachable` | Chaque point d'accès SIP découvert accepte une connexion sur son transport. | Critique |
| `sip.options_response` | Chaque point d'accès SIP accessible répond à `OPTIONS` par une réponse 2xx. | Critique |
| `sip.options_capabilities` | Examine l'en-tête `Allow` annoncé dans les réponses `OPTIONS` (prise en charge d'INVITE, présence d'`Allow`). | Avertissement |
| `sip.ipv6_coverage` | Au moins un point d'accès SIP est accessible en IPv6. | Info |
| `sip.tls_quality` | Réintègre les constats du vérificateur TLS aval (chaîne, correspondance du nom d'hôte, expiration) sur le service SIP. | Critique |

Le vérificateur effectue, dans l'ordre : la recherche NAPTR (`SIP+D2U`, `SIP+D2T`, `SIPS+D2T`), la recherche SRV pour les trois transports (avec le repli `5060`/`5061`), la résolution A/AAAA de chaque cible SRV, la connexion TCP / l'envoi UDP / la poignée de main TLS, et la requête SIP `OPTIONS` avec son statut, ses en-têtes et son `Allow` analysés.

## Options

| Option | Signification | Défaut |
|---|---|---|
| Domaine SIP | Le domaine à tester (renseigné automatiquement depuis la portée du service). Obligatoire. | *(auto)* |
| Délai d'attente par point d'accès (secondes) | Délai de sonde pour chaque point d'accès. | `5` |
| Sonder `_sip._udp` | Faut-il sonder le transport UDP. À désactiver si l'UDP est filtré ou si l'hôte du vérificateur ne peut pas émettre d'UDP. | `true` |
| Sonder `_sip._tcp` | Faut-il sonder le transport TCP. | `true` |
| Sonder `_sips._tcp` (TLS) | Faut-il sonder le transport TLS. | `true` |

## Dans happyDomain

Activez le vérificateur SIP depuis l'onglet **Vérifications** d'un service SIP. Le domaine est renseigné automatiquement. Consultez {{< relref "/pages/checks" >}} pour le fonctionnement complet, et {{< relref "/reference/checkers/tls" >}} pour la posture des certificats des points d'accès `_sips._tcp`.
