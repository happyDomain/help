---
date: 2026-06-10T12:00:00+02:00
title: Fonctionnalités des fournisseurs
weight: 700
description: "Comparer ce que prend en charge chaque hébergeur DNS grâce à la matrice des fonctionnalités"
---

Tous les hébergeurs DNS ne prennent pas en charge les mêmes capacités. Certains savent lister automatiquement les domaines de votre compte, d'autres ne gèrent que les types d'enregistrements les plus courants, et quelques-uns prennent en charge des enregistrements plus spécialisés comme `CAA` ou `TLSA`. La page **Fournisseurs supportés** présente ces informations sous la forme d'un unique tableau comparatif, afin de vous aider à choisir le bon hébergeur, ou à comprendre pourquoi un [service]({{% relref "services" %}}) donné n'est pas disponible pour l'un de vos domaines.

## Lire la matrice des fonctionnalités

La page liste tous les hébergeurs avec lesquels happyDomain s'intègre, un par ligne, avec son logo et son nom. Chaque colonne correspond à une capacité, et la cellule indique si l'hébergeur la prend en charge :

- une coche verte signifie que la capacité **est prise en charge** ;
- une croix rouge signifie qu'elle **n'est pas prise en charge**.

<!-- TODO: capture d'écran de la matrice des fonctionnalités des fournisseurs -->

Le tableau défile horizontalement s'il est plus large que votre écran, et la ligne d'en-tête reste visible pendant le défilement : vous savez ainsi toujours à quelle capacité se rapporte chaque colonne.

## Les capacités

La matrice compare les capacités suivantes :

| Colonne | Signification |
|---------|---------------|
| **Fournisseurs supportés** | L'hébergeur sait lister automatiquement les domaines de votre compte, ce qui vous permet de les importer sans saisir chaque nom. Lorsque ce n'est pas pris en charge, vous ajoutez les domaines manuellement. |
| **Types courants** | L'hébergeur prend en charge les types d'enregistrements du quotidien (comme `A`, `AAAA`, `MX`, `TXT`, `CNAME`). C'est ce dont la plupart des domaines ont besoin. |
| **CAA** | Prise en charge des enregistrements `CAA`, qui déclarent les autorités de certification autorisées à émettre des certificats pour votre domaine. |
| **OPENPGPKEY** | Prise en charge des enregistrements `OPENPGPKEY`, utilisés pour publier des clés publiques OpenPGP dans le DNS. |
| **PTR** | Prise en charge des enregistrements `PTR`, utilisés principalement pour le DNS inverse (associer une adresse IP à un nom). |
| **SRV** | Prise en charge des enregistrements `SRV`, qui annoncent l'emplacement d'un service (port et hôte) pour les protocoles qui s'appuient dessus. |
| **SSHFP** | Prise en charge des enregistrements `SSHFP`, qui publient les empreintes des clés d'hôte SSH dans le DNS. |
| **TLSA** | Prise en charge des enregistrements `TLSA`, utilisés par DANE pour lier un certificat à un nom. |

{{% notice style="info" title="Pourquoi c'est important" icon="circle-info" %}}
Lorsque vous ajoutez un [service]({{% relref "services" %}}) à un sous-domaine, happyDomain ne propose que les types de services que votre hébergeur peut réellement publier. Si un service que vous attendiez apparaît grisé, la matrice des fonctionnalités est l'endroit où vérifier si le type d'enregistrement sous-jacent est pris en charge par cet hébergeur.
{{% /notice %}}

## Choisir un hébergeur

Si vous hésitez encore sur l'endroit où héberger votre DNS, servez-vous de cette page pour vous assurer que l'hébergeur envisagé couvre les types d'enregistrements dont vous avez besoin. Un hébergeur qui prend en charge les **types courants** suffit pour la plupart des sites web et des configurations de messagerie ; les colonnes spécialisées ne comptent que si vous utilisez les fonctionnalités correspondantes (DANE, DNS inverse, empreintes SSH, etc.).

Une fois votre hébergeur choisi, rendez-vous sur la page pour l'[ajouter]({{% relref "provider-new-choice" %}}) à happyDomain.
