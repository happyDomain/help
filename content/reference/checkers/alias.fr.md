---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Chaîne d'alias
description: "Parcourt une chaîne CNAME/DNAME/ALIAS et valide le nombre de sauts, les TTL, la résolvabilité de la cible, la coexistence à l'apex et la signature DNSSEC."
weight: 130
---

Le vérificateur **Chaîne CNAME / DNAME / ALIAS** suit la chaîne d'alias d'un nom et vérifie qu'elle se résout proprement : pas de boucle, pas trop de sauts, des TTL raisonnables, une cible finale résolvable, aucune coexistence d'enregistrements interdite et un RRset CNAME correctement signé lorsque la zone utilise DNSSEC.

Il s'agit d'un vérificateur de **niveau service** : il s'exécute sur un service `CNAME` (ou CNAME spécial) et se configure depuis l'onglet **Vérifications** de ce service. À partir du nom interrogé, il localise l'apex de la zone, parcourt chaque saut CNAME/DNAME, puis résout la cible de la chaîne vers des enregistrements A/AAAA.

## Ce qui est vérifié

Chaque règle émet un code de constat. Certaines sévérités sont atténuées par les options ci-dessous.

| Règle | Ce qui est vérifié ou signalé | Sévérité |
|-------|-------------------------------|----------|
| `apex_lookup` | L'apex de la zone (SOA) du nom interrogé peut être localisé. | Critique |
| `chain_loop` | Un cycle CNAME/DNAME dans la chaîne de résolution. | Critique |
| `chain_length` | La chaîne dépasse le nombre de sauts de **Longueur maximale de chaîne**. | Critique |
| `chain_query_error` | Une requête DNS échoue pendant le parcours de la chaîne (erreur réseau, délai dépassé). | Avertissement |
| `chain_rcode` | Un code de réponse autre que NOERROR en milieu de chaîne ou sur la résolution A/AAAA finale. | Critique (milieu) / Avertissement (final) |
| `hop_ttl` | Un saut CNAME/DNAME a un TTL inférieur à **TTL minimal de la cible**. | Avertissement |
| `cname_at_apex` | Un CNAME existe à l'apex de la zone, en conflit avec SOA/NS (RFC 1912 §2.4). | Critique / Avertissement |
| `apex_flattening` | Des A/AAAA coexistent avec SOA/NS à l'apex sans CNAME (aplatissement ALIAS/ANAME côté hébergeur). | Info |
| `cname_coexistence` | D'autres RRset (au-delà de A/AAAA) coexistent chez un propriétaire CNAME, en violation des RFC 1034 §3.6.2 / RFC 2181 §10.1. | Critique / Avertissement à l'apex |
| `cname_dnssec` | La zone est signée DNSSEC mais le RRset CNAME ne possède pas de RRSIG. | Critique |
| `target_resolvable` | La cible finale de la chaîne n'a aucun enregistrement A ni AAAA. | Critique |
| `multiple_records` | Un propriétaire de la chaîne porte plus d'un enregistrement CNAME/DNAME (malformé). | Critique |

{{% notice style="info" title="Pourquoi un CNAME à l'apex pose problème" %}}
Un propriétaire de CNAME ne peut porter aucun autre type d'enregistrement, mais l'apex de la zone doit toujours contenir des enregistrements SOA et NS. Ces deux exigences sont incompatibles : un CNAME à l'apex est donc invalide (RFC 1912 §2.4). Certains hébergeurs contournent cela par un aplatissement ALIAS/ANAME côté serveur qui publie de simples A/AAAA à l'apex ; la règle `apex_flattening` reconnaît ce motif comme intentionnel lorsque l'option **Reconnaître l'aplatissement ALIAS/ANAME** est activée.
{{% /notice %}}

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Longueur maximale de chaîne | Au-delà de ce nombre de sauts, la chaîne est rapportée comme critique. | `8` |
| TTL minimal de la cible | Les sauts dont le TTL est inférieur à ce seuil (en secondes) sont signalés comme avertissement. | `60` |
| Autoriser un CNAME à l'apex | Lorsqu'activée, un CNAME à l'apex d'une zone et ses violations de coexistence sont rétrogradés en avertissements. La RFC 1912 l'interdit : il est fortement recommandé de laisser cette option désactivée. | `false` |
| Reconnaître l'aplatissement ALIAS/ANAME | Lorsqu'activée, les hébergeurs servant des A/AAAA à l'apex (pseudo-enregistrements ALIAS/ANAME) sont reconnus comme intentionnels et dispensés des violations de coexistence. | `true` |

## Dans happyDomain

Ajoutez le service CNAME au sous-domaine, puis activez ce vérificateur depuis l'onglet **Vérifications** de ce service. Consultez {{< relref "/pages/checks" >}} pour configurer et planifier les vérifications. Le domaine parent et le sous-domaine sont renseignés automatiquement.

Vérificateurs apparentés : {{< relref "/reference/checkers/dangling" >}} surveille les cibles d'alias devenues non enregistrées ou NXDOMAIN, et {{< relref "/reference/checkers/ptr" >}} couvre le versant DNS inverse.
