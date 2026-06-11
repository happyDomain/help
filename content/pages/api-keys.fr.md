---
date: 2026-06-10T12:00:00+02:00
title: Sessions et accès à l'API
weight: 2200
description: "Gérez vos sessions actives et utilisez les jetons d'authentification pour appeler l'API REST de happyDomain"
---

happyDomain ne propose pas de fonctionnalité distincte de « clés d'API ». À la place, toutes les façons d'accéder à la plateforme, que ce soit via l'interface web ou via l'API REST, reposent sur le même type de **jeton de session**. Gérer ses sessions revient donc aussi à gérer ses accès à l'API.

Vous gérez vos sessions depuis la section sécurité de votre compte, sous « Sessions actives ».

<!-- TODO: screenshot of the sessions manager -->

## Lister vos sessions actives

La liste présente toutes les sessions actuellement ouvertes sur votre compte. Pour chacune, vous voyez :

- sa **description** (le nom donné lors de sa création), suivie d'une courte empreinte dérivée du jeton ;
- un badge « session actuelle » sur la session que vous utilisez en ce moment ;
- sa date de **création**, sa date de **dernière utilisation** et sa date d'**expiration**.

Les sessions sont triées de la plus récemment utilisée à la plus ancienne.

## Créer un jeton pour accéder à l'API

Pour obtenir un jeton utilisable pour appeler l'API :

1. Cliquez sur « Créer une clé d'API ».
2. Donnez une **description** à la session (par exemple le nom du script ou de la machine qui l'utilisera). Cela vous aidera à la reconnaître plus tard dans la liste.
3. Validez la création.

Le secret du jeton est alors affiché **une seule fois**. Utilisez le bouton en forme d'œil pour le révéler et le bouton de copie pour le placer dans votre presse-papiers.

{{% notice style="warning" title="Copiez le secret immédiatement" icon="triangle-exclamation" %}}
Le secret de la session n'est affiché qu'au moment de sa création et n'est plus jamais montré ensuite. Copiez-le et conservez-le en lieu sûr avant de fermer la fenêtre. En cas de perte, supprimez la session et créez-en une nouvelle.
{{% /notice %}}

## Utiliser un jeton avec l'API REST

Le jeton se transmet dans l'en-tête HTTP `Authorization`, en tant que jeton « Bearer ». Par exemple, pour lister vos domaines :

```sh
curl -H "Authorization: Bearer VOTRE_JETON_SECRET" https://votre-serveur-happydomain/api/domains
```

Remplacez `VOTRE_JETON_SECRET` par le secret que vous avez copié, et l'hôte par l'adresse de votre instance happyDomain. On atteint n'importe quel point d'accès de l'API de la même manière, en envoyant cet en-tête à chaque requête.

## Révoquer une session

Pour révoquer une session (et le jeton qui lui est associé), cliquez sur le bouton de suppression sur sa ligne. Le jeton cesse aussitôt de fonctionner. Vous ne pouvez pas supprimer depuis cette liste la session que vous utilisez actuellement ; pour y mettre fin, déconnectez-vous simplement, ou utilisez l'option ci-dessous.

{{% notice style="info" title="Fermer toutes les sessions" icon="circle-info" %}}
Le bouton « Fermer toutes les sessions » referme l'ensemble des sessions d'un coup, y compris celle que vous utilisez. C'est utile si vous soupçonnez la fuite d'un jeton : tous les jetons existants deviennent invalides et vous êtes renvoyé vers la [page de connexion]({{% relref "login" %}}). Il vous faudra alors vous reconnecter et recréer les jetons d'API dont vous avez encore besoin.
{{% /notice %}}
