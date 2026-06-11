---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: XMPP
description: "Sonde le déploiement XMPP d'un domaine : découverte SRV, accessibilité, STARTTLS, mécanismes SASL et authentification de fédération."
weight: 290
---

Le vérificateur **XMPP** sonde de bout en bout le déploiement XMPP (Jabber) d'un domaine, à la manière de [xmpp.net](https://xmpp.net/) : il découvre les enregistrements SRV pertinents, ouvre un flux vers chaque point d'accès, négocie STARTTLS, inspecte les mécanismes SASL proposés et confirme que la fédération de serveur à serveur peut s'authentifier.

Il s'agit d'un vérificateur de **niveau service**. Il s'applique aux services de type **XMPP** et se configure depuis l'onglet **Vérifications** de ce service. Il sonde les quatre noms de service standard (`_xmpp-client._tcp`, `_xmpp-server._tcp`, `_xmpps-client._tcp`, `_xmpps-server._tcp`), l'ancien `_jabber._tcp`, et se rabat sur `<domaine>:5222` / `:5269` lorsqu'aucun enregistrement SRV n'est publié.

{{% notice style="info" title="La posture TLS est vérifiée séparément" %}}
La chaîne de certificats, la correspondance du nom d'hôte (SAN), l'expiration et la posture des chiffrements sont **hors périmètre** ici : un vérificateur {{< relref "/reference/checkers/tls" >}} dédié s'en charge. Le vérificateur XMPP confirme seulement que STARTTLS aboutit, enregistre la version et le chiffrement TLS négociés à titre de contexte, et reverse les conclusions TLS dans le rapport du service XMPP via la règle `xmpp.tls_quality`.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qu'elle vérifie | Sévérité |
|-------|--------------------|----------|
| `xmpp.srv_c2s` | Les enregistrements SRV client vers serveur (`_xmpp-client` / `_xmpps-client` / `_jabber`) sont publiés et résolvables. | Critique |
| `xmpp.srv_s2s` | Les enregistrements SRV serveur vers serveur (`_xmpp-server` / `_xmpps-server`) sont publiés et résolvables. | Critique |
| `xmpp.c2s_reachable` | Au moins un point d'accès client vers serveur accepte le TCP et achève le TLS. | Critique |
| `xmpp.s2s_reachable` | Au moins un point d'accès serveur vers serveur accepte le TCP et achève le TLS. | Critique |
| `xmpp.starttls_required` | STARTTLS est annoncé et requis sur chaque point d'accès c2s/s2s joignable. | Critique |
| `xmpp.sasl_mechanisms` | L'offre SASL c2s est saine (présence de SCRAM, absence d'un PLAIN seul équivalent à un mot de passe en clair). | Critique |
| `xmpp.s2s_dialback` | Les points d'accès serveur vers serveur annoncent le dialback ou SASL EXTERNAL après le TLS (authentification de fédération). | Critique |
| `xmpp.ipv6_reachable` | Signale les déploiements joignables uniquement en IPv4. | Info |
| `xmpp.direct_tls` | Signale les déploiements c2s qui ne publient pas d'enregistrements SRV direct-TLS XEP-0368 (`_xmpps-*`). | Info |
| `xmpp.tls_quality` | Reverse sur le service XMPP les conclusions du vérificateur TLS sous-jacent (chaîne de certificats, correspondance du nom d'hôte, expiration). | Critique |

La sonde couvre aussi l'accessibilité TCP des cibles A/AAAA, l'analyse des fonctionnalités de flux et la couverture IPv4/IPv6, exposées au travers des règles ci-dessus et du rapport HTML.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Domaine | Domaine XMPP (domaine du JID) à tester. Renseigné automatiquement depuis le service. | (renseigné automatiquement) |
| Mode | Côté à sonder : `c2s` (client vers serveur), `s2s` (serveur vers serveur) ou `both` (les deux). | `both` |
| Délai par point d'accès (secondes) | Budget de temps alloué à chaque point d'accès sondé. | 10 |

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** d'un service XMPP ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. Le domaine est renseigné automatiquement depuis le service. Pour le volet certificat de ces mêmes points d'accès, associez-le au vérificateur {{< relref "/reference/checkers/tls" >}}.
