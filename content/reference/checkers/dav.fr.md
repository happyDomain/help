---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: CalDAV / CardDAV
description: "Découvre et sonde les serveurs CalDAV et CardDAV d'un domaine : découverte via well-known/SRV, accessibilité HTTPS, capacités DAV annoncées et (avec des identifiants) le parcours authentifié principal, home-set, collections et REPORT."
weight: 240
---

Les vérificateurs **CalDAV** et **CardDAV** vérifient que les serveurs de calendrier (RFC 4791) et de contacts (RFC 6352) d'un domaine sont découvrables, accessibles, et annoncent correctement les extensions WebDAV sur lesquelles s'appuient les clients. Il s'agit de deux vérificateurs distincts partageant la même logique : `caldav` s'attache à un service *CalDAV* (`abstract.CalDAV`), `carddav` à un service *CardDAV* (`abstract.CardDAV`). La seule différence de comportement porte sur la capacité DAV requise (`calendar-access` contre `addressbook`) et sur une vérification de planification propre à CalDAV.

Les deux vérificateurs sont de **niveau service** : ils ciblent le service correspondant publié sur un sous-domaine et se configurent depuis l'onglet **Vérifications** de ce service. La découverte suit la RFC 6764 : le vérificateur résout une URL de contexte à partir de `/.well-known/{caldav,carddav}` et des enregistrements SRV `_caldavs._tcp` / `_carddavs._tcp` (avec un indice TXT `path=` facultatif), puis sonde ce point d'accès.

Lorsqu'aucun identifiant n'est fourni, seule la phase anonyme s'exécute (découverte, transport, OPTIONS). Fournir un nom d'utilisateur et un mot de passe déverrouille la phase authentifiée : découverte du principal, home-set, énumération des collections et sonde REPORT.

{{% notice style="info" title="La posture TLS est hors périmètre" %}}
Ces vérificateurs confirment uniquement qu'une session HTTPS peut être établie vers l'URL de contexte. La validation du certificat (chaîne, nom d'hôte, expiration, algorithmes) est volontairement laissée au vérificateur TLS dédié. Consultez {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qui est vérifié | Conditions |
|---|---|---|
| `dav_discovery` | Une URL de contexte se résout via `/.well-known` ou SRV. | **Critique** si aucune URL de contexte ne peut être résolue. **Avertissement** quand `/.well-known` renvoie `200` au lieu d'une redirection `301`/`302` (légal mais beaucoup de clients ne la suivent pas). **Info** quand l'URL a été résolue via SRV mais que `/.well-known` est cassé. |
| `dav_transport` | La connexion HTTPS vers l'URL de contexte est accessible. | **Critique** si la connexion échoue. |
| `dav_options` | Une requête HTTP `OPTIONS` annonce la capacité DAV requise (`calendar-access` pour CalDAV, `addressbook` pour CardDAV) et les méthodes `PROPFIND`/`REPORT`. | **Critique** si OPTIONS échoue ou si la capacité manque. **Avertissement** si l'en-tête `Allow` ne contient pas `PROPFIND` ou `REPORT`. |
| `dav_principal` | L'URL du principal de l'utilisateur courant est découverte via un `PROPFIND` authentifié. | **Critique** en cas d'échec. **Inconnu** si aucun identifiant n'est fourni (phase ignorée). |
| `dav_home_set` | Le home-set calendrier/carnet d'adresses est découvert depuis le principal. | **Critique** en cas d'échec. **Inconnu** si ignoré. |
| `dav_collections` | Les collections calendrier/carnet d'adresses s'énumèrent et exposent leurs propriétés (jeu de composants pris en charge, données d'adresse prises en charge, nom affiché...). | **Critique** en cas d'échec. **Avertissement** si le home-set est vide. **Inconnu** si ignoré. |
| `dav_report` | Le serveur accepte une requête `REPORT` minimale (`calendar-query` / `addressbook-query`) sur la première collection. | **Critique** en cas d'échec. **Avertissement** si la requête renvoie une réponse inattendue. **Inconnu** si ignoré. |
| `caldav_scheduling` | *(CalDAV uniquement)* Lorsque `calendar-schedule` est annoncé, le principal expose `schedule-inbox-URL` et `schedule-outbox-URL`. | **Avertissement** si annoncé mais que les URL manquent ou que la sonde échoue. **Info** quand la planification n'est pas annoncée. **Inconnu** si ignoré. |

Le rapport HTML met en avant les erreurs de configuration les plus courantes sous forme d'encarts : `/.well-known` renvoyant `200`, absence de SRV et de well-known (service inaccessible), un enregistrement SRV en clair sans équivalent sécurisé, un serveur n'annonçant pas la classe DAV requise, et la phase authentifiée ignorée faute d'identifiants.

## Options

Les deux vérificateurs acceptent les mêmes options.

| Option | Signification | Défaut |
|---|---|---|
| Nom d'utilisateur | Facultatif. Fournir des identifiants déverrouille les vérifications authentifiées (principal, home-set, collections, sonde REPORT). | *(vide)* |
| Mot de passe ou jeton | Facultatif. Associé au nom d'utilisateur pour l'authentification HTTP Basic. | *(vide)* |
| URL de contexte explicite | Facultatif. Contourne la découverte `/.well-known` et SRV ; à utiliser pour les serveurs à la disposition non standard. | *(vide)* |
| Nom de domaine | Le domaine à sonder (renseigné automatiquement depuis la portée du service). | *(auto)* |
| Délai d'attente (secondes) | Délai d'attente HTTP par requête. | `10` |

## Dans happyDomain

Activez le vérificateur CalDAV ou CardDAV depuis l'onglet **Vérifications** d'un service CalDAV ou CardDAV. Le nom de domaine est renseigné automatiquement ; ajoutez des identifiants uniquement si vous souhaitez exécuter les vérifications authentifiées des collections et de REPORT. Consultez {{< relref "/pages/checks" >}} pour le fonctionnement complet, et {{< relref "/reference/checkers/tls" >}} pour la posture des certificats de ces mêmes points d'accès.
