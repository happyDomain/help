---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Clés de messagerie (DKIM / OpenPGP)
description: "Valide les clés OpenPGP (OPENPGPKEY) et les certificats S/MIME (SMIMEA) publiés dans le DNS pour le chiffrement de bout en bout du courrier, ainsi que leur authentification DNSSEC."
weight: 230
---

Le vérificateur de **clés de messagerie** (nommé *OPENPGPKEY & SMIMEA* dans happyDomain) valide les clés cryptographiques qu'un domaine publie dans le DNS afin que ses correspondants puissent chiffrer le courrier destiné à ses utilisateurs. Il couvre deux types d'enregistrements de style DANE :

- **`OPENPGPKEY`** ([RFC 7929](https://www.rfc-editor.org/rfc/rfc7929)) : la clé publique OpenPGP d'un utilisateur, publiée sous un nom haché du propriétaire, sous `._openpgpkey.<zone>`.
- **`SMIMEA`** ([RFC 8162](https://www.rfc-editor.org/rfc/rfc8162)) : le certificat S/MIME d'un utilisateur, publié sous un nom haché du propriétaire, sous `._smimecert.<zone>`.

Ce vérificateur s'applique au **niveau service** : il concerne les services OpenPGP et S/MIME d'un sous-domaine, exécute une suite de tests complète, puis produit un rapport HTML dont le bloc supérieur pointe vers la correction des défauts les plus courants.

{{% notice style="info" title="Publication et structure, pas confiance cryptographique" %}}
Ce vérificateur valide la publication DNS ainsi que la structure et les métadonnées des clés qu'il trouve. Il ne les vérifie pas cryptographiquement : les signatures OpenPGP (auto-signatures, certifications par des tiers) ne sont pas vérifiées, et les chaînes S/MIME ne sont ni construites ni validées par rapport à une ancre de confiance (pas de CRL/OCSP). L'authenticité des enregistrements eux-mêmes est déléguée à un résolveur validant via le drapeau DNSSEC `AD`. Considérez un rapport vert comme « l'enregistrement est bien formé et signé par DNSSEC », et non comme « la clé est digne de confiance ».
{{% /notice %}}

## Ce qui est vérifié

### DNS et DNSSEC

| Règle | Ce qui est vérifié | Sévérité |
|---|---|---|
| `dns_query_failed` | La requête DNS de l'enregistrement aboutit. | Critique |
| `dns_no_record` | Un enregistrement est publié au nom de propriétaire attendu. | Critique |
| `dns_record_mismatch` | L'enregistrement renvoyé par le DNS correspond à celui déclaré par le service. | Avertissement |
| `dnssec_not_validated` | L'enregistrement est authentifié par DNSSEC (drapeau `AD` posé). | Critique (Avertissement si DNSSEC non exigé) |
| `owner_hash_mismatch` | Le premier label du nom de propriétaire vaut `hex(sha256(username))[:28]`. | Critique |

### OpenPGP (`OPENPGPKEY`)

| Règle | Ce qui est vérifié | Sévérité |
|---|---|---|
| `pgp_parse_error` | L'enregistrement se décode en une clé OpenPGP valide. | Critique |
| `pgp_primary_revoked` | La clé primaire ne porte aucune signature de révocation. | Critique |
| `pgp_primary_expired` | La clé primaire n'a pas dépassé l'expiration de son auto-signature. | Critique |
| `pgp_primary_expiring_soon` | La clé primaire n'expire pas dans la fenêtre configurée. | Avertissement |
| `pgp_weak_algorithm` | Aucun algorithme ancien (DSA/ElGamal) n'est employé. | Avertissement |
| `pgp_weak_key_size` | Les clés RSA respectent la taille minimale de 2048 bits (3072+ préférée). | Critique |
| `pgp_no_encryption_subkey` | Au moins une clé active annonce une capacité de chiffrement. | Critique |
| `pgp_no_identity` | La clé porte au moins un identifiant utilisateur auto-signé. | Avertissement |
| `pgp_uid_mismatch` | Au moins un UID référence `<username@...>`. | Info |
| `pgp_multiple_entities` | L'enregistrement porte une seule entité OpenPGP (RFC 7929). | Avertissement |
| `pgp_record_too_large` | L'enregistrement reste sous 4 Kio pour tenir dans une réponse UDP typique. | Avertissement |

### S/MIME (`SMIMEA`)

| Règle | Ce qui est vérifié | Sévérité |
|---|---|---|
| `smimea_bad_usage` | Le champ usage vaut 0, 1, 2 ou 3. | Critique |
| `smimea_bad_selector` | Le champ sélecteur vaut 0 (Cert) ou 1 (SPKI). | Critique |
| `smimea_bad_match_type` | Le type de correspondance vaut 0 (Full), 1 (SHA-256) ou 2 (SHA-512). | Critique |
| `smimea_cert_parse_error` | L'enregistrement se décode en un certificat X.509 ou un SPKI valide. | Critique |
| `smimea_cert_not_yet_valid` | Le `NotBefore` du certificat est dans le passé. | Critique |
| `smimea_cert_expired` | Le `NotAfter` du certificat est dans le futur. | Critique |
| `smimea_cert_expiring_soon` | Le certificat n'expire pas dans la fenêtre configurée. | Avertissement |
| `smimea_no_email_protection_eku` | Le certificat annonce l'EKU `emailProtection`. | Critique (Avertissement si non exigé) |
| `smimea_missing_key_usage` | Le certificat porte l'usage de clé `digitalSignature` et/ou `keyEncipherment`. | Avertissement |
| `smimea_weak_signature_algorithm` | Le certificat n'est pas signé avec un algorithme déprécié (MD2/MD5/SHA-1). | Critique |
| `smimea_weak_key_size` | Les clés RSA respectent la taille minimale de 2048 bits (3072+ préférée). | Critique |
| `smimea_self_signed` | Signale les certificats auto-signés associés à PKIX-EE (usage 1). | Info |
| `smimea_email_mismatch` | Au moins un SAN courriel commence par `<username>@`. | Info |
| `smimea_hash_only` | Note que les types de correspondance 1/2 ne transportent qu'un condensat, empêchant l'inspection du certificat. | Info |

## Options

| Option | Signification | Défaut |
|---|---|---|
| Résolveur DNS (`resolver`) | Résolveur validant à interroger (liste séparée par des virgules acceptée). Vide : résolveur système. | (système) |
| `certExpiryWarnDays` | Fenêtre, en jours, des avertissements `expiring_soon` (PGP et S/MIME). | 30 |
| `requireDNSSEC` | Si faux, un drapeau `AD` absent devient un Avertissement plutôt qu'un Critique. | true |
| `requireEmailProtection` | Si faux, une EKU `emailProtection` absente devient un Avertissement plutôt qu'un Critique. | true |

L'origine de la zone, le sous-domaine, le service et le type de service sont préremplis par happyDomain.

{{% notice style="info" title="Interrogez un résolveur validant" %}}
L'authenticité des enregistrements étant déléguée au DNSSEC, exécutez ce vérificateur contre un résolveur en qui vous avez confiance pour effectuer la validation DNSSEC, afin que le drapeau `AD` reflète une validation réelle.
{{% /notice %}}

## Dans happyDomain

Activez ce vérificateur depuis l'onglet **Vérifications** du service OpenPGP ou S/MIME concerné. Voir {{< relref "/pages/checks" >}} pour le fonctionnement général.

Ces enregistrements partagent leur modèle de sécurité avec le DNSSEC : pour confirmer que la chaîne de signature de votre zone est elle-même saine, voir {{< relref "/reference/checkers/dnssec" >}}. Pour la configuration de messagerie environnante, voir {{< relref "/reference/services/email" >}}.
