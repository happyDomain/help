---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Disponibilité du domaine
description: "Vous notifie lorsqu'un nom de domaine surveillé devient disponible à l'enregistrement."
weight: 40
---

Le vérificateur **Disponibilité du domaine** surveille un nom de domaine que vous ne possédez pas et vous notifie dès qu'il devient disponible à l'enregistrement. C'est le pendant de {{< relref "/reference/checkers/domain-expiry" >}} : au lieu de protéger un domaine que vous détenez, il vous permet d'en saisir un dès qu'il se libère.

Il s'agit d'un vérificateur de **niveau domaine** : l'état d'enregistrement est déterminé auprès du registre via une requête WHOIS/RDAP. Un domaine est considéré comme disponible lorsque le registre indique qu'il n'existe pas.

## Ce qui est vérifié

Une seule règle, `domain_availability_check`, indique si le domaine surveillé est toujours enregistré ou s'il s'est libéré.

| Statut | Condition |
|--------|-----------|
| **Critique** | Le domaine est désormais disponible à l'enregistrement |
| **OK** | Le domaine est toujours enregistré (le bureau d'enregistrement et la date d'expiration sont rapportés lorsqu'ils sont connus) |
| **Erreur** | La requête de disponibilité a échoué |

{{% notice style="info" title="Pourquoi la disponibilité est rapportée comme Critique" %}}
Le statut est volontairement inversé par rapport à la convention habituelle. Rapporter *Critique* lorsque le domaine devient disponible fait franchir le seuil de notification à la transition enregistré → disponible, de sorte que vous êtes alerté exactement une fois lorsque le domaine se libère.
{{% /notice %}}

## Options

Ce vérificateur ne propose aucune option configurable. Le nom de domaine surveillé est fourni automatiquement.

## Dans happyDomain

Contrairement aux autres vérificateurs de niveau domaine, **Disponibilité du domaine** n'est pas planifié sur les domaines que vous gérez. Il est piloté par la liste dédiée de surveillance de disponibilité. Consultez {{< relref "/pages/domain-availability" >}} pour savoir comment ajouter un domaine à surveiller, et {{< relref "/pages/checks" >}} pour le fonctionnement général des vérifications.
