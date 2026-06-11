---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Contacts du domaine
description: "Vérifie que les contacts enregistrés du domaine (titulaire, administratif, technique) correspondent aux valeurs attendues, avec détection des enregistrements protégés par confidentialité."
weight: 30
---

Le vérificateur **Contacts du domaine** compare les informations de contact enregistrées pour un domaine (contacts titulaire, administratif et technique) aux valeurs que vous attendez. Une modification inattendue d'un contact, en particulier le titulaire, peut être un signe précoce de détournement ou d'une erreur administrative.

Il s'agit d'un vérificateur de **niveau domaine** : les données de contact sont lues auprès du registre via une requête WHOIS/RDAP. De nombreux registres masquent les champs de contact pour des raisons de confidentialité ; ce vérificateur détecte ce masquage et le signale au lieu de le traiter comme une divergence.

## Ce qui est vérifié

Une seule règle, `domain_contact_check`, évalue chaque rôle de contact sélectionné et émet un résultat par rôle.

| Statut | Condition |
|--------|-----------|
| **OK** | Le contact du rôle correspond à chaque valeur attendue fournie |
| **Avertissement** | Un champ ne correspond pas à la valeur attendue, ou le rôle est absent de l'enregistrement |
| **Info** | Le contact est protégé par confidentialité (masqué) et ne peut donc pas être comparé |
| **Inconnu** | Aucune valeur attendue n'est configurée, ou aucun rôle n'est sélectionné (rien à vérifier) |
| **Erreur** | La requête WHOIS/RDAP a échoué |

Les comparaisons sont exactes et insensibles à la casse. Le masquage est détecté à partir de marqueurs courants dans les champs de contact comme *redacted*, *privacy*, *withheld*, *not disclosed*, *data protected*, *contact privacy* et *whoisguard*.

## Options

| Option | Signification | Défaut |
|--------|---------------|--------|
| Nom du titulaire attendu | Si renseigné, les rôles configurés doivent rapporter ce nom exact (insensible à la casse). | *(vide)* |
| Organisation attendue | Si renseignée, les rôles configurés doivent rapporter cette organisation exacte (insensible à la casse). | *(vide)* |
| Adresse e-mail attendue | Si renseignée, les rôles configurés doivent rapporter cette adresse exacte (insensible à la casse). | *(vide)* |
| Rôles de contact à vérifier | Liste de rôles séparés par des virgules parmi `registrant`, `admin`, `tech`. | `registrant` |

{{% notice style="info" title="Au moins une valeur attendue est nécessaire" %}}
Si ni le nom, ni l'organisation, ni l'adresse e-mail attendus ne sont renseignés, le vérificateur n'a rien à comparer et rapporte un statut *Inconnu*. Renseignez au moins une valeur attendue pour que la vérification ait du sens.
{{% /notice %}}

## Dans happyDomain

Activez ce vérificateur depuis la vue **Vérifications** du domaine ; consultez {{< relref "/pages/checks" >}} pour savoir comment configurer et planifier les vérifications. Le nom de domaine est renseigné automatiquement.

Ce vérificateur complète {{< relref "/reference/checkers/domain-expiry" >}} et {{< relref "/reference/checkers/domain-lock" >}}, qui ensemble gardent l'enregistrement d'un domaine sous surveillance.
