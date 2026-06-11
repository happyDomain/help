---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SMTP
description: "Sonde chaque cible MX d'un domaine sur le port 25 comme le ferait un opérateur avec swaks : connexion TCP, bannière, EHLO, STARTTLS, transactions de messagerie et sonde de relais ouvert, DNS inverse et couverture IPv6."
weight: 200
---

Le vérificateur **SMTP entrant (posture MX)** examine le côté *entrant* du service de messagerie d'un domaine. Pour chaque cible MX de la zone, il effectue les sondes en direct qu'un opérateur exécuterait avec `swaks` ou `telnet … 25` : connexion TCP, bannière ESMTP et EHLO, négociation STARTTLS, sondes de transaction (expéditeur nul, postmaster, relais ouvert), DNS inverse / FCrDNS, inventaire des extensions et couverture IPv4/IPv6. Le résultat est un rapport HTML exploitable.

**Portée** : niveau service. Il s'attache aux services de type `svcs.MXs` (l'ensemble des enregistrements MX) et se configure depuis l'onglet **Vérifications** de ce service.

La sonde répond à la question « ce domaine peut-il *recevoir* le courrier correctement ? ». Elle ne teste **pas** la délivrabilité sortante (alignement SPF/DKIM/DMARC, score anti-spam, statut de liste noire), ce qui est le rôle du vérificateur {{< relref "/reference/checkers/happydeliver" >}}. Les sondes de transaction s'arrêtent toujours à `RCPT` et émettent `RSET` : aucun `DATA` n'est envoyé, donc aucun message n'est délivré.

## Ce qu'il vérifie

| Règle | Vérifie | Sévérité |
|-------|---------|----------|
| `smtp.null_mx` | Indique si le domaine publie un MX nul (RFC 7505). | Info |
| `smtp.mx_present` | Le domaine publie au moins un enregistrement MX (ou un MX nul). | Critique |
| `smtp.mx_sanity` | Signale les cibles MX violant la RFC 5321 § 5.1 (littéraux IP, chaînes CNAME, noms non résolus). | Critique |
| `smtp.endpoint_reachable` | Chaque point de terminaison MX accepte une connexion TCP sur le port 25. | Critique |
| `smtp.banner_sanity` | Chaque point de terminaison joignable émet une salutation SMTP 220. | Critique |
| `smtp.ehlo_supported` | Chaque point de terminaison accepte EHLO. | Critique |
| `smtp.starttls_offered` | Chaque point de terminaison annonce l'extension STARTTLS. | Critique |
| `smtp.starttls_handshake` | La poignée de main STARTTLS aboutit là où elle est annoncée. | Critique |
| `smtp.auth_posture` | Signale les points de terminaison annonçant SMTP AUTH avant STARTTLS (identifiants en clair). | Critique |
| `smtp.reverse_dns` | Chaque point de terminaison a un enregistrement PTR concordant (FCrDNS). | Avertissement |
| `smtp.null_sender` | Les points de terminaison acceptent l'expéditeur nul `MAIL FROM:<>` (requis pour les DSN). | Critique |
| `smtp.postmaster` | Les points de terminaison acceptent `RCPT TO:<postmaster@domaine>` (RFC 5321 § 4.5.1). | Critique |
| `smtp.open_relay` | Signale les points de terminaison relayant le courrier pour des destinataires hors du domaine testé. | Critique |
| `smtp.extension_posture` | Rapporte la posture des extensions ESMTP (PIPELINING, 8BITMIME). | Info |
| `smtp.ipv6_reachable` | Au moins un point de terminaison MX est joignable en IPv6. | Info |
| `smtp.tls_quality` | Intègre les constats TLS en aval (chaîne, nom, expiration) au rapport SMTP. | Critique |

La posture des certificats elle-même est hors de portée ici : chaque cible MX est publiée comme entrée de découverte `tls.endpoint.v1` (STARTTLS opportuniste), et le vérificateur {{< relref "/reference/checkers/tls" >}} effectue l'analyse des certificats sur la même connexion. Ses constats sont réintégrés dans la règle `smtp.tls_quality` et dans le rapport HTML.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Domaine (`domain`) | Domaine à tester. Rempli automatiquement depuis le service. | (depuis le service) |
| Délai par point de terminaison (secondes) (`timeout`) | Délai de connexion par point de terminaison. | 12 |
| Nom EHLO (`helo_name`) | Nom annoncé dans EHLO/HELO. Utilisez un nom qui résout et possède un PTR valide. | `mx-checker.happydomain.org` |
| Sonder l'expéditeur nul (`test_null_sender`) | Sonde `MAIL FROM:<>` (acceptation des DSN, RFC 5321). | true |
| Sonder le postmaster (`test_postmaster`) | Sonde `RCPT TO:<postmaster@domaine>` (RFC 5321 § 4.5.1). | true |
| Sonder la posture de relais ouvert (`test_open_relay`) | Sonde un destinataire hors du domaine testé pour détecter les relais ouverts. | true |
| Destinataire de la sonde de relais ouvert (`test_probe_address`) | Boîte (hors du domaine testé) utilisée pour la sonde de relais ouvert. | `postmaster@example.com` |

## Dans happyDomain

C'est un vérificateur de niveau service : configurez-le depuis l'onglet **Vérifications** du service « Serveurs e-mail » (MX). Pour confirmer que le courrier que votre domaine *envoie* arrive en boîte de réception, associez-le au vérificateur {{< relref "/reference/checkers/happydeliver" >}}. Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
