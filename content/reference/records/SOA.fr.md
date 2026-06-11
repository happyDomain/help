---
date: 2020-12-15T01:01:08+01:00
title: SOA (Start Of Authority)
description: "Comprendre l'enregistrement Start Of Authority, présent obligatoirement à la racine de chaque zone DNS, et la façon dont happyDomain le gère pour vous."
weight: 10
aliases:
    records/SOA
---

L'enregistrement **Start Of Authority** (SOA) est défini par la [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035). Il est obligatoire et unique : un seul SOA figure à l'apex (la racine) de chaque zone DNS. Sa présence indique que le serveur de noms fait autorité sur la zone. Il porte également les paramètres qui régissent la réplication de la zone entre serveurs et sa mise en cache par les résolveurs.

## Les champs de l'enregistrement SOA

Le SOA réunit sept valeurs :

| Champ | Description |
|---|---|
| **MNAME** (serveur primaire) | Le nom du serveur de noms primaire (maître) de la zone. |
| **RNAME** (responsable) | L'adresse de courriel du responsable de la zone, encodée à la manière du DNS : le `@` est remplacé par un point. Ainsi, `hostmaster.example.com.` désigne `hostmaster@example.com`. |
| **Serial** | Un numéro de version de la zone. Il augmente à chaque modification, afin que les serveurs secondaires sachent qu'ils doivent transférer le nouveau contenu. |
| **Refresh** | L'intervalle (en secondes) au bout duquel un serveur secondaire vérifie le numéro de série auprès du primaire. |
| **Retry** | Le délai (en secondes) que respecte un secondaire avant de retenter une vérification ayant échoué. |
| **Expire** | La durée (en secondes) pendant laquelle un secondaire continue de servir la zone lorsqu'il ne parvient pas à joindre le primaire, avant de considérer les données comme périmées. |
| **Minimum** (cache négatif) | La durée pendant laquelle les résolveurs conservent en cache les réponses négatives (NXDOMAIN), selon la [RFC 2308](https://www.rfc-editor.org/rfc/rfc2308). |

## Le SOA dans happyDomain

happyDomain ne présente pas le SOA comme un enregistrement à éditer champ par champ. La racine de votre zone est plutôt modélisée par un service **Origin**, qui regroupe le SOA et les serveurs de noms de la zone (les enregistrements NS). On retrouve donc le SOA à la racine du domaine, aux côtés de la liste des serveurs faisant autorité, et non dans un formulaire distinct.

Le **numéro de série** est, dans la plupart des cas, géré automatiquement. Lors de la publication de vos modifications, de nombreux hébergeurs DNS gèrent eux-mêmes ce numéro. happyDomain détecte cette capacité, puis relit la zone après publication afin de refléter le numéro de série réellement attribué par l'hébergeur. Vous n'avez normalement pas à le renseigner à la main.

{{% notice style="info" title="Ce que l'on peut modifier, et ce que l'on ne peut pas" %}}
Le comportement exact dépend de votre hébergeur DNS. Certains exposent l'ensemble du SOA et laissent happyDomain en transmettre les valeurs ; d'autres gèrent eux-mêmes le numéro de série (et parfois les autres minuteries). Lorsque l'hébergeur prend en charge le numéro de série, la valeur affichée par happyDomain reflète simplement l'état publié et se met à jour automatiquement.
{{% /notice %}}

Pour en savoir plus sur la représentation de l'apex et des autres regroupements, consultez le chapitre {{< relref "../services" >}}.
