---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Expiration du domaine
description: "Avertit lorsqu'un nom de domaine approche de sa date d'expiration."
weight: 10
---

Le vérificateur **Expiration du domaine** surveille la date d'expiration de l'enregistrement d'un nom de domaine et vous avertit avant qu'elle ne soit dépassée. Laisser un domaine expirer peut conduire à le perdre au profit d'un autre titulaire : c'est l'une des vérifications les plus importantes au niveau du domaine.

Il s'agit d'un vérificateur de **niveau domaine** : il concerne l'enregistrement du domaine lui-même, et non ses enregistrements DNS. La date d'expiration est obtenue auprès du registre via une requête WHOIS/RDAP, en même temps que le nom du bureau d'enregistrement.

## Ce qui est vérifié

Une seule règle, `domain_expiry_check`, compare le nombre de jours restant avant l'expiration à deux seuils et rapporte le statut correspondant.

| Statut | Condition |
|--------|-----------|
| **Critique** | Jours restants ≤ seuil critique |
| **Avertissement** | Jours restants ≤ seuil d'avertissement (mais au-dessus du critique) |
| **OK** | Jours restants au-dessus du seuil d'avertissement |
| **Erreur** | La requête WHOIS/RDAP a échoué ou aucune date d'expiration n'est disponible |

Le message indique toujours le nombre de jours restant avant l'expiration, quel que soit le statut.

Le vérificateur expose également une métrique, `domain_expiry_days_remaining`, étiquetée avec le bureau d'enregistrement, afin de suivre l'évolution du temps restant.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Seuil d'avertissement (jours) | Nombre de jours avant l'expiration déclenchant un avertissement. Doit être positif. | 30 |
| Seuil critique (jours) | Nombre de jours avant l'expiration déclenchant une alerte critique. Doit être positif. | 7 |

{{% notice style="info" title="Le critique doit être inférieur à l'avertissement" %}}
Le seuil critique doit être strictement inférieur au seuil d'avertissement. happyDomain refuse une configuration où `critical_days` est supérieur ou égal à `warning_days`.
{{% /notice %}}

## Dans happyDomain

Activez ce vérificateur depuis la vue **Vérifications** du domaine ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. Le nom de domaine est renseigné automatiquement.

Pour la situation inverse, surveiller un domaine que vous ne possédez pas encore afin de l'enregistrer dès qu'il se libère, consultez le vérificateur {{< relref "/reference/checkers/domain-availability" >}} et {{< relref "/pages/domain-availability" >}}. Les vérificateurs de niveau domaine apparentés incluent {{< relref "/reference/checkers/domain-lock" >}} et {{< relref "/reference/checkers/domain-contact" >}}.
