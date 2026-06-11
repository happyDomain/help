---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SSH
description: "Se connecte aux ports SSH annoncés d'un serveur et audite l'accessibilité, les correspondances bannière/CVE, la posture complète des algorithmes, les clés d'hôte observées, l'alignement SSHFP et les méthodes d'authentification."
weight: 280
---

Le vérificateur **SSH** produit un audit de sécurité complet du service SSH exposé par un service « Serveur ». Il se connecte aux ports SSH annoncés sur chaque adresse `A`/`AAAA` et rapporte l'accessibilité, les correspondances bannière/CVE, la posture complète des algorithmes (échange de clés, clé d'hôte, chiffrement, MAC, compression), les clés d'hôte observées, l'alignement des empreintes SSHFP et les méthodes d'authentification exposées par le serveur. Les résultats sont présentés dans un rapport HTML orienté correction rapide.

**Portée** : niveau service. Il s'attache aux services de type `abstract.Server` (un sous-domaine publiant des enregistrements `A`/`AAAA` et éventuellement `SSHFP`) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qu'il vérifie

| Règle | Vérifie | Sévérité |
|-------|---------|----------|
| `ssh.tcp_reachable` | Chaque paire (adresse, port) sondée accepte une connexion TCP. | Critique |
| `ssh.handshake` | L'échange de bannière SSH et l'analyse KEXINIT aboutissent sur chaque point de terminaison joignable. | Critique |
| `ssh.protocol_version` | Chaque point de terminaison annonce SSH-2 et rejette l'ancien SSH-1. | Critique |
| `ssh.banner_software` | Signale les serveurs dont la bannière n'est pas une version OpenSSH reconnue. | Info |
| `ssh.known_vulnerabilities` | Confronte la version OpenSSH annoncée à un catalogue de CVE (regreSSHion, Terrapin, etc.). | Critique |
| `ssh.host_key_strength` | Signale les clés d'hôte sous la taille minimale acceptée (par exemple RSA < 2048 bits). | Critique |
| `ssh.kex_algorithms` | Signale les algorithmes d'échange de clés faibles ou cassés. | Critique |
| `ssh.host_key_algorithms` | Signale les algorithmes de clé d'hôte faibles ou obsolètes (ssh-rsa/SHA-1, ssh-dss, etc.). | Critique |
| `ssh.cipher_algorithms` | Signale les chiffrements symétriques faibles ou cassés (CBC, 3DES, RC4, etc.). | Critique |
| `ssh.mac_algorithms` | Signale les algorithmes MAC faibles (SHA-1, non-ETM, etc.). | Critique |
| `ssh.strict_kex` | Le serveur annonce le marqueur strict-KEX (mitigation Terrapin, CVE-2023-48795). | Avertissement |
| `ssh.preauth_compression` | Signale les serveurs offrant la compression zlib avant authentification. | Info |
| `ssh.auth_methods` | Examine les méthodes d'authentification annoncées (exposition par mot de passe, disponibilité de la clé publique). | Avertissement |
| `ssh.sshfp_alignment` | Compare les enregistrements SSHFP publiés aux clés d'hôte observées (concordance, manquant, divergence). | Critique |
| `ssh.sshfp_hash` | Signale les jeux SSHFP ne publiant que des empreintes SHA-1 (type 1). | Avertissement |

La correspondance CVE couvre notamment regreSSHion (CVE-2024-6387), la RCE PKCS#11 de ssh-agent (CVE-2023-38408), Terrapin (CVE-2023-48795) et plusieurs anciennes failles d'énumération d'utilisateurs et d'injection de commandes.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Ports (`ports`) | Ports TCP supplémentaires à sonder, séparés par des virgules. Le port 22 est toujours sondé. | (vide) |
| Délai de sonde par point de terminaison (ms) (`probeTimeoutMs`) | Temps maximal pour la connexion, la bannière, le KEXINIT et la poignée de main sur un point de terminaison. | 10000 |
| Énumérer les méthodes d'authentification (`includeAuthProbe`) | Ouvre une seconde connexion avec un utilisateur factice pour découvrir les méthodes d'authentification annoncées. | true |

## Dans happyDomain

C'est un vérificateur de niveau service : configurez-le depuis l'onglet **Vérifications** du service « Serveur » sur le sous-domaine concerné. Ses règles SSHFP recoupent les enregistrements `SSHFP` publiés dans votre zone ; garder ces enregistrements synchronisés avec les clés d'hôte du serveur améliore le résultat. Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
