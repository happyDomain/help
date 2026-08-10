---
data: 2023-02-11T11:58:55+01:00
title: Configuration
weight: 20
---

happyDomain respecte la méthodologie [*12 factor*](https://12factor.net/) et permet notamment d'agir sur la configuration de l'application de plusieurs manières.

## Par quels moyens configurer happyDomain ?

Il est possible de configurer happyDomain de trois manières différentes : fichier de configuration, environnement, ligne de commande. Toutes les options sont disponibles pour chacun de ces mécanismes.

La précédence, lorsqu'une option est définie par plusieurs mécanismes simultanément, est qu'une option présente dans un fichier de configuration sera écrasé par l'environnement, qui sera écrasée par une option passée sur la ligne de commande.

### Configuration par fichier

Au lancement de l'application, le premier fichier de configuration parmi la liste suivante sera utilisé :

- `./happydomain.conf`
- `$XDG_CONFIG_HOME/happydomain/happydomain.conf`
- `/etc/happydomain.conf`

**Seulement le premier fichier existant est pris en compte.** Il n'est pas possible d'avoir une partie de ses options dans `/etc/happydomain.conf` et une autre dans `./happydomain.conf`, seul ce dernier fichier de configuration sera pris en compte.

Il est possible de préciser un chemin personnalisé en l'ajoutant comme paramètre supplémentaire à la ligne de commande. Ainsi, pour utiliser le fichier de configuration situé à `/etc/happydomain/config`, on utilisera :

```
./happydomain /etc/happydomain/config
```

#### Format du fichier de configuration

Une ligne de commentaires commence par `#`, il n'est pas possible d'avoir des commentaires à la fin d'une ligne, en ajoutant `#` suivi d'un commentaire.

Placez sur chaque ligne le nom de l'option de configuration et la valeur attendue, séparés par `=`. Par exemple :

```
storage-engine=leveldb
leveldb-path=/var/lib/happydomain/db/
```

### Configuration par l'environnement

Au lancement d'happyDomain, toutes les variables commençant par `HAPPYDOMAIN_` sont analysées à la recherche d'options de configuration valides.

Vous pouvez réaliser la même chose que dans l'exemple précédent, avec les variables d'environnement suivantes :

```
HAPPYDOMAIN_STORAGE_ENGINE=leveldb
HAPPYDOMAIN_LEVELDB_PATH=/var/lib/happydomain/db/
```

Notez que les `-` sont remplacés par des `_` dans les variables d'environnement.


### Configuration par la ligne de commande

Enfin, la ligne de commande peut être utilisée pour passer des options, selon le format UNIX usuel.

Pour continuer l'exemple précédent, nous pouvons réaliser la même configuration avec la ligne de commande suivante :

```
./happydomain -storage-engine leveldb -leveldb-path /var/lib/happydomain/db/
```

ou encore en utilisant le signe `=` pour assigner clairement la valeur.

```
./happydomain -storage-engine=leveldb -leveldb-path=/var/lib/happydomain/db/
```


## Éléments de configuration

La liste exhaustive des éléments configurables peut être listé en appelant `happyDomain` avec l'option `-h` ou `--help`.

Voici la liste des principales options :

### Paramètres généraux

`bind`
: Interface (ip, port) à utiliser pour exposer le service happyDomain.

`admin-bind`
: Interface (ip, port ou socket) à utiliser pour exposer l'API d'administration.

`default-ns`
: Adresse et port du serveur résolveur de noms à utiliser par défaut lorsqu'une résolution de nom est nécessaire.

`dev`
: URL vers laquelle toutes les requêtes liées à l'interface graphique seront renvoyées.

`externalurl`
: URL du service, tel qu'il doit apparaître dans les mails et contenus à destination du public.

`disable-providers-edit`
: Interdit toute action sur les fournisseurs de service DNS (ajour/édition/suppression), par exemple pour avoir un mode de démonstration.


#### Mise en page

`custom-head-html`
: Chaîne de caractères à placer avant la fin de l'en-tête HTML.

`custom-body-html`
: Chaîne de caractères à placer avant la fin du corps HTML.

`hide-feedback-button`
: Cache l'icône permettant de donner son retour d'expérience.

`msg-header-text`
: Ajoute un message personnalisé dans une bannière en haut de toutes les pages.

`msg-header-color`
: Classe de couleur de fond pour la bannière ajoutée en haut de l'application web (par défaut "danger", pourrait être primary, secondary, info, success, warning, danger, light, dark, ou toute autre classe de couleur bootstrap).


### Stockage des données

`storage-engine`
: Permet de choisir le mécanisme de stockage des données parmi tous les mécanismes supportés.

### LevelDB (`storage-engine=leveldb`)

`leveldb-path`
: Chemin vers le dossier contenant la base LevelDB à utiliser.

### Paramètres e-mail

Nous employons [`go-mail`](https://github.com/go-mail/mail) comme bibliothèque pour envoyer les mails.

`mail-from`
: Définit le nom et l'adresse de l'expéditeur des mails envoyés par le service. Accepte soit une adresse seule (`info@domain.com`), soit un nom d'affichage suivi de l'adresse entre chevrons (`Domain <info@domain.com>`). Par exemple, en ligne de commande : `-mail-from "Domain <info@domain.com>"` ; via l'environnement : `HAPPYDOMAIN_MAIL_FROM="Domain <info@domain.com>"`.

Notez que sans les options `mail-smtp-*`, happyDomain utilisera le binaire `sendmail` pour envoyer les mails. Cela peut être couplé aux paquets `msmtp` ou `ssmtp` par exemple, pour définir les paramètres pour tout le système.

`mail-smtp-host`
: IP ou nom d'hôte du serveur SMTP à utiliser.

`mail-smtp-port`
: Port à utiliser sur le serveur distant.

`mail-smtp-username`
: Lorsque de l'authentification est nécessaire sur le serveur distant, nom d'utilisateur à utiliser.

`mail-smtp-password`
: Lorsque de l'authentification est nécessaire sur le serveur distant, mot de passe à utiliser.

`no-mail`
: Désactive tous les e-mails automatiques, et saute l'étape de validation de l'adresse e-mail lors de l'inscription.


### Authentification

`no-auth`
: Désactive la notion d'utilisateurs et de contrôle d'accès. Un compte par défaut est utilisé.

`disable-embedded-login`
: Désactive le mécanisme de connexion interne en faveur de l'external-auth ou d'OIDC.

`disable-registration`
: Interdit la création de nouveau compte à travers le formulaire ou l'API (cela ne désactive pas la création de compte lorsque l'on se connecte pour la première fois à partir d'un service d'authentification externe).

`external-auth`
: Base de l'URL du service d'authentification et d'enregistrement à utiliser à la place du système de connexion embarqué.

`jwt-secret-key`
: Clef secrète utilisée pour vérifier les tokens JWT.

Voir aussi [paramètres OpenID Connect]({{% relref "oidc" %}}).

#### Comptes et données

Par défaut (connexion embarquée, sans `external-auth`), chaque compte est créé et stocké dans la base de données de **votre propre instance**, quel que soit le `storage-engine` que vous avez configuré ci-dessus. Rien n'est jamais envoyé vers happydomain.org : l'instance hébergée par le projet, `app.happydomain.org`, n'est qu'un déploiement parmi d'autres, totalement indépendant de toute instance que vous faites tourner vous-même.

Les trois options ci-dessus peuvent se combiner pour définir le comportement de l'inscription sur votre instance :

- Configuration par défaut : tout le monde peut s'inscrire librement, et (sauf si `no-mail` est activé) doit valider son adresse e-mail avant que le compte soit utilisable.
- `no-mail` : l'inscription reste ouverte, mais l'étape de validation par e-mail est sautée ; les comptes sont utilisables immédiatement. Pratique lorsqu'aucun relais SMTP n'est configuré.
- `disable-registration` : aucun nouveau compte ne peut être créé via le formulaire public ou l'API ; les comptes existants continuent de fonctionner, et des comptes peuvent toujours être créés via `external-auth` ou OIDC.
- `no-auth` : supprime toute notion de compte ; chaque visiteur partage un unique compte par défaut. Adapté aux installations locales/mono-utilisateur ou d'évaluation.


### Spécifique aux bureaux d'enregistrement

Certain bureau d'enregistrement nécessitent que les applications tierces s'identifient en plus d'identifier l'utilisateur.

#### Bind

`with-bind-provider`
: Active BIND en tant que fournisseur DNS, en le confinant au répertoire indiqué (attention, ce paramètre n'est pas adapté à un environnement partagé/cloud car il accède au système de fichiers local).

Cette option attend le chemin absolu d'un répertoire contenant vos fichiers de zone, ce n'est pas un simple interrupteur : il n'est pas possible d'activer le fournisseur sans déclarer au moins un répertoire.

Les utilisateurs de l'instance ne peuvent alors faire pointer leur connecteur *Bind files/RFC 1035* que vers ce répertoire ou vers l'un de ses sous-répertoires. Tout autre chemin est refusé, aussi bien à l'enregistrement du connecteur qu'à chaque listage, lecture ou écriture de zone : un connecteur enregistré avant l'existence de cette restriction est donc lui aussi neutralisé.

Les liens symboliques sont résolus avant la comparaison, si bien qu'un lien déposé dans un répertoire autorisé ne permet pas d'en sortir. Les répertoires autorisés sont supposés appartenir à l'administrateur de l'instance, et non être accessibles en écriture à ses utilisateurs.

Répétez l'option sur la ligne de commande pour autoriser plusieurs répertoires :

```
./happydomain -with-bind-provider /etc/named/zones -with-bind-provider /var/named
```

Dans l'environnement ou dans un fichier de configuration, où une option ne peut apparaître qu'une seule fois, séparez les répertoires avec le séparateur de liste de chemins (`:` sous UNIX) :

```
HAPPYDOMAIN_WITH_BIND_PROVIDER=/etc/named/zones:/var/named
```


#### OVH

Veuillez vous référer à [cette documentation]({{% relref "ovh" %}}) afin de générer les identifiants.

`ovh-application-key`
: Application key pour l'API d'OVH

`ovh-application-secret`
: Clef secrète pour l'API d'OVH
