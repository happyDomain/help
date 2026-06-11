---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: DNSViz
description: "Lance DNSViz sur un domaine et parcourt toute la chaîne DNSSEC, de la racine jusqu'à la feuille, en faisant remonter chaque erreur et avertissement à la zone exacte où il survient."
weight: 340
---

Le vérificateur **DNSViz** (nommé *DNSSEC (DNSViz)* dans happyDomain) évalue la chaîne de validation DNSSEC d'un domaine en pilotant [DNSViz](https://github.com/dnsviz/dnsviz). Il analyse le nom interrogé **et chacun de ses ancêtres jusqu'à la racine**, de sorte qu'un échec DNSSEC récursif peut être localisé au niveau exact, et à l'enregistrement exact, où il s'est produit.

Ce vérificateur s'applique au **niveau domaine** : il concerne le domaine et sa chaîne de délégation plutôt qu'un service isolé.

## Fonctionnement

DNSViz est exécuté en externe (un outil Python empaqueté avec le vérificateur), et happyDomain délègue la collecte à ce point d'accès. Chaque exécution revient à effectuer :

```
dnsviz probe -A . <ancêtres…> <domaine> | dnsviz grok -t <ancre-de-confiance-racine>
```

Chaque maillon de la chaîne (racine, TLD, intermédiaires, feuille) est listé explicitement afin que l'analyse complète apparaisse dans le rapport. `dnsviz grok` reçoit une ancre de confiance DNSKEY au format BIND pour la zone racine ; sans elle, la racine n'a pas de parent auquel se rattacher et reste classée au niveau DNS (`NOERROR`) au lieu de `SECURE` au sens DNSSEC.

La sortie est analysée : les erreurs et avertissements par zone sont extraits de l'arbre d'enregistrements imbriqués et transformés en états individuels, étiquetés du chemin JSON où ils ont été trouvés. Un catalogue d'échecs DNSSEC courants (chaîne rompue, RRSIG expirée, condensat DS incohérent, algorithme déprécié) est confronté aux constats pour alimenter une section « Fix these first » du rapport HTML.

{{% notice style="info" title="Périmètre" %}}
Ce vérificateur rapporte exactement ce que DNSViz rapporte. Le durcissement contre le parcours de zone NSEC/NSEC3 et la politique d'itérations NSEC3PARAM (RFC 9276) sont hors périmètre ici et traités par le vérificateur DNSSEC dédié : voir {{< relref "/reference/checkers/dnssec" >}}.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qui est vérifié |
|---|---|
| `dnsviz_overall_status` | Statut DNSViz du domaine interrogé (SECURE / INSECURE / BOGUS / INDETERMINATE). |
| `dnsviz_per_zone_status` | Un état par zone de la chaîne (racine, TLD, intermédiaires, feuille). |
| `dnsviz_zone_errors` | Chaque erreur rapportée par DNSViz, rattachée à la zone où elle a été trouvée. |
| `dnsviz_zone_warnings` | Chaque avertissement rapporté par DNSViz, rattaché à la zone où il a été trouvé. |
| `dnsviz_common_failures` | Confronte les constats à un catalogue d'échecs DNSSEC courants. |

Les statuts sont projetés ainsi : `SECURE` vers OK ; `BOGUS` vers Critique ; `INDETERMINATE` vers Avertissement ; `INSECURE`, `NON_EXISTENT` et un simple `NOERROR` (résout mais non signé) vers Info.

## Options

| Option | Signification | Défaut |
|---|---|---|
| Délai de sondage (`probeTimeoutSeconds`, admin) | Délai maximal pour l'appel à `dnsviz probe` ; le parcours récursif peut être lent sur certaines zones. | 120 |
| Nom de domaine (`domain_name`) | Domaine à analyser. | prérempli |

L'environnement d'exécution de DNSViz nécessite quant à lui la présence de `dnsviz` dans le `PATH` de l'hôte et un fichier d'ancre de confiance DNSKEY racine au format BIND (détecté automatiquement à partir du paquet système `dnssec-root` / `dns-root-data`, ou indiqué explicitement). L'image conteneur fournie embarque ces éléments, de sorte que l'ancre de confiance est en place d'emblée.

## Dans happyDomain

Activez ce vérificateur pour le domaine depuis la vue **Vérifications**. Voir {{< relref "/pages/checks" >}} pour la planification et la lecture des vérifications.

DNSViz et {{< relref "/reference/checkers/dnssec" >}} sont complémentaires : DNSViz valide la chaîne de bout en bout, tandis que le vérificateur DNSSEC couvre le durcissement NSEC/NSEC3. Un second avis sur la même chaîne est disponible auprès de {{< relref "/reference/checkers/zonemaster" >}}.
