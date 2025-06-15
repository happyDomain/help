---
data: 2024-06-26T20:44:25+02:00
title: Connexion à un serveur BIND distant
weight: 30
---

[BIND](https://www.isc.org/bind/) est un serveur DNS récursif et faisant autorité développé par l'[Internet Systems Consortium](https://isc.org).

Il est possible de l'utiliser avec happyDomain en passant par le [Dynamic DNS (RFC 2136)](https://www.rfc-editor.org/rfc/rfc2136).

Cette documentation vous guidera dans la configuration de BIND pour activer le DNS dynamique, puis pour relier ensemble happyDoain et votre serveur BIND.


## Configurer BIND pour activer le DNS dynamique

Tout d'abord, vous devez modifier le fichier de configuration principal de BIND (généralement `/etc/named.conf` ou `/etc/bind/named.conf` selon votre distribution) pour ajouter un secret qui sera partagé entre happyDomain et BIND pour authentifier les modifications. Ensuite, vous devez indiquer quels domaines seront gérés par happyDomain.

### Ajouter un secret partagé

Sous la section principale `key` de votre configuration, ajoutez la clé suivante :

```conf
key "happydomain" {
    algorithm hmac-sha512;
    secret "<SOME_SECRET>";
};
```

Remplacez `<SOME_SECRET>` par une chaîne obtenue en utilisant `openssl rand -base64 48`.

### Créer une règle d'autorisation pour happyDomain

En plus de la clé, vous devez spécifier comment la clé peut être utilisée en définissant une ACL et en autorisant les mises à jour à partir de celle-ci.

Ajoutez l'ACL suivante à votre configuration :

```conf
acl "happydomain_acl" {
    key happydomain;
};
```

### Autoriser les mises à jour pour chaque zone

Maintenant que vous avez créé une règle permettant à la clé `happydomain` d'apporter des modifications, vous devez indiquer à quelles zones cette règle s'applique.

Pour chaque zone, vous devez ajouter une déclaration `update-policy` référencant l'ACL `happydomain_acl` :

Par exemple, pour une zone existante `happydomain.org`, ajoutez la déclaration `update-policy` comme suit :

```conf
zone "happydomain.org" {
    type master;
    file "/var/named/happydomain.org.db";
    update-policy {
        grant happydomain_acl name happydomain.org. ANY;
    };
};
```

La déclaration `update-policy` est une liste, donc vous pouvez déjà avoir d'autres politiques dans cette liste. Dans ce cas, ajoutez simplement la déclaration `grant` pour `happydomain_acl`.

### Autoriser les mises à jour pour routes les zones

Si vous gérez de nombreuses zones, il peut être plus pratique de définir l'autorisation par défaut pour toutes les zones. Dans ce cas, vous pouvez utiliser une `update-policy` globale dans la section `options` :

```conf
options {
    update-policy {
        grant happydomain_acl zonesub ANY;
    };
};
```

Cela appliquera la `update-policy` à toutes les zones, permettant à l'ACL `happydomain_acl` de mettre à jour n'importe quel enregistrement.

### Appliquer la configuration

Après avoir modifié le fichier de configuration, rechargez le service BIND pour appliquer les modifications :

```sh
rndc reload
```

## Lier happyDomain et BIND

Une fois BIND bien configuré, vous pouvez le lier à happyDomain en utilisant [le connecteur *Dynamic DNS*]({{% ref "/pages/provider-new-choice.md" %}}) :.

![Le connecteur Dynamic DNS sur la page de choix de l'hébergeur](/img/choose-dynamic-dns.png)

Remplissez ensuite le formulaire avec l'adresse à laquelle votre serveur BIND est accessible, puis ensuite les différents champs *Key* avec les informations indiqués dans votre configuration :

- **Key Name** : correspond au champ `id` ;
- **Key Algorithm** : correspond au champ `algorithm` ;
- **Secret Key** : correspond au champ `secret`.

Une fois le fournisseur ajouté, il ne permet pas de lister les domaines existants, mais vous pouvez toujours ajouter manuellement tous vos domaines.

En suivant ces étapes, vous aurez configuré BIND pour fonctionner avec happyDomain en utilisant le DNS dynamique, assurant des mises à jour DNS sécurisées et authentifiées.
