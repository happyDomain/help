---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Enregistrements PTR
description: "Valide le DNS inverse d'une adresse IP : présence du PTR, syntaxe de la cible, résolution directe et Forward-Confirmed Reverse DNS (FCrDNS)."
weight: 120
---

Le vérificateur **PTR / DNS inverse** vérifie qu'une adresse IP dispose d'un enregistrement de DNS inverse correct et exploitable. Un PTR associe une IP à un nom d'hôte ; les serveurs de messagerie, les démons SSH et de nombreux outils de journalisation s'appuient dessus, et un PTR absent ou incohérent est l'une des causes les plus fréquentes de rejet du courrier sortant.

Il s'agit d'un vérificateur de **niveau service** : il s'exécute sur un service `PTR` et se configure depuis l'onglet **Vérifications** de ce service. Il localise la zone inverse (sous `in-addr.arpa` ou `ip6.arpa`), interroge les serveurs faisant autorité et inspecte ce qu'ils servent réellement, aussi bien pour les adresses IPv4 qu'IPv6.

## Ce qui est vérifié

Le vérificateur enchaîne une série de règles, de la cohérence structurelle au succès de la requête, puis à l'hygiène de la cible et au Forward-Confirmed Reverse DNS (FCrDNS).

| Règle | Ce qui est vérifié | Sévérité |
|-------|--------------------|----------|
| `ptr.in_reverse_arpa` | Le propriétaire du PTR se trouve sous `in-addr.arpa` ou `ip6.arpa`. | Critique |
| `ptr.owner_decodable` | Le nom du propriétaire en `.arpa` se décode bien en une adresse IP. | Critique |
| `ptr.reverse_zone_located` | La zone inverse servant ce propriétaire peut être localisée (SOA trouvé). | Critique |
| `ptr.query_succeeded` | La requête PTR renvoie NOERROR depuis les serveurs faisant autorité. | Critique |
| `ptr.record_present` | Au moins un enregistrement PTR est servi au nom du propriétaire. | Critique |
| `ptr.single_record` | Signale plusieurs PTR sur la même IP (la RFC 1912 §2.1 en recommande un seul). | Avertissement |
| `ptr.declared_match` | La cible PTR servie fait autorité correspond à celle déclarée dans happyDomain. | Critique |
| `ptr.target_syntax_valid` | La cible PTR est un nom d'hôte syntaxiquement valide (RFC 952/1123). | Critique |
| `ptr.generic_hostname` | Signale les cibles PTR qui contiennent l'IP ou suivent un motif générique de FAI. | Avertissement |
| `ptr.target_resolves` | La cible PTR se résout vers au moins un enregistrement A ou AAAA. | Critique / Avertissement |
| `ptr.fcrdns_match` | Le A/AAAA de la cible PTR se résout de nouveau vers l'IP d'origine (FCrDNS). | Critique / Avertissement |
| `ptr.ipv6` | Indique si le PTR concerne une adresse IPv6 (`ip6.arpa`) et qu'un PTR est présent pour elle. | Critique |
| `ptr.ttl_hygiene` | Le TTL du PTR est supérieur ou égal au minimum configuré. | Avertissement |

Les règles `ptr.target_resolves` et `ptr.fcrdns_match` sont critiques par défaut, mais passent en avertissement lorsque l'option **Exiger le Forward-Confirmed Reverse DNS** est désactivée.

{{% notice style="info" title="Le FCrDNS, la règle de l'aller-retour" %}}
Le Forward-Confirmed Reverse DNS signifie que la chaîne boucle sur elle-même : le PTR de l'IP pointe vers un nom d'hôte, et le A/AAAA de ce nom d'hôte inclut l'IP d'origine. Les serveurs de messagerie rejettent les connexions des IP qui échouent à cet aller-retour ; laissez donc **Exiger le Forward-Confirmed Reverse DNS** activé pour tout hôte qui envoie du courrier.
{{% /notice %}}

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Exiger le Forward-Confirmed Reverse DNS (FCrDNS) | Lorsqu'activée, un PTR dont la cible ne se résout pas vers l'IP d'origine est critique ; sinon, c'est un avertissement. | `true` |
| Autoriser plusieurs PTR sur la même IP | Lorsqu'elle est désactivée, plusieurs PTR au même propriétaire sont signalés comme avertissement (RFC 1912 §2.1). | `false` |
| TTL minimal du PTR (secondes) | Les PTR dont le TTL est inférieur à ce seuil sont signalés comme avertissement. | `300` |
| Signaler les noms d'hôte PTR génériques | Lorsqu'activée, les cibles PTR contenant l'IP en notation pointée ou suivant un motif générique de FAI déclenchent un avertissement. | `true` |

## Dans happyDomain

Ajoutez le service PTR au sous-domaine portant l'enregistrement inverse, puis activez ce vérificateur depuis l'onglet **Vérifications** de ce service. Consultez {{< relref "/pages/checks" >}} pour configurer et planifier les vérifications. La zone inverse, l'enregistrement PTR et la cible déclarée sont renseignés automatiquement à partir du service.

Pour le versant direct de la résolution des alias et des noms d'hôte, voyez le vérificateur {{< relref "/reference/checkers/alias" >}}.
