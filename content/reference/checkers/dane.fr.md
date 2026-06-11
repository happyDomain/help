---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: DANE / TLSA
description: "Compare les enregistrements TLSA d'un service à la chaîne de certificats réellement présentée par chaque point d'accès, selon la RFC 6698."
weight: 160
---

Le vérificateur **DANE / TLSA** vérifie que les enregistrements `TLSA` publiés pour un service épinglent correctement le certificat TLS que le point d'accès correspondant présente réellement. DANE (DNS-based Authentication of Named Entities) permet à un domaine de lier un certificat ou une clé publique à un nom via le DNS, sécurisé par DNSSEC ; ce vérificateur confirme que la liaison tient.

Il s'agit d'un vérificateur de **niveau service** : il s'exécute sur un service `TLSAs`. Il regroupe les enregistrements TLSA déclarés par `(port, proto, base)`, publie une entrée de découverte de point d'accès TLS par point d'accès afin que `checker-tls` le sonde, puis compare chaque TLSA à la chaîne de certificats observée selon la RFC 6698.

## Ce qui est vérifié

| Règle | Ce qui est vérifié ou signalé | Sévérité |
|-------|-------------------------------|----------|
| `dane.has_records` | Au moins un enregistrement TLSA est déclaré sur le service. | variable |
| `dane.dnssec_validated` | Les enregistrements TLSA ont été récupérés via un résolveur validant DNSSEC (bit AD positionné). | variable |
| `dane.probe_available` | Une sonde TLS est disponible pour chaque point d'accès DANE afin de comparer la chaîne. | variable |
| `dane.handshake_ok` | La poignée de main TLS réussit sur chaque point d'accès DANE. | variable |
| `dane.records_match_chain` | Au moins un enregistrement TLSA correspond à la chaîne de certificats présentée par chaque point d'accès. | variable |
| `dane.pkix_chain_valid` | Lorsque les usages 0 ou 1 sont publiés, la chaîne se valide aussi face aux racines de confiance du système. | variable |
| `dane.usage_coherent` | Signale les TLSA dont l'usage déclaré ne correspond pas à l'emplacement de chaîne qu'ils hachent réellement (par exemple un usage 3 correspondant à un intermédiaire). | variable |

### Interprétation des champs TLSA

- **Usage 0 (PKIX-TA) / 1 (PKIX-EE)** : le TLSA doit correspondre *et* la chaîne doit se valider face aux racines de confiance PKIX publiques.
- **Usage 2 (DANE-TA) / 3 (DANE-EE)** : le TLSA fait lui-même office d'ancre de confiance ; la validité PKIX est informative.
- Le **sélecteur** 0 (certificat complet) ou 1 (Subject Public Key Info), et le **type de correspondance** 0 (complet) / 1 (SHA-256) / 2 (SHA-512), sont comparés à l'emplacement de chaîne impliqué par l'usage.

{{% notice style="info" title="DANE repose sur DNSSEC" %}}
DANE n'est digne de confiance que si les enregistrements TLSA proviennent d'une zone signée DNSSEC et sont validés par le résolveur. La règle `dane.dnssec_validated` vérifie que les enregistrements sont arrivés avec le bit AD (Authenticated Data) positionné ; sans DNSSEC, un enregistrement TLSA pourrait être falsifié et toute la liaison perd son sens.
{{% /notice %}}

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Délai de sonde (ms) | Transmis à `checker-tls` comme délai d'expiration de la sonde TLS par point d'accès. | défaut de `checker-tls` |
| Surcharge STARTTLS | Une table indexée par `"<port>/<proto>"` surchargeant l'application STARTTLS utilisée lors de la sonde. Les ports STARTTLS courants (25, 110, 143, 389, 587, 5222, 5269) sont associés automatiquement ; ne renseignez ceci que pour des ports non standard. | (auto) |

Le domaine, le sous-domaine et le service TLSAs sont renseignés automatiquement.

## Dans happyDomain

Ajoutez le service TLSA au sous-domaine, puis activez ce vérificateur depuis l'onglet **Vérifications** de ce service. Consultez {{< relref "/pages/checks" >}} pour configurer et planifier les vérifications. Le vérificateur publie ses points d'accès pour que `checker-tls` les sonde : laissez donc passer un cycle de sonde avant l'apparition du premier résultat de correspondance.

Vérificateur apparenté : {{< relref "/reference/checkers/caa" >}} vérifie, sur les mêmes certificats, que l'autorité émettrice était bien autorisée par la politique CAA du domaine.
