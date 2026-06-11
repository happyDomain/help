---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Autoconfiguration de la messagerie
description: "Vérifie qu'un domaine publie une configuration de messagerie découvrable via l'autoconfig Thunderbird, l'Autodiscover Microsoft et les enregistrements SRV de la RFC 6186."
weight: 220
---

Le vérificateur d'**autoconfiguration de la messagerie** s'assure que les clients de messagerie peuvent découvrir automatiquement comment se connecter à vos serveurs de courriel. Lorsque l'autoconfiguration est publiée correctement, l'utilisateur saisit seulement son adresse et son mot de passe, et son client (Thunderbird, Outlook, applications mobiles) trouve de lui-même les bons hôtes IMAP/POP et SMTP, les ports et les réglages de sécurité.

Ce vérificateur s'applique au **niveau service** : il concerne les services d'autoconfiguration de messagerie et de RFC 6186 d'un domaine. À chaque exécution, il sonde les mécanismes de découverte utilisés par les clients réels :

- **Autoconfig Thunderbird** (brouillon Bucksch) : `https://autoconfig.<domaine>/mail/config-v1.1.xml` comme URL primaire, avec le repli sur l'apex `https://<domaine>/.well-known/autoconfig/...`, une variante optionnelle en HTTP simple, le repli sur l'ISPDB de Mozilla et les replis via le parent MX.
- **Autodiscover Microsoft** (POX) : `https://autodiscover.<domaine>/autodiscover/autodiscover.xml`.
- **Enregistrements SRV de la RFC 6186** : `_imaps`, `_imap`, `_pop3s`, `_pop3`, `_submissions`, `_submission`, `_autodiscover`.
- **Résolution MX**, pour le contexte et la découverte fondée sur les MX.

Il analyse chaque réponse, recoupe les serveurs annoncés par les différentes sources et produit un rapport HTML accompagné d'extraits de correction prêts à coller pour les défauts les plus courants.

## Ce qui est vérifié

| Règle | Ce qui est vérifié | Sévérité en cas d'échec |
|---|---|---|
| `autoconfig_presence` | Au moins une méthode de découverte d'autoconfiguration répond pour le domaine. | Critique |
| `autoconfig_preferred_endpoint` | L'URL primaire `https://autoconfig.<domaine>/mail/config-v1.1.xml` est joignable et sert un clientConfig valide. | Avertissement |
| `autoconfig_tls` | Les points d'accès d'autoconfig sont servis en HTTPS avec un certificat TLS valide. | Critique |
| `autoconfig_server_encryption` | Les serveurs annoncés par l'autoconfig utilisent SSL ou STARTTLS et une méthode d'authentification non en clair. | Critique |
| `autoconfig_consistency` | Les noms d'hôtes et ports rapportés par l'autoconfig, l'Autodiscover et les SRV concordent entre eux. | Avertissement |
| `autoconfig_srv_records` | Les enregistrements SRV de la RFC 6186 complètent le XML d'autoconfig. | Avertissement |
| `autoconfig_autodiscover` | Si l'Autodiscover Microsoft (POX) répond sur le domaine. | Avertissement |

En cas d'échec, la section « Fix this first » du rapport fournit des extraits prêts à copier : un exemple de `config-v1.1.xml` et ses URL canoniques lorsque rien n'est publié, une incitation à ajouter le sous-domaine `autoconfig.` lorsque seul `.well-known` répond, une redirection vers HTTPS, un rappel sur la couverture du certificat, un aide-mémoire de ports pour les serveurs en clair (SSL 993/465, STARTTLS 143/587) et un extrait de zone SRV prêt à coller.

## Options

### Par utilisateur

| Option | Signification | Défaut |
|---|---|---|
| Partie locale des sondes (`probeEmail`) | Partie locale envoyée dans la chaîne de requête de l'URL d'autoconfig (avant le `@`). La plupart des serveurs l'ignorent. | `test` |
| Délai HTTP (`httpTimeout`) | Délai par requête, en secondes, lors du sondage des points d'accès. | 8 |
| Tenter le repli ISPDB de Mozilla (`tryISPDB`) | Lorsque le domaine ne publie aucun fichier d'autoconfig, interroger aussi l'ISPDB public de Mozilla. | true |
| Autoriser le repli HTTP simple (`tryHTTPAutoconfig`) | Sonder aussi la variante en HTTP simple de `autoconfig.<domaine>` (optionnelle dans le brouillon) ; utile pour repérer les fournisseurs encore en HTTP. | false |
| Sonder l'Autodiscover Microsoft (`tryAutodiscoverPost`) | Sonder les points d'accès Autodiscover Exchange/Outlook. À désactiver pour ne vérifier que le flux Thunderbird. | true |

### Admin

| Option | Signification | Défaut |
|---|---|---|
| URL de base de l'ISPDB Mozilla (`ispdbURL`) | URL de base de la base de repli d'autoconfig de Mozilla. | `https://autoconfig.thunderbird.net/v1.1/` |
| User-Agent des sondes (`userAgent`) | User-Agent annoncé dans chaque requête de sondage. | `happyDomain-autoconfig/1.0 (+https://happydomain.org)` |

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** du service d'autoconfiguration de messagerie concerné. Voir {{< relref "/pages/checks" >}} pour la planification et la lecture des vérifications.

Ce vérificateur est le compagnon naturel d'une configuration de messagerie complète : voir {{< relref "/reference/services/email" >}} pour les services MX, SPF, DKIM et DMARC qui régissent la remise et l'authentification du courrier.
