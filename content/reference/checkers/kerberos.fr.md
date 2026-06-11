---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Kerberos
description: "Audite un royaume Kerberos à partir de ses enregistrements DNS : disposition SRV, accessibilité KDC/kadmin/kpasswd, sonde AS-REQ anonyme (royaume, enctypes, pré-authentification, dérive d'horloge) et aller-retour authentifié facultatif."
weight: 260
---

Le vérificateur **Kerberos** audite un royaume Kerberos à partir de ses enregistrements DNS. À partir du nom du royaume (dérivé en majuscules depuis le domaine) et des enregistrements SRV regroupés sous le service *Kerberos*, il exécute une série de **sondes anonymes** et, lorsque des identifiants sont fournis, un **aller-retour authentifié** facultatif, ce qui donne une vue combinée de la disponibilité et de la posture de sécurité du royaume.

Il s'agit d'un vérificateur de **niveau service** : il cible un service *Kerberos* (`abstract.Kerberos`) publié sur un sous-domaine et se configure depuis l'onglet **Vérifications** de ce service. Il inspecte la disposition SRV (`_kerberos._tcp`, `_kerberos._udp`, `_kerberos-master._tcp`, `_kerberos-adm._tcp`, `_kpasswd._tcp`, `_kpasswd._udp`), résout en avant chaque cible SRV (A + AAAA), teste l'accessibilité TCP de chaque hôte KDC/kadmin/kpasswd et l'accessibilité UDP du KDC via un véritable AS-REQ. La sonde AS-REQ anonyme confirme le royaume, lit les enctypes pris en charge depuis `ETYPE-INFO2`, détecte un indice PKINIT (`PA-PK-AS-REQ`) et mesure la dérive d'horloge.

{{% notice style="info" title="Les identifiants sont transmis au KDC" %}}
Lorsqu'un principal et un mot de passe sont fournis, ils servent une fois à obtenir un TGT puis à effectuer un TGS-REQ pour le service cible ; ils sont transmis au KDC sur le réseau et ne sont jamais stockés par le vérificateur. Laissez-les vides pour n'exécuter que les sondes anonymes.
{{% /notice %}}

## Ce qui est vérifié

| Règle | Ce qui est vérifié | Gravité |
|---|---|---|
| `kerberos.srv_present` | Au moins un enregistrement SRV `_kerberos._tcp` / `_kerberos._udp` est publié pour le royaume. | Critique |
| `kerberos.kdc_reachable` | Au moins un point d'accès KDC (TCP/UDP 88) accepte une connexion. | Critique |
| `kerberos.as_probe` | La sonde AS-REQ anonyme a reçu une réponse cohérente (`KRB-ERROR` ou `AS-REP`). | Critique |
| `kerberos.realm_match` | Le KDC répond pour le nom de royaume attendu. | Critique |
| `kerberos.preauth_required` | Signale les KDC qui renvoient un `AS-REP` sans exiger de pré-authentification (exposition à l'AS-REP roasting). | Avertissement |
| `kerberos.clock_skew` | L'horloge du KDC est dans la tolérance par rapport à celle du vérificateur. | Critique |
| `kerberos.enctypes` | Examine les types de chiffrement annoncés par le KDC, signalant les configurations limitées à DES/RC4. | Critique |
| `kerberos.kadmin_reachable` | Signale les points d'accès kadmin publiés via SRV mais inaccessibles. | Avertissement |
| `kerberos.kpasswd_reachable` | Signale les points d'accès kpasswd publiés via SRV mais inaccessibles. | Avertissement |
| `kerberos.auth_tgt` | Le principal/mot de passe fourni peut obtenir un TGT (ne s'exécute que si des identifiants sont fournis). | Critique |
| `kerberos.auth_tgs` | Un TGS-REQ réussit pour le service cible fourni (ne s'exécute que si des identifiants et un service cible sont fournis). | Avertissement |

Le rapport HTML met en avant les erreurs de configuration les plus courantes avec une piste de remédiation directe : absence d'enregistrements SRV (publier `_kerberos._tcp.REALM. SRV …`), une cible SRV sans A/AAAA, le port 88 inaccessible (ouvrir TCP+UDP 88), une dérive d'horloge au-dessus du maximum (lancer ntpd/chrony), des royaumes limités aux enctypes faibles (passer à `aes256-cts-hmac-sha1-96`), un mauvais royaume dans la réponse, et l'exposition à l'AS-REP roasting (activer `requires_preauth`).

## Options

| Option | Signification | Défaut |
|---|---|---|
| Royaume Kerberos | Domaine DNS annonçant le royaume (renseigné automatiquement depuis la portée du service ; le nom du royaume est dérivé en majuscules). Obligatoire. | *(auto)* |
| Principal | Facultatif. À fournir pour exécuter un aller-retour authentifié ; laissez vide pour n'exécuter que les sondes anonymes. | *(vide)* |
| Mot de passe | Facultatif, secret. Mot de passe du principal ci-dessus ; utilisé une fois par exécution et jamais stocké. | *(vide)* |
| Service à demander (TGS) | Facultatif. SPN demandé via TGS-REQ une fois un TGT obtenu. Par défaut `krbtgt` (auto-test du royaume). | *(vide)* |
| Délai d'attente par sonde (secondes) | Délai d'attente pour chaque sonde. | `5` |
| Exiger des enctypes forts | Lorsqu'activé, les royaumes n'annonçant que DES/RC4 sont signalés comme Critique. | `true` |
| Dérive d'horloge maximale tolérée (secondes) | La tolérance Kerberos par défaut est de 300 s ; des valeurs plus strictes font apparaître la dérive plus tôt. | `300` |

## Dans happyDomain

Activez le vérificateur Kerberos depuis l'onglet **Vérifications** d'un service Kerberos. Le domaine du royaume est renseigné automatiquement ; fournissez un principal et un mot de passe uniquement si vous souhaitez exécuter l'aller-retour authentifié TGT/TGS. Consultez {{< relref "/pages/checks" >}} pour le fonctionnement complet.
