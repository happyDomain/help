---
data: 2023-02-09T08:03:25+01:00
title: Connexion à un knot local
weight: 31
---

[Knot](https://knot-dns.cz) est un serveur DNS faisant autorité développé par l'association [cz.nic](https://nic.cz).

Il est possible de l'utiliser avec happyDomain en passant par le [Dynamic DNS (RFC 2136)](https://www.rfc-editor.org/rfc/rfc2136).


## Configurer Knot pour permettre le Dynamic DNS

Tout d'abord, il faut éditer le fichier de configuration principal de knot (généralement `/etc/knot/knot.conf`) afin d'ajouter un secret qui sera partagé entre happyDomain et knot pour authentifier les modifications. Puis il faudra indiquer quels domaines vont être gérés par happyDomain.

### Ajout d'un secret partagé {#shared-secret}

Sous la section principale [`key`](https://knot.readthedocs.io/en/latest/reference.html#key-section) de votre configuration, ajoutez la clef suivante :

```yaml
key:
  [...]
  - id: happydomain
    algorithm: hmac-sha512
    secret: "<SOME_SECRET>"
```

Remplacez évidemment `<SOME_SECRET>` par une chaîne de caractères telle qu'obtenue avec `openssl rand -base64 48`.


### Création d'une autorisation pour happyDomain

En plus de la clef, vous devez préciser dans la configuration comment la clef peut être utilisée.

Pour cela, sous la section principale [`acl`](https://knot.readthedocs.io/en/latest/reference.html#acl-section), on ajoute :

```yaml
acl:
  [...]
  - id: acl_happydomain
    key: happydomain
    action: transfer
    action: update
```

Cela associe la `key` définie juste avant aux actions `transfer` et `update`, respectivement pour permettre de récupérer la zone et pour mettre à jour des enregistrements.


### Associer l'autorisation à chaque zone

Maintenant que vous avez créé une règle autorisant la clef `happydomain` à apporter des modifications, il faut indiquer à quelles zones cette règle s'applique.

Pour chaque [zone](https://knot.readthedocs.io/en/latest/reference.html#zone-section), il faut ajouter un élément [`acl`](https://knot.readthedocs.io/en/latest/reference.html#acl) référençant la règle `acl_happydomain` :

Par exemple, pour une zone `happydomain.org` déjà existante, on ajoutera la ligne `acl` comme suit :

```yaml
zone:
  [...]
  - domain: happydomain.org
    acl:
      - acl_happydomain
    [...]
```

L'élément `acl` est une liste, vous pouvez donc avoir déjà d'autres éléments `acl` dans cette liste. Il convient alors d'ajouter simplement l'élément `acl_happydomain` à la liste déjà existante.

Il faut ajouter cet élément `acl` pour chaque zone, sauf à utiliser l'astuce suivante.


### Associer l'autorisation à toutes les zones

Si vous gérez de nombreuses zones, il peut être plus pratique de définir l'autorisation par défaut pour toutes les zones. Dans ce cas, à la place de la section précédente, on modifiera le modèle `default` :

```yaml
template:
  - id: default
    acl:
      - acl_happydomain
  [...]
```

Le modèle `default` est appliqué par défaut à toutes les zones. En faisant ainsi, toutes les zones hériteront de la règle `acl_happydomain`.


### Appliquer la configuration

Maintenant que le fichier de configuration a été modifié, indiquez à `knotd` qu'il doit recharger sa configuration :

```sh
knotc reload
```


## Liaison entre happyDomain et knot

Une fois `knot` bien configuré, vous pouvez le relier à happyDomain en utilisant [le connecteur *Dynamic DNS*]({{% ref "/pages/source-new-choice.md" %}}) :

![Le connecteur Dynamic DNS sur la page de choix de l'hébergeur](/img/choose-dynamic-dns.png)

Remplissez ensuite le formulaire avec l'adresse à laquelle votre serveur `knot` est accessible, puis ensuite les différents champs *Key* avec [les informations de la section `key` de `knot`](#shared-secret) :

- **Key Name** : correspond au champ `id` ;
- **Key Algorithm** : correspond au champ `algorithm` ;
- **Secret Key** : correspond au champ `secret`.

Une fois le fournisseur ajouté, il ne permet pas de lister les domaines existants, mais vous pouvez tout de même ajouter manuellement tous vos domaines.
