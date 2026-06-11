---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Fédération Matrix
description: "Vérifie qu'un serveur Matrix fédère correctement, à l'aide du Matrix Federation Tester."
weight: 300
---

Le vérificateur **Fédération Matrix** vérifie qu'un serveur Matrix (homeserver) est correctement configuré pour fédérer avec le reste du réseau Matrix. Il délègue le sondage proprement dit à une instance du [Matrix Federation Tester](https://federationtester.matrix.org/), stocke le rapport complet sous forme d'observation, et produit un résumé HTML détaillé couvrant les connexions, les certificats, la délégation well-known et la résolution DNS/SRV.

Il s'agit d'un vérificateur de **niveau service**. Il s'applique aux services de type **Matrix** (messagerie instantanée) et se configure depuis l'onglet **Vérifications** de ce service.

## Ce qui est vérifié

| Règle | Ce qu'elle vérifie | Sévérité |
|-------|--------------------|----------|
| `matrix.connection_reachable` | Chaque point d'accès de fédération découvert accepte une connexion entrante. | Critique |
| `matrix.federation_ok` | Le statut global de fédération rapporté par le Matrix Federation Tester. | Critique |
| `matrix.srv_records` | La résolution SRV Matrix (`_matrix-fed._tcp` / `_matrix._tcp`) a réussi ou a été légitimement ignorée. | Critique |
| `matrix.tls_checks` | La posture TLS sur chaque point d'accès de fédération joignable (chaîne de certificats, nom d'hôte, clé Ed25519). | Critique |
| `matrix.version` | Le serveur répond à `/_matrix/federation/v1/version` avec son nom et sa version. | Avertissement |
| `matrix.well_known` | `/.well-known/matrix/server`, lorsqu'il est publié, est valide et pointe vers le `server_name` déclaré. | Critique |

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Domaine Matrix | Le `server_name` Matrix à tester. Renseigné automatiquement depuis le service. | `matrix.org` |

Une option supplémentaire de **niveau administrateur**, `federationTesterServer`, définit le modèle d'URL de l'instance du Federation Tester à interroger. Elle est configurée par l'opérateur happyDomain, et non par vérification, et vaut par défaut `https://federationtester.matrix.org/api/report?server_name=%s`.

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** d'un service Matrix ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. Le domaine Matrix est renseigné automatiquement depuis le service.
