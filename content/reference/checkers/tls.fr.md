---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Posture TLS
description: "Se connecte à chaque point de terminaison TLS découvert pour un domaine, termine la poignée de main (avec bascule STARTTLS si nécessaire) et audite la chaîne de certificats, la couverture du nom, l'expiration et la robustesse du protocole."
weight: 170
---

Le vérificateur **TLS** (nom interne « TLS ») évalue la posture de sécurité du transport de chaque point de terminaison TLS exposé par un domaine. Il ne scanne pas de ports lui-même : il consomme des **entrées de découverte** de type `tls.endpoint.v1` publiées par d'autres vérificateurs de service (XMPP, SRV, CalDAV, CardDAV, SMTP, etc.). Pour chaque point de terminaison découvert, il effectue une véritable connexion TCP, une bascule STARTTLS spécifique au protocole le cas échéant, puis une poignée de main TLS complète, et il rapporte une posture par point de terminaison.

**Portée** : niveau domaine. Le vérificateur s'exécute sur l'ensemble du domaine et intègre les points de terminaison annoncés par les autres vérificateurs de service. Un point de terminaison n'est donc sondé que si un vérificateur de service (par exemple {{< relref "/reference/checkers/smtp" >}}) l'a publié.

## Ce qu'il vérifie

| Règle | Vérifie | Sévérité |
|-------|---------|----------|
| `tls.endpoints_discovered` | Au moins un point de terminaison TLS a été découvert pour cette cible. | Info |
| `tls.reachability` | Chaque point de terminaison découvert accepte une connexion TCP. | Critique |
| `tls.handshake` | La poignée de main TLS aboutit sur chaque point de terminaison joignable. | Critique |
| `tls.starttls_advertised` | Les points de terminaison STARTTLS annoncent la capacité de bascule. | Critique |
| `tls.starttls_dialect_supported` | Le dialecte STARTTLS découvert est implémenté par le vérificateur. | Critique |
| `tls.peer_certificate_present` | Le serveur a présenté un certificat pendant la poignée de main. | Critique |
| `tls.chain_validity` | La chaîne présentée se valide avec le magasin de confiance du système. | Critique |
| `tls.hostname_match` | Le certificat feuille couvre le nom sondé (SNI). | Critique |
| `tls.expiry` | Signale les certificats feuilles expirés ou proches de l'expiration. | Critique |
| `tls.version` | Signale les points de terminaison négociant une version TLS inférieure à TLS 1.2. | Avertissement |
| `tls.cipher_suite` | Rapporte la suite cryptographique négociée sur chaque point de terminaison. | Info |
| `tls.enum.versions` | Signale les points de terminaison acceptant encore des versions TLS inférieures à TLS 1.2 (option d'énumération). | Avertissement |
| `tls.enum.ciphers` | Signale les points de terminaison acceptant des suites cryptographiques cassées (NULL, anonyme, EXPORT, RC4, 3DES) (option d'énumération). | Avertissement |

Les bascules STARTTLS sont prises en charge pour SMTP/submission, IMAP, POP3 et XMPP (c2s et s2s). Lorsqu'un vérificateur de service marque un point de terminaison comme exigeant STARTTLS, l'absence de bascule est rapportée comme critique ; sinon, elle est considérée comme opportuniste et rapportée comme avertissement.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Délai de sonde par point de terminaison (ms) (`probeTimeoutMs`) | Temps maximal alloué pour la connexion, la bascule STARTTLS et la poignée de main TLS sur un point de terminaison. | 10000 |
| Énumérer les versions et suites TLS acceptées (`enumerateCiphers`) | Lorsqu'activée, chaque point de terminaison en TLS direct est balayé avec un ClientHello par paire (version, suite) afin de découvrir l'ensemble exact accepté par le serveur. Ajoute environ 50 poignées de main par point de terminaison. | false |

La liste des entrées de découverte est remplie automatiquement à partir de ce que les autres vérificateurs publient et n'est pas modifiable par l'utilisateur.

## Dans happyDomain

Le vérificateur TLS est une vérification de niveau domaine : activez-le depuis la vue des vérifications du domaine. Comme il travaille sur des points de terminaison découverts par d'autres vérificateurs, il s'associe naturellement aux vérificateurs de service qui publient des points de terminaison TLS, comme {{< relref "/reference/checkers/smtp" >}}.

{{% notice style="info" title="Posture TLS et DANE" %}}
Ce vérificateur valide le certificat en direct avec le magasin de confiance du système. L'épinglage du certificat dans le DNS via des enregistrements TLSA est un sujet distinct, traité par le vérificateur {{< relref "/reference/checkers/dane" >}}.
{{% /notice %}}

Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
