---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: CAA
description: "Recoupe les certificats TLS observés sur un domaine avec sa politique CAA issue/issuewild pour détecter les émissions non autorisées."
weight: 150
---

Le vérificateur **Conformité CAA** vérifie que les certificats TLS réellement déployés sur un domaine ont été émis par une autorité de certification que les enregistrements `CAA` du domaine autorisent. Un enregistrement `CAA` (Certification Authority Authorization) permet à un domaine de déclarer quelles autorités de certification peuvent émettre des certificats pour lui ; ce vérificateur confirme que la réalité correspond à cette politique.

Il s'agit d'un vérificateur de **niveau service** : il s'exécute sur un service de politique `CAA`. Il n'effectue **aucune sonde réseau** : il lit la politique CAA déjà analysée dans le corps du service et les certificats TLS observés par la famille {{< relref "/reference/checkers/dane" >}} / `checker-tls`, puis associe chaque émetteur observé à son domaine d'identifiant CAA à l'aide de la base de données Common CA Database (CCADB).

## Ce qui est vérifié

Une seule règle, `caa_compliance`, charge la politique CAA `issue` / `issuewild` de la zone, rassemble chaque sonde TLS observée sur la cible, résout l'émetteur de chaque certificat (par Authority Key Identifier, avec repli sur le Distinguished Name de l'émetteur) à l'aide du mappage « CAA Identifiers » embarqué de CCADB, puis compare le résultat à la liste d'autorisations.

| Résultat | Signification | Statut |
|----------|---------------|--------|
| `caa_ok` | Chaque émetteur observé est autorisé par la politique CAA. | OK |
| `caa_no_tls` | Aucune sonde TLS liée à cette cible n'a encore été publiée. | Inconnu |
| `caa_not_authorized` | CCADB a associé un émetteur observé à un domaine que la politique ne liste pas. | Critique |
| `caa_issuance_disallowed` | La politique contient `CAA 0 issue ";"` (émission explicitement interdite) mais un certificat a tout de même été observé. | Critique |
| `caa_issuer_unknown` | CCADB n'a aucun mappage pour l'émetteur observé (AKI + DN). | Info |

{{% notice style="info" title="Cohérence à terme avec les sondes TLS" %}}
L'état `caa_no_tls` est un état stable normal, pas une erreur : tant que `checker-tls` n'a pas sondé un point d'accès sur la cible ni publié de certificat, le vérificateur CAA n'a rien à comparer et rapporte **Inconnu**. Dès que les sondes arrivent, la vérification se résout à son exécution suivante. Le résultat `caa_issuer_unknown` signifie simplement que l'autorité n'est pas dans l'instantané CCADB courant ; le correctif consiste à soumettre une mise à jour à CCADB, pas à modifier votre zone.
{{% /notice %}}

Le mappage émetteur vers domaine CAA provient d'un instantané embarqué du rapport « CAA Identifiers (V2) » de CCADB. Le rafraîchir ne demande que de ré-embarquer un CSV plus récent et de recompiler : aucun changement de code ni dépendance réseau à la compilation.

## Options

Ce vérificateur n'a aucune option réglable par l'utilisateur. Ses deux entrées sont renseignées automatiquement :

| Option | Signification |
|--------|---------------|
| Domaine | Le domaine vérifié (renseigné automatiquement, requis). |
| Service | Le corps du service de politique CAA (renseigné automatiquement). |

## Dans happyDomain

Ajoutez le service de politique CAA au sous-domaine concerné (généralement l'apex), puis activez ce vérificateur depuis l'onglet **Vérifications** de ce service. Consultez {{< relref "/pages/checks" >}} pour configurer et planifier les vérifications. Pour que la comparaison produise un verdict, un vérificateur de sonde TLS comme {{< relref "/reference/checkers/dane" >}} (ou le `checker-tls` autonome) doit avoir observé des certificats sur la cible.
