---
date: 2020-12-15T01:01:08+01:00
title: TXT
weight: 20
aliases:
    records/TXT
description: "L'enregistrement TXT associe du texte libre à un nom DNS. Découvrez à quoi il sert et comment happyDomain l'édite grâce au service Enregistrement texte."
---

Un **enregistrement TXT** (défini par la [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035)) associe une ou plusieurs chaînes de texte libre à un nom de votre zone. Il ne porte aucune signification propre. C'est ce qui en fait l'un des types d'enregistrement les plus polyvalents : chaque application est libre de définir sa propre convention pour le texte qu'elle y dépose.

## Usages courants

Comme un enregistrement TXT peut contenir n'importe quel texte, il est devenu le support de nombreuses conventions répandues :

- **Vérification de propriété ou de site** : un fournisseur demande de publier un jeton afin de confirmer que l'on contrôle bien le domaine.
- **SPF** : déclare quels serveurs sont autorisés à envoyer du courrier pour le domaine.
- **DKIM** : publie la clé publique servant à signer le courrier sortant.
- **DMARC** : définit la politique à appliquer lorsqu'une vérification SPF ou DKIM échoue.
- Diverses autres publications de clés ou de politiques propres à différents outils.

Pour la plupart de ces usages, happyDomain propose des **services** dédiés, de plus haut niveau (SPF, DKIM, DMARC, vérification de site, etc.). Ils sont plus simples et plus sûrs à utiliser qu'un enregistrement TXT brut : ils guident la saisie avec les bons champs et en valident la syntaxe. On les retrouve dans le chapitre {{< relref "/reference/services" >}}, notamment parmi les services liés au courrier électronique. Mieux vaut recourir à ces services dès que l'un d'eux correspond au besoin.

{{% notice style="info" title="Quand un enregistrement TXT devient un service" %}}
Lorsque happyDomain lit votre zone, il reconnaît les enregistrements TXT qui suivent une convention connue (SPF, DKIM, DMARC, etc.) et les présente sous leur service dédié plutôt que comme un simple enregistrement texte. Seuls les enregistrements TXT sans préfixe ni syntaxe reconnus apparaissent comme un **Enregistrement texte** brut.
{{% /notice %}}

## Éditer un enregistrement TXT dans happyDomain

Dans l'éditeur de zone, un enregistrement TXT ordinaire apparaît sous la forme d'un service **Enregistrement texte**. Il est volontairement minimal : un unique champ contient le contenu textuel de l'enregistrement.

Pour le manipuler :

1. Ouvrez le sous-domaine où l'enregistrement doit se trouver (la racine de la zone, ou un sous-domaine comme `www`).
2. Ajoutez ou ouvrez le service **Enregistrement texte**.
3. Saisissez la chaîne de texte complète dans son unique champ.
4. Ajustez la **durée de vie** (TTL) si besoin, puis publiez vos modifications pour les appliquer.

La valeur saisie est conservée telle quelle. La modifier puis publier met à jour l'enregistrement TXT correspondant sur ce sous-domaine.

{{% notice style="note" title="Chaînes longues" %}}
Au niveau du protocole DNS, une chaîne de texte ne peut excéder 255 octets dans un enregistrement TXT. Les valeurs plus longues sont automatiquement découpées en fragments de 255 octets. Il suffit de saisir la chaîne complète dans happyDomain : aucun découpage manuel n'est nécessaire.
{{% /notice %}}
