---
data: 2023-01-19T19:31:08+02:00
title: Avec Docker
weight: 15
---

happyDomain est sponsorisé par Docker.
Vous trouverez notre image officielle sur [le Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

L'image exécute happyDomain en tant que processus unique avec une base de données LevelDB stockée sur le disque : aucune base de données supplémentaire à configurer. **Tous les vérificateurs (*checkers*) livrés avec happyDomain sont intégrés au binaire**, un conteneur unique constitue donc déjà un happyDomain complet.


## Versions, étiquettes et architectures supportées

Toutes les étiquettes (*tags*) sont construites pour `amd64`, `arm64` et `arm/v7` et sont basées sur Alpine.

Les étiquettes actuellement disponibles :

- `latest` : la version la plus récente, correspondant à la branche `master` de notre dépôt.

Les étiquettes suffixées par **`-cgo`** contiennent une version liée
dynamiquement, la seule capable de charger des
[greffons]({{% relref "/reference/plugins" %}}). Ne les utilisez que si vous en
avez besoin : les étiquettes par défaut sont compilées statiquement, plus
légères et plus portables, mais elles ignorent tout répertoire de greffons.


## Démarrage rapide (conteneur unique)

Pour un test rapide ou un usage personnel, utilisez `HAPPYDOMAIN_NO_AUTH=1` pour désactiver la gestion des comptes :

```
docker run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Les données sont stockées à l'intérieur du conteneur. Pour les conserver entre les redémarrages, attachez un volume :

```
docker volume create happydomain_data
docker run -e HAPPYDOMAIN_NO_AUTH=1 -v happydomain_data:/data -p 8081:8081 happydomain/happydomain
```

Pour une instance de production avec envoi d'e-mails :

```
docker run \
  -e HAPPYDOMAIN_MAIL_SMTP_HOST=smtp.votreentreprise.com \
  -e HAPPYDOMAIN_MAIL_SMTP_USERNAME=happydomain \
  -e HAPPYDOMAIN_MAIL_SMTP_PASSWORD=secret \
  -v /var/lib/happydomain:/data \
  -p 8081:8081 \
  happydomain/happydomain
```


## Déploiement recommandé : `docker compose`

C'est la configuration que nous recommandons pour **la quasi-totalité des
déploiements**, de l'instance personnelle à l'instance d'entreprise. Il s'agit
du fichier que vous trouverez à la racine du
[dépôt happyDomain](https://github.com/happyDomain/happydomain/blob/master/docker-compose.yml).

En plus d'happyDomain, il fait tourner les quelques services tiers que certains
vérificateurs doivent contacter (DNSViz, Zonemaster et le testeur de fédération
Matrix), ainsi qu'un résolveur récursif local afin que les résolutions DNS ne
dépendent pas du résolveur de votre hébergeur.

Enregistrez le fichier sous le nom `docker-compose.yml` et lancez
`docker compose up -d`.

```yaml
services:
  happydomain:
    image: happydomain/happydomain
    ports:
      - "8081:8081"
    environment:
      # Décommentez pour un usage mono-utilisateur / test
      # HAPPYDOMAIN_NO_AUTH: "1"

      # Configuration mail (obligatoire en production multi-utilisateurs)
      # HAPPYDOMAIN_MAIL_SMTP_HOST: "mailer"

      # Utilise les services locaux plutôt que les services publics
      HAPPYDOMAIN_CHECKER_DNSVIZ_ENDPOINT: "http://dnsviz:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL: "http://zonemaster:5000"
      HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER: "http://matrixfederationtester:8080/api/report?server_name=%s"

    dns:
      - 172.28.0.53
    restart: unless-stopped
    volumes:
      - storage:/data:rw

  # Résolveur récursif local, pour ne pas dépendre d'un résolveur tiers
  unbound:
    image: alpinelinux/unbound
    restart: unless-stopped
    configs:
      - source: unbound_conf
        target: /etc/unbound/unbound.conf
        uid: "100"
        gid: "101"
    networks:
      default:
        ipv4_address: 172.28.0.53

  # Moteur d'analyse DNSViz, utilisé par le vérificateur de visualisation DNSSEC
  dnsviz:
    image: happydomain/checker-dnsviz
    restart: unless-stopped

  # Service Zonemaster, utilisé par le vérificateur Zonemaster
  zonemaster:
    image: zonemaster/backend
    command: full
    restart: unless-stopped

  # Testeur de fédération Matrix, utilisé par le vérificateur Matrix
  matrixfederationtester:
    image: matrixdotorg/federation-tester-backend
    environment:
      BIND_ADDRESS: "0.0.0.0:8080"
    restart: unless-stopped

configs:
  unbound_conf:
    content: |
      server:
          verbosity: 1
          interface: 0.0.0.0
          port: 53
          do-ip4: yes
          do-ip6: no
          do-udp: yes
          do-tcp: yes

          access-control: 127.0.0.0/8 allow
          access-control: 172.28.0.0/24 allow

          cache-max-ttl: 60

          so-sndbuf: 0
          so-rcvbuf: 0

          trust-anchor-file: "/etc/unbound/root.key"

volumes:
  storage:

networks:
  default:
    ipam:
      config:
        - subnet: 172.28.0.0/24
```

Avec cette pile, **tous les vérificateurs livrés avec happyDomain sont
disponibles** : ils s'exécutent dans le processus happyDomain, et les trois
services ci-dessus ne fournissent que les moteurs d'analyse externes dont deux
ou trois d'entre eux ont besoin.


## Où s'exécutent les vérificateurs ?

Il y a ici deux questions distinctes, et les confondre est une source classique
de malentendus.

**D'abord, comment happyDomain sait-il qu'un vérificateur existe ?** Seulement
deux réponses :

- **intégré** (*built-in*) : le vérificateur est compilé dans le binaire
  happyDomain. Tous les vérificateurs que nous distribuons sont disponibles
  ainsi, c'est le fonctionnement des déploiements ci-dessus ;
- **greffon** (*plugin*) : une bibliothèque partagée (`.so`) déposée dans un
  répertoire indiqué par `-plugins-directory` et chargée au démarrage. C'est le
  seul moyen d'ajouter un vérificateur qui ne fait *pas* partie d'happyDomain,
  écrit pour vos propres besoins ou fourni par un tiers, sans avoir à modifier
  puis recompiler le serveur. Cela nécessite une **étiquette d'image `-cgo`** :
  l'image par défaut est une version statique, sans prise en charge des
  greffons (voir [Greffons]({{% relref "/reference/plugins" %}})).

**Ensuite, où le travail s'exécute-t-il réellement ?** Par défaut, un
vérificateur enregistré s'exécute dans le processus happyDomain. Si vous
définissez `HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT`, il délègue à la place la collecte
à un **conteneur autonome** qui expose ce vérificateur en HTTP.

La conséquence importante : un conteneur de vérificateur n'est *pas* un moyen
d'ajouter un vérificateur. Le réglage `endpoint` n'existe que pour les
vérificateurs qu'happyDomain connaît déjà ; un vérificateur maison doit donc
d'abord être chargé sous forme de greffon, et seulement ensuite vous pourrez le
faire pointer vers un conteneur.

Pour un vérificateur donné, l'exécuter localement ou le déléguer à un conteneur
effectue exactement les mêmes vérifications et produit les mêmes résultats. La
délégation vous apporte l'isolation des processus et la possibilité de mettre à
l'échelle un vérificateur indépendamment des autres ; elle vous coûte beaucoup
plus de mémoire vive, davantage de pièces mobiles et une expérience de débogage
nettement moins agréable lorsque l'un d'eux se comporte mal.

À moins d'exploiter une instance à une échelle où ce compromis est rentable, ou
d'avoir besoin d'un vérificateur qui n'est pas livré avec happyDomain, restez
sur le fichier `docker compose` ci-dessus. Si vous en avez réellement besoin,
consultez
[Exécuter les vérificateurs dans des conteneurs séparés]({{% relref "checkers-containers" %}}).


## Mettre à jour la pile

**1. Vérifiez si le `docker-compose.yml` de référence a changé.** Mettre à jour
les images ne suffit pas toujours : une nouvelle version peut s'accompagner
d'un nouveau service (un vérificateur qui a besoin de son propre moteur
d'analyse, par exemple) ou de nouveaux paramètres à déclarer. Comparez votre
fichier avec celui du
[dépôt happyDomain](https://github.com/happyDomain/happydomain/blob/master/docker-compose.yml),
et reportez dans votre copie les changements qui vous concernent :

```
curl -sO https://raw.githubusercontent.com/happyDomain/happydomain/master/docker-compose.yml
diff -u docker-compose.yml /chemin/vers/votre/docker-compose.yml
```

**2. Récupérez les images et recréez les conteneurs.**

```
docker compose up -d --pull always
```

`--pull always` récupère la dernière image de chaque service avant de recréer
uniquement ceux qui ont réellement changé ; les autres ne sont pas touchés. Vos
données se trouvent dans le volume `storage` et survivent à l'opération.

Vérifiez ensuite que tout est bien reparti, puis récupérez l'espace disque
occupé par les images remplacées :

```
docker compose ps
docker compose logs -f happydomain
docker image prune
```

Pour un conteneur unique lancé avec `docker run`, l'équivalent consiste à
récupérer l'image, supprimer l'ancien conteneur et le relancer avec les mêmes
options (votre volume conserve les données) :

```
docker pull happydomain/happydomain
docker rm -f my_container
```


## Interface d'administration

happyDomain expose des commandes d'administration à travers un socket Unix.
Le conteneur inclut l'utilitaire `hadmin` :

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

`hadmin` est une surcouche légère autour de `curl` : commencez par le chemin
d'URL, puis ajoutez les options `curl` après.


## Utilisation d'un fichier de configuration

Plutôt que des variables d'environnement, vous pouvez placer un fichier de
configuration dans `/data/happydomain.conf` (dans le volume de données) ou le
monter directement sur `/etc/happydomain.conf` :

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```
