---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Zonemaster
description: "Lance la suite de tests Zonemaster sur un domaine via son API JSON-RPC et regroupe les résultats par catégorie de test (délégation, cohérence, DNSSEC…)."
weight: 350
---

Le vérificateur **Zonemaster** lance la suite de tests [Zonemaster](https://zonemaster.net/) sur un domaine et rapporte ses constats regroupés par catégorie de test. Zonemaster est un validateur reconnu de la santé DNS et de la délégation ; ce vérificateur le pilote via son API JSON-RPC, conserve l'ensemble des résultats sous forme d'observation et produit un rapport HTML regroupé par module et par sévérité.

Ce vérificateur s'applique au **niveau domaine** : il évalue la délégation et le contenu de zone du domaine plutôt qu'un service isolé.

## Fonctionnement

À chaque exécution, le vérificateur soumet le domaine à un point d'accès JSON-RPC Zonemaster (l'API publique officielle par défaut), attend la fin du lot de tests et conserve chaque message renvoyé par Zonemaster. Chaque message porte un module, un cas de test et une sévérité Zonemaster (INFO / NOTICE / WARNING / ERROR / CRITICAL).

{{% notice style="info" title="Ne pointez que vers une instance Zonemaster de confiance" %}}
L'URL de l'API Zonemaster est une option d'administrateur. Comme le vérificateur émet des requêtes vers l'URL configurée et en restitue les réponses, ne le pointez que vers une instance Zonemaster en laquelle vous avez confiance.
{{% /notice %}}

## Ce qui est vérifié

Le vérificateur projette chaque module de test Zonemaster sur une règle de catégorie. Chaque règle émet un état `<règle>.summary`, plus un état par message de niveau WARNING ou pire (afin que les consommateurs en aval puissent s'appuyer sur des codes stables) ; les messages INFO et NOTICE sont agrégés dans les compteurs du résumé.

| Règle | Ce qu'elle couvre |
|---|---|
| `zonemaster.dnssec` | Tests DNSSEC (signatures, NSEC/NSEC3, cohérence DS/DNSKEY). |
| `zonemaster.delegation` | Tests de délégation (accord NS parent/enfant, glue, renvois). |
| `zonemaster.consistency` | Tests de cohérence (numéro de série SOA, ensemble NS, contenu de zone entre serveurs). |
| `zonemaster.connectivity` | Tests de connectivité (joignabilité UDP/TCP des serveurs autoritaires, diversité d'AS). |
| `zonemaster.nameserver` | Tests des serveurs de noms (comportement du serveur, EDNS, traitement des RR inconnus). |
| `zonemaster.syntax` | Tests de syntaxe (syntaxe des noms de domaine, légalité des noms d'hôtes). |
| `zonemaster.zone` | Tests de zone (valeurs SOA, présence de MX, enregistrements obligatoires). |
| `zonemaster.address` | Tests d'adresses (adresses IP des serveurs de noms, plages privées/réservées). |
| `zonemaster.basic` | Tests de base/système (joignabilité initiale et prérequis fondamentaux). |

Les sévérités Zonemaster sont projetées sur les statuts de happyDomain : CRITICAL et ERROR vers Critique ; WARNING vers Avertissement ; NOTICE, INFO et DEBUG vers Info. Chaque résumé retient le pire statut parmi les messages de sa catégorie. Lorsque Zonemaster ne renvoie aucun message pour une catégorie, la règle correspondante rapporte Inconnu (non testée).

## Options

| Option | Signification | Défaut |
|---|---|---|
| Nom de domaine à vérifier (`domainName`) | Domaine soumis à Zonemaster. **Obligatoire.** | prérempli |
| Profil (`profile`) | Nom du profil Zonemaster sous lequel exécuter les tests. | `default` |
| Langue des résultats (`language`, par utilisateur) | Langue des messages renvoyés (`en`, `fr`, `de`, `es`, `sv`, `da`, `fi`, `nb`, `nl`, `pt`). | `en` |
| URL de l'API Zonemaster (`zonemasterAPIURL`, admin) | Point d'accès JSON-RPC à interroger. | `https://zonemaster.net/api` |

## Dans happyDomain

Activez ce vérificateur pour le domaine depuis la vue **Vérifications**. Voir {{< relref "/pages/checks" >}} pour la planification et la lecture des vérifications.

Zonemaster recoupe en partie {{< relref "/reference/checkers/dnsviz" >}} sur le DNSSEC, mais couvre un éventail plus large de tests de délégation, de connectivité et de contenu de zone. Pour le durcissement NSEC/NSEC3 en particulier, voir {{< relref "/reference/checkers/dnssec" >}}.
