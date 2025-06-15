---
data: 2023-01-19T19:31:08+02:00
title: Avec Docker
weight: 15
---

happyDomain est sponsorisé Docker.
Vous trouverez notre image officielle sur [le Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

Cette image exécutera happyDomain en tant que processus unique, avec une base de données LevelDB (similaire à sqlite, LevelDB est une base de données stockée sur le disque, il n'est pas nécessaire de configurer quoi que ce soit d'autre).


## Versions, étiquettes and architectures supportés

Toutes les étiquettes (*tags*) sont construites pour les architectures de processeur les plus courantes (`amd64`, `arm64` et `arm/v7`).
Nous ne construisons des images uniquement basées sur la distribution Alpine Linux, ce qui assure des images de taille minimale.

Actuellement, les étiquettes disponibles sont :

- `latest`: il s'agit de la version la plus récente, correspondant à la branche `master` de notre dépôt de sources.


## Utilisation de l'image

### À des fins de test

Vous pouvez tester happyDomain ou l'utiliser pour votre usage personnel, avec l'option `HAPPYDOMAIN_NO_AUTH=1` : cela créera automatiquement un compte par défaut, et désactivera toutes les fonctionnalités liées à la gestion des utilisateurs (inscription, connexion, ...).

```
docker run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Les données sont stockées dans le répertoire `/data`.
Si vous souhaitez conserver vos paramètres d'une exécution à l'autre, vous devrez attacher ce répertoire à un volume géré par Docker ou à un répertoire sur votre hôte :

```
docker volume create happydomain_data
docker run -e HAPPYDOMAIN_NO_AUTH=1 -v happydomain_data:/data -p 8081:8081 happydomain/happydomain
```


### En production

happyDomain a besoin d'envoyer du courrier électronique, afin de vérifier les adresses et d'effectuer la récupération des mots de passe, vous devez donc configurer un relais SMTP.
Utilisez les options `HAPPYDOMAIN_MAIL_SMTP_HOST`, `HAPPYDOMAIN_MAIL_SMTP_PORT` (par défaut 25), `HAPPYDOMAIN_MAIL_SMTP_USERNAME` et `HAPPYDOMAIN_MAIL_SMTP_PASSWORD` à cette fin :

```
docker run -e HAPPYDOMAIN_MAIL_SMTP_HOST=smtp.yourcompany.com -e HAPPYDOMAIN_MAIL_SMTP_USERNAME=happydomain -e HAPPYDOMAIN_MAIL_SMTP_PASSWORD=secret -v /var/lib/happydomain:/data -p 8081:8081 happydomain/happydomain
```

Si vous préférez utiliser un fichier de configuration, vous pouvez le placer soit dans `/data/happydomain.conf` pour utiliser le volume, soit lier votre fichier à `/etc/happydomain.conf` :

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```


#### Étendre l'image de base

Par défaut, happyDomain utilise `sendmail`, si vous préférez, vous pouvez créer votre propre image avec le paquet `ssmtp` :

```
FROM happydomain/happydomain
RUN apk --no-cache add ssmtp
COPY my_ssmtp.conf /etc/ssmtp/ssmtp.conf
```


## Interface d'administration

happyDomain expose certaines commandes d'administration à travers un socket unix.
Le conteneur docker contient un script pour accéder à cette partie d'administration : `hadmin`.

Vous pouvez l'utiliser de cette manière :

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

Il s'agit en fait d'une surcouche au-dessus de `curl`, mais vous devez commencer par l'URL, et placer les options après.
