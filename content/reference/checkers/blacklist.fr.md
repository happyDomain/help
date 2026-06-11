---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Liste noire (DNSBL)
description: "Vérifie si un domaine est actuellement listé sur des systèmes de réputation répandus : listes noires de domaines basées sur le DNS, flux de phishing/malware téléchargés et API de renseignement sur les menaces."
weight: 320
---

Le vérificateur **Liste noire et réputation** signale si un domaine est actuellement listé sur des systèmes de réputation répandus. Il interroge en parallèle une série de sources et produit un rapport HTML orienté diagnostic dont la section « Action requise » liste les inscriptions les plus impactantes avec une procédure de retrait propre à chaque opérateur.

**Portée** : niveau domaine. La vérification cible le nom de domaine lui-même, indépendamment de ses enregistrements DNS, et s'active depuis la vue des vérifications du domaine.

## Ce qu'il vérifie

Chaque source configurée contribue à sa propre règle ; une inscription sur l'une d'elles élève le statut du domaine. Les sources se répartissent en trois familles :

- **Listes noires de domaines basées sur le DNS (DBL/RHSBL)** : interrogées via DNS, sans clé d'API : Spamhaus DBL, SURBL multi, URIBL multi, NordSpam DBL, SpamEatingMonkey Fresh, Tiopan DBL, SORBS RHSBL, ainsi que toute zone DNSBL supplémentaire ajoutée par un administrateur.
- **Flux de phishing/malware téléchargés** : récupérés et mis en cache en mémoire : flux public OpenPhish, PhishTank, Botvrij.eu, Disconnect.me, OISD et une comparaison avec le DNS sécurisé Quad9.
- **Recherches HTTPS de renseignement sur les menaces** : nécessitant une clé d'API configurée par un administrateur : Google Safe Browsing, abuse.ch URLhaus / ThreatFox / MalwareBazaar, VirusTotal v3, AlienVault OTX, Pulsedive, Criminal IP.

Lorsqu'une requête DNSBL est refusée (de nombreux résolveurs publics sont bloqués par les opérateurs de DBL) ou qu'un quota d'API est épuisé, la source est rapportée comme avertissement afin de ne pas polluer le statut OK. Une source multi-fournisseurs comme VirusTotal rapporte « critique » lorsqu'au moins un fournisseur signale le domaine comme malveillant, et « avertissement » lorsqu'il est seulement suspect.

## Options

La seule option de niveau domaine est la cible elle-même :

| Option | Signification | Défaut |
|--------|---------------|--------|
| Nom de domaine (`domain_name`) | Le domaine à rechercher. Rempli automatiquement depuis le domaine. | (depuis le domaine) |

La sélection des sources et les identifiants se configurent source par source. La plupart des sources s'activent ou se désactivent (et leurs clés d'API se renseignent) au niveau **administrateur** ; un sous-ensemble des flux téléchargés est activé par défaut et peut être basculé par l'utilisateur. Voir la documentation de chaque source pour savoir lesquelles nécessitent une clé d'API et comment l'obtenir.

## Dans happyDomain

C'est un vérificateur de niveau domaine : activez-le depuis la vue des vérifications du domaine. Les inscriptions reflètent souvent une compromission ; traitez un résultat positif comme un signal pour auditer l'hôte et renouveler les identifiants, puis suivez les liens de retrait propres à chaque opérateur indiqués dans le rapport. Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
