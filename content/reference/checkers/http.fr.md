---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: HTTP / HTTPS
description: "Sonde un serveur web en HTTP et HTTPS et évalue l'accessibilité, la chaîne de redirections, ainsi que l'ensemble des en-têtes de sécurité HTTP et des règles d'hygiène des cookies."
weight: 180
---

Le vérificateur **HTTP / HTTPS** sonde le serveur web déclaré par un service « Serveur » en HTTP simple (port 80) et en HTTPS (port 443), puis évalue une série de règles indépendantes sur les réponses : accessibilité, chaîne de redirections HTTP vers HTTPS et ensemble moderne des en-têtes de sécurité HTTP (HSTS, CSP, options de cadre, isolation inter-origines, etc.), ainsi que l'hygiène des cookies.

**Portée** : niveau service. Il s'attache aux services de type `abstract.Server` (un sous-domaine publiant des enregistrements `A`/`AAAA`) et se configure depuis l'onglet **Vérifications** de ce service.

L'analyse approfondie du TLS et des certificats est volontairement déléguée au vérificateur {{< relref "/reference/checkers/tls" >}} ; ce vérificateur ne s'appuie sur TLS que comme transport.

## Ce qu'il vérifie

| Règle | Vérifie | Sévérité |
|-------|---------|----------|
| `http.tcp_reachable` | Chaque IP sondée accepte une connexion HTTP sur le port 80. | Critique |
| `https.tcp_reachable` | Chaque IP sondée accepte une connexion HTTPS sur le port 443. | Critique |
| `http.https_redirect` | Le HTTP simple redirige vers une URL HTTPS sur le même hôte. | Avertissement |
| `http.redirect_chain` | La chaîne de redirections est sans boucle, sans longueur excessive ni rétrogradation de schéma. | Avertissement |
| `http.redirect_permanence` | La bascule HTTP vers HTTPS utilise 301 ou 308 (permanent) plutôt que 302/307. | Avertissement |
| `http.hsts` | Présence et qualité de l'en-tête Strict-Transport-Security en HTTPS. | Avertissement |
| `http.csp` | Présence et qualité de l'en-tête Content-Security-Policy en HTTPS. | Avertissement |
| `http.x_frame_options` | Les réponses définissent X-Frame-Options ou une directive CSP `frame-ancestors`. | Avertissement |
| `http.x_content_type_options` | Les réponses définissent `X-Content-Type-Options: nosniff`. | Avertissement |
| `http.x_xss_protection` | Rapporte la valeur de l'en-tête hérité X-XSS-Protection (CSP est le remplaçant adéquat). | Info |
| `http.referrer_policy` | Les réponses définissent un en-tête Referrer-Policy respectueux de la vie privée. | Avertissement |
| `http.permissions_policy` | L'en-tête Permissions-Policy restreint les API sensibles (caméra, microphone, géolocalisation, etc.). | Avertissement |
| `http.coop` | L'en-tête Cross-Origin-Opener-Policy est défini pour l'isolation de processus inter-origines. | Avertissement |
| `http.coep` | L'en-tête Cross-Origin-Embedder-Policy est défini (requis avec COOP pour l'isolation inter-origines). | Avertissement |
| `http.corp` | L'en-tête Cross-Origin-Resource-Policy restreint l'inclusion inter-origines. | Avertissement |
| `http.cookie_flags` | Les cookies posés en HTTPS utilisent les attributs Secure, HttpOnly et SameSite. | Avertissement |
| `http.cookie_prefixes` | Les cookies utilisant les préfixes `__Secure-` / `__Host-` respectent les contraintes RFC 6265bis. | Avertissement |
| `http.cookie_size` | Signale les lignes Set-Cookie dépassant les 4096 octets que les navigateurs doivent supporter au minimum. | Avertissement |
| `http.sri` | Rapporte les balises script/style inter-origines dépourvues d'attributs Subresource Integrity. | Avertissement |
| `http.security_txt` | Indique si `/.well-known/security.txt` (RFC 9116) est publié. | Avertissement |

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Délai par requête (ms) (`probeTimeoutMs`) | Temps maximal alloué à une requête HTTP/HTTPS. | 10000 |
| Nombre maximal de redirections (`maxRedirects`) | Arrête de suivre les redirections au-delà de ce nombre de sauts. | 5 |
| User-Agent (`userAgent`) | En-tête User-Agent envoyé à chaque requête. | `happyDomain-checker-http/1.0` |
| Exiger HTTPS (`requireHTTPS`) | Le HTTP simple doit rediriger vers HTTPS. | true |
| Exiger HSTS (`requireHSTS`) | Les réponses HTTPS doivent inclure un en-tête Strict-Transport-Security. | true |
| max-age HSTS minimal (jours) (`minHSTSMaxAgeDays`) | Valeur max-age HSTS minimale acceptable, en jours. | 180 |
| Exiger Content-Security-Policy (`requireCSP`) | Les réponses HTTPS doivent inclure un en-tête Content-Security-Policy. | false |

## Dans happyDomain

C'est un vérificateur de niveau service : configurez-le depuis l'onglet **Vérifications** du service « Serveur » sur le sous-domaine concerné. Pour la posture détaillée des certificats, ajoutez aussi le vérificateur {{< relref "/reference/checkers/tls" >}}. Pour le fonctionnement général de la configuration et de la lecture des vérifications, voir {{< relref "/pages/checks" >}}.
