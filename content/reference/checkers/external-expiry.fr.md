---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Expiration des références externes
description: "Récupère les informations d'enregistrement WHOIS/RDAP des domaines externes vers lesquels pointe votre zone, afin d'évaluer leur expiration."
weight: 50
---

Le vérificateur **Expiration des références externes** récupère les informations d'enregistrement des domaines *externes* auxquels votre zone fait référence. Lorsqu'un enregistrement pointe vers un nom que vous ne possédez pas (un `CNAME` vers un service tiers, un `MX` vers un prestataire externe, une délégation `NS`, etc.), la sûreté de votre zone dépend du maintien de l'enregistrement de ce domaine externe. S'il expire et qu'il est ré-enregistré par quelqu'un d'autre, le pointeur peut être détourné.

Il s'agit d'un vérificateur de **niveau zone** : il énumère les cibles externes découvertes dans la zone et effectue une requête WHOIS/RDAP par domaine enregistrable distinct. Les cibles partageant le même domaine enregistrable réutilisent une seule requête, et les requêtes s'exécutent avec une concurrence limitée afin qu'une zone volumineuse ne sature pas le registre.

## Ce qui est vérifié

Ce vérificateur est avant tout un **collecteur**. Il rassemble les informations WHOIS par cible (nom enregistrable, date d'expiration, date de création, bureau d'enregistrement, statuts) et les publie pour le vérificateur de *références orphelines*, où sont émis les verdicts exploitables sur l'expiration, la période de rédemption ou un ré-enregistrement récent.

Sa propre règle, `external_whois_collected`, ne rapporte que le déroulement de la collecte :

| Statut | Condition |
|--------|-----------|
| **OK** | Le WHOIS a été collecté pour toutes les cibles externes |
| **Info** | Certaines requêtes ont réussi et d'autres ont échoué (résultat partiel), ou aucune cible externe n'a été signalée |
| **Avertissement** | La requête WHOIS a échoué pour *toutes* les cibles externes |
| **Erreur** | L'observation WHOIS externe n'a pas pu être lue |

{{% notice style="info" title="Les verdicts se trouvent dans le vérificateur de références orphelines" %}}
Ce vérificateur ne décide pas si un domaine externe est dangereusement proche de l'expiration. Il se contente d'en récupérer les informations. Les verdicts d'expiration, de rédemption et de ré-enregistrement sont produits par le vérificateur de références orphelines associé, qui consomme les informations collectées ici.
{{% /notice %}}

## Options

Ce vérificateur ne propose aucune option configurable. La liste des cibles externes est fournie automatiquement à partir des références découvertes dans la zone.

## Dans happyDomain

Activez ce vérificateur depuis la vue **Vérifications** de la zone ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. La découverte des cibles externes est automatique.

Pour l'enregistrement des domaines que vous possédez directement, consultez {{< relref "/reference/checkers/domain-expiry" >}}.
