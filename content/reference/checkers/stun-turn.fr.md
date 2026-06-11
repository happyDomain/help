---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: STUN / TURN
description: "Sonde de bout en bout les serveurs STUN et TURN : découverte, accessibilité, TLS/DTLS, binding STUN et relais TURN authentifié."
weight: 310
---

Le vérificateur **STUN / TURN** sonde de bout en bout les serveurs STUN et TURN. STUN et TURN sont les serveurs de traversée de NAT sur lesquels reposent les applications temps réel (WebRTC, voix et vidéo) pour établir un flux média pair à pair : STUN permet à un hôte de découvrir son adresse réflexive publique, tandis que TURN relaie le média lorsqu'aucun chemin direct ne peut être ouvert.

Il s'agit d'un vérificateur de **niveau service**. Il effectue la découverte SRV (ou utilise une URI explicite), contrôle l'accessibilité TCP/UDP et la poignée de main TLS/DTLS, émet une requête de binding STUN, vérifie que le serveur TURN exige une authentification, réalise un `Allocate` TURN authentifié, puis éprouve le chemin de relais par un aller-retour `CreatePermission + Send`.

## Ce qui est vérifié

| Règle | Ce qu'elle vérifie | Sévérité |
|-------|--------------------|----------|
| `stun_turn.discovery` | Au moins un point d'accès STUN/TURN a pu être découvert (URI explicite ou résolution SRV). | Critique |
| `stun_turn.srv_stun` | Au moins un point d'accès STUN est disponible via SRV (`_stun` / `_stuns`) ou URI explicite. | Avertissement |
| `stun_turn.srv_turn` | Au moins un point d'accès TURN est disponible via SRV (`_turn` / `_turns`) ou URI explicite. | Critique |
| `stun_turn.dial` | Chaque point d'accès découvert accepte une connexion (poignée de main TCP/TLS ou socket UDP). | Critique |
| `stun_turn.tls_transport` | Au moins un transport TLS/DTLS (`stuns` / `turns`) aboutit lorsqu'il est présent. | Critique |
| `stun_turn.ipv6_coverage` | Au moins un nom d'hôte STUN/TURN se résout vers une adresse IPv6. | Avertissement |
| `stun_turn.stun_binding` | La requête de binding STUN reçoit une réponse XOR-MAPPED-ADDRESS. | Critique |
| `stun_turn.reflexive_public` | Signale les points d'accès renvoyant une adresse réflexive privée ou de bouclage (serveur ignorant son IP publique). | Critique |
| `stun_turn.stun_latency` | Compare le temps d'aller-retour du binding STUN aux seuils d'avertissement et critique. | Critique |
| `stun_turn.turn_open_relay` | Le serveur TURN exige une authentification (répond à un `Allocate` non authentifié par un 401). | Critique |
| `stun_turn.turn_auth` | Les identifiants TURN fournis (ou le secret partagé REST) permettent un `Allocate` réussi. | Critique |
| `stun_turn.relay_public` | Signale les serveurs TURN dont l'adresse de relais allouée est privée ou de bouclage (IP de relais publique manquante). | Critique |
| `stun_turn.relay_echo` | Le chemin de relais TURN peut acheminer du trafic vers le pair de test configuré (`CreatePermission + Send`). | Avertissement |

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Zone | Zone utilisée pour la découverte SRV (`_stun._udp` / `_turn._udp` / `_turns._tcp`) en l'absence d'URI explicite. Renseignée automatiquement. | (renseignée automatiquement) |
| URI du serveur | URI STUN/TURN explicite (RFC 7064/7065). Prend le pas sur la découverte SRV. | (aucun) |
| Mode | `auto` sonde à la fois STUN et TURN ; `stun` ignore les tests d'allocation TURN ; `turn` exige l'allocation TURN. | `auto` |
| Nom d'utilisateur TURN | Nom d'utilisateur pour les identifiants TURN à long terme. | (aucun) |
| Mot de passe TURN | Mot de passe pour les identifiants TURN à long terme (secret). | (aucun) |
| Secret partagé API REST | Secret partagé pour dériver des identifiants éphémères (draft-uberti-rtcweb-turn-rest) ; prend le pas sur le nom d'utilisateur et le mot de passe (secret). | (aucun) |
| Realm | Realm TURN explicite facultatif. | (aucun) |
| Transports | Liste de transports à tester, séparés par des virgules, parmi `udp`, `tcp`, `tls`, `dtls`. | `udp,tcp,tls` |
| Cible de l'écho de relais | `hôte:port` utilisé pour valider le chemin de relais ; un `CreatePermission + Send` est émis, aucune donnée utile n'est échangée. | `1.1.1.1:53` |
| Tester aussi ChannelBind | Éprouve en plus ChannelBind sur la connexion de relais. | `false` |
| Seuil d'avertissement de RTT (ms) | Temps d'aller-retour du binding STUN au-delà duquel un avertissement est déclenché. | 200 |
| Seuil critique de RTT (ms) | Temps d'aller-retour du binding STUN au-delà duquel une alerte critique est déclenchée. | 1000 |
| Délai par sonde (s) | Budget de temps alloué à chaque sonde individuelle. | 5 |

{{% notice style="info" title="Des identifiants sont nécessaires pour les tests TURN" %}}
Les règles d'authentification, de relais public et d'écho de relais ne s'exécutent que lorsque des identifiants TURN valides sont fournis : soit un couple nom d'utilisateur/mot de passe, soit un secret partagé d'API REST. Sans eux, le vérificateur valide tout de même la découverte, l'accessibilité, le TLS et le binding STUN, mais ne peut pas éprouver le chemin de relais TURN.
{{% /notice %}}

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** du service concerné ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. La zone est renseignée automatiquement ; fournissez une URI de serveur et des identifiants TURN selon les besoins de votre déploiement.
