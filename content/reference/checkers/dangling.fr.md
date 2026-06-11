---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Enregistrements orphelins
description: "Analyse une zone à la recherche d'enregistrements CNAME/MX/SRV/NS dont la cible renvoie NXDOMAIN ou dont le domaine externe a expiré et pourrait être ré-enregistré."
weight: 140
---

Le vérificateur **Sous-domaines orphelins** analyse une zone à la recherche d'enregistrements pointeurs (`CNAME`, `MX`, `SRV`, `NS`) dont les cibles sont devenues obsolètes : elles renvoient NXDOMAIN, ou leur domaine enregistrable externe a expiré, est en `pendingDelete`, ou a été ré-enregistré récemment. C'est la classe d'attaques par prise de contrôle de sous-domaine popularisée en 2017, où des institutions ont fini par servir du contenu hostile depuis des CNAME pointant vers des services tiers désaffectés, après que des attaquants eurent ré-enregistré les cibles libérées.

Il s'agit d'un vérificateur de **niveau zone** : il nécessite le contenu complet de la zone et la parcourt en une seule passe, consolidant les constats par propriétaire plutôt que de produire un résultat par enregistrement.

## Ce qui est vérifié

Le vérificateur parcourt chaque service de la zone de travail et extrait les enregistrements pointeurs des corps `CNAME`, CNAME spécial, `MX`, `SRV` inconnu et orphelin (enregistrements `NS`/`CNAME`/`MX` nus). Pour chaque triplet `(propriétaire, type, cible)`, il classe la cible comme interne ou externe à la zone (par rapport au domaine enregistrable de la zone), effectue une résolution DNS unique et bornée dans le temps pour détecter une rupture immédiate, et publie une entrée de découverte afin qu'un vérificateur `domain_expiry` compagnon puisse lancer une requête RDAP/WHOIS sur les cibles externes.

Il émet un constat par propriétaire concerné, classé par sévérité décroissante :

| Signal | Sévérité | Source |
|--------|----------|--------|
| Cible NXDOMAIN | Critique | Résolution DNS locale |
| Cible SERVFAIL | Avertissement | Résolution DNS locale |
| Cible NOERROR avec réponse vide | Info | Résolution DNS locale |
| Domaine enregistrable expiré | Critique | Observation `whois` apparentée |
| Statut enregistrable `pendingDelete` / `redemptionPeriod` | Critique | Observation `whois` apparentée |
| Domaine enregistrable créé au cours des 90 derniers jours | Avertissement | Observation `whois` apparentée |

{{% notice style="info" title="Les signaux WHOIS nécessitent un vérificateur compagnon" %}}
Les signaux issus de la résolution DNS (NXDOMAIN, SERVFAIL, réponse vide) fonctionnent seuls. Les signaux issus du WHOIS (expiré, `pendingDelete`, créé récemment) ne se déclenchent que lorsque le vérificateur `domain_expiry` de l'hôte s'abonne aux entrées de découverte de cibles externes de ce vérificateur et publie une observation `whois` par cible. Sans ce câblage, le vérificateur fonctionne tout de même comme un détecteur d'orphelins basé uniquement sur le DNS.
{{% /notice %}}

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Ignorer la résolution DNS en direct | Lorsqu'elle est activée, le vérificateur ne rapporte que la structure statique des enregistrements pointeurs (analyse hors ligne), sans résoudre les cibles. | `false` |

## Dans happyDomain

Activez ce vérificateur sur le domaine depuis la vue {{< relref "/pages/checks" >}} ; le nom de domaine et le contenu de la zone sont renseignés automatiquement. Étant de niveau zone, il s'exécute sur l'ensemble de la zone en une seule passe.

Vérificateurs apparentés : {{< relref "/reference/checkers/alias" >}} valide la structure des chaînes d'alias individuelles, et {{< relref "/reference/checkers/domain-expiry" >}} surveille l'expiration de vos propres domaines, avec la même mécanique WHOIS qui alimente les signaux de cibles externes de ce vérificateur.
