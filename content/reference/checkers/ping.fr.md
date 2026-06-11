---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Ping
description: "Envoie des sondes ICMP à chaque adresse d'un service et rapporte l'accessibilité, le temps d'aller-retour moyen, la perte de paquets et la couverture IPv6."
weight: 190
---

Le vérificateur **Ping (ICMP)** mesure l'accessibilité réseau de base des adresses derrière un service. Il envoie un petit nombre de requêtes echo ICMP à chaque adresse `A`/`AAAA` et rapporte si la cible a répondu, son temps d'aller-retour moyen (RTT), le taux de perte de paquets observé, et si au moins une adresse IPv6 a répondu.

**Portée** : niveau service. Il s'attache aux services de type `abstract.Server` (un sous-domaine publiant des enregistrements `A`/`AAAA`) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

| Règle | Vérifie | Conditions avertissement / critique |
|-------|---------|-------------------------------------|
| `ping.reachable` | Chaque cible a répondu à au moins une sonde ICMP. | Critique lorsqu'une cible ne répond jamais. |
| `ping.rtt` | Le temps d'aller-retour moyen reste dans les seuils. | Avertissement au-dessus du RTT d'avertissement, critique au-dessus du RTT critique. |
| `ping.ipv6_reachable` | Au moins une cible IPv6 a répondu à une sonde ICMP. | Avertissement quand aucune adresse IPv6 ne répond. |
| `ping.packet_loss` | Le taux de perte de paquets reste dans les seuils. | Avertissement au-dessus du taux d'avertissement, critique au-dessus du taux critique. |

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Seuil d'avertissement RTT (ms) (`warningRTT`) | RTT moyen au-dessus duquel la vérification avertit. | 100 |
| Seuil critique RTT (ms) (`criticalRTT`) | RTT moyen au-dessus duquel la vérification est critique. | 500 |
| Seuil d'avertissement de perte de paquets (%) (`warningPacketLoss`) | Taux de perte au-dessus duquel la vérification avertit. | 10 |
| Seuil critique de perte de paquets (%) (`criticalPacketLoss`) | Taux de perte au-dessus duquel la vérification est critique. | 50 |
| Nombre de pings à envoyer (`count`) | Requêtes echo ICMP envoyées par cible. | 5 |

## Dans happyDomain

C'est un vérificateur de niveau service : configurez-le depuis l'onglet **Vérifications** du service « Serveur » sur le sous-domaine concerné. Son intervalle par défaut court le rend adapté à une surveillance d'accessibilité légère et fréquente. Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
