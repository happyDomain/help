---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: LDAP
description: "Sonde l'annuaire LDAP d'un domaine de bout en bout : découverte SRV, chiffrement du transport (StartTLS / LDAPS), introspection RootDSE, exposition anonyme, refus du bind en clair et bind authentifié facultatif."
weight: 250
---

Le vérificateur **LDAP** sonde le déploiement LDAP d'un domaine de bout en bout. À partir de la découverte SRV (`_ldap._tcp`, `_ldaps._tcp`), il teste l'accessibilité de chaque point d'accès, confirme qu'un canal chiffré est disponible (StartTLS selon la RFC 2830 ou TLS implicite sur le port 636), introspecte le RootDSE, recherche une divulgation d'information anonyme, vérifie que l'annuaire refuse les binds en clair et, lorsque des identifiants sont fournis, effectue un bind authentifié sur TLS avec un test de lecture facultatif sur un DN de base.

Il s'agit d'un vérificateur de **niveau service** : il cible un service *LDAP* (`abstract.LDAP`) publié sur un sous-domaine et se configure depuis l'onglet **Vérifications** de ce service. Pour chaque transport, il sonde `_ldap._tcp` (repli sur le port 389) et `_ldaps._tcp` (repli sur le port 636), en testant chaque adresse A/AAAA résolue par famille d'adresses.

{{% notice style="info" title="La posture TLS est intégrée, pas dupliquée" %}}
Le vérificateur LDAP confirme uniquement qu'une session TLS peut être établie, en enregistrant la version et l'algorithme négociés à titre de contexte. Chaque point d'accès sondé est publié comme entrée de découverte `tls.endpoint.v1` afin que le vérificateur TLS dédié puisse vérifier la chaîne de certificats, la correspondance du nom d'hôte et l'expiration. Ces constats sont réintégrés sur la page du service LDAP via la règle `ldap.tls_quality` : un certificat défectueux sur un point d'accès LDAP apparaît ici, et pas seulement dans une vue TLS distincte. Consultez {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qui est vérifié | Gravité |
|---|---|---|
| `ldap.has_srv` | Les enregistrements SRV `_ldap._tcp` / `_ldaps._tcp` sont publiés et résolvables. | Avertissement |
| `ldap.endpoint_reachable` | Chaque point d'accès LDAP découvert accepte une connexion TCP. | Critique |
| `ldap.has_encrypted_transport` | Au moins un point d'accès accessible propose un canal chiffré (LDAPS ou StartTLS). | Critique |
| `ldap.starttls_supported` | StartTLS est proposé et la bascule réussit sur chaque point d'accès LDAP en clair accessible. | Critique |
| `ldap.ldaps_handshake` | La poignée de main TLS directe réussit sur chaque point d'accès LDAPS. | Critique |
| `ldap.starttls_on_ldaps` | Signale les serveurs qui annoncent inutilement StartTLS sur le port LDAPS à TLS implicite. | Info |
| `ldap.ipv6_reachable` | Au moins un point d'accès est accessible en IPv6. | Info |
| `ldap.refuses_plain_bind` | L'annuaire refuse l'authentification sur un canal en clair (`confidentialityRequired`, resultCode 13, selon la RFC 4513 §5.1.2). | Critique |
| `ldap.anonymous_search_blocked` | Signale les annuaires qui autorisent une recherche `baseObject` anonyme du contexte de nommage (divulgation d'information). | Avertissement |
| `ldap.rootdse_readable` | Le RootDSE est lisible sur TLS et annonce des contextes de nommage. | Avertissement |
| `ldap.sasl_mechanisms` | Examine `supportedSASLMechanisms` : présence de mécanismes forts (SCRAM-*, EXTERNAL, GSSAPI), absence de mécanismes équivalents à un mot de passe (PLAIN/LOGIN seuls). | Avertissement |
| `ldap.protocol_version` | Signale les serveurs qui annoncent encore le protocole LDAPv2 déprécié. | Avertissement |
| `ldap.bind_credentials` | Les identifiants de bind fournis sont acceptés par l'annuaire (ne s'exécute que si `bind_dn` est défini). | Critique |
| `ldap.base_dn_read` | Le compte authentifié peut lire le DN de base fourni (ne s'exécute que si `base_dn` est défini et que le bind a réussi). | Critique |
| `ldap.tls_quality` | Réintègre les constats du vérificateur TLS aval (chaîne de certificats, correspondance du nom d'hôte, expiration) sur le service LDAP. | Critique |

Le rapport HTML commence par les défaillances les plus courantes et inclut une remédiation propre à chaque serveur (`olcSecurity` sous OpenLDAP, `require_tls` sous 389-ds...) : aucun point d'accès chiffré accessible, StartTLS absent sur 389, une poignée de main StartTLS qui échoue, un bind en clair accepté sur 389, une poignée de main LDAPS qui échoue, une recherche anonyme exposant le DIT, un SASL limité à PLAIN/LOGIN, un LDAPv2 résiduel, et des identifiants de bind rejetés.

## Options

| Option | Signification | Défaut |
|---|---|---|
| Domaine | Le domaine de l'annuaire (renseigné automatiquement depuis la portée du service). Obligatoire. | *(auto)* |
| Délai d'attente par point d'accès (secondes) | Délai de connexion/sonde pour chaque point d'accès. | `10` |
| Bind DN | Facultatif. DN avec lequel se connecter ; utilisé uniquement si un mot de passe de bind est aussi défini, et seulement sur un canal protégé par TLS. | *(vide)* |
| Mot de passe de bind | Facultatif, secret. Le mot de passe n'est pas conservé dans la charge d'observation et n'est jamais transmis en clair. | *(vide)* |
| DN de base (test de lecture) | Facultatif. Après un bind réussi, une recherche `baseObject` sur ce DN confirme que le compte dispose d'un accès en lecture. Repli sur une recherche `baseObject` anonyme si aucun bind DN n'est fourni. | *(vide)* |

## Dans happyDomain

Activez le vérificateur LDAP depuis l'onglet **Vérifications** d'un service LDAP. Le domaine est renseigné automatiquement ; fournissez un bind DN et un mot de passe uniquement si vous souhaitez exécuter les tests de bind authentifié et de lecture du DN de base. Consultez {{< relref "/pages/checks" >}} pour le fonctionnement complet, et {{< relref "/reference/checkers/tls" >}} pour la posture des certificats de ces mêmes points d'accès.
