---
data: 2023-01-19T18:28:25+02:00
title: Environnement de test ou unipersonnel local
weight: 10
---

Suivez ce guide d'installation si vous souhaitez utiliser happyDomain directement sur votre machine, le plus simplement possible, sans disposer de la gestion d'utilisateurs.

C'est une configuration simple, qui est appropriée pour évaluer happyDomain ou pour l'utiliser sans authentification sur sa propre machine, ou avec une authentification fournie par un reverse proxy.

Vous pouvez le déployer en utiliser Docker ou bien en téléchargeant l'un des exécutables mis à votre disposition. Nous verrons ci-après les deux méthodes.


## Via Docker/podman

Si vous être familier de Docker (ou `podman`), vous pouvez disposer d'happyDomain en quelques secondes avec la ligne de commande suivante :

```
docker container run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Si vous souhaitez rendre les données persistantes, il faut ajouter un volume :

```
docker volume create happydns_data
docker container run -v happydns_data:/data -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Quelque soit votre choix, rendez-vous ensuite sur <http://localhost:8081> pour commencer à utiliser happyDomain.


## Via l'exécutable

1. Commencez par télécharger l'exécutable correspondant à votre architecture de processeur sur : <https://get.happydomain.org/master/>.
  Les versions `darwin` sont pour MacOS, tandis que les versions `linux` sont pour GNU/Linux. Toutes les versions distribuées sont statiques et devraient fonctionner quelque soit votre libc (GNU libc la plupart du temps, musl pour Alpine, ...).

1. Rendez le binaire que vous avez téléchargé exécutable : `chmod +x happydomain-OS-ARCH`.

1. Exécutez le binaire : `HAPPYDOMAIN_NO_AUTH=1 ./happydomain-OS-ARCH`.

1. Rendez-vous sur <http://localhost:8081/> pour commencer à utiliser happyDomain.


## Que font ces options ?

L'option `HAPPYDOMAIN_NO_AUTH=1` est un paramètre qui indique à happyDomain qu'il ne doit pas attendre d'authentification : les domaines sont partagées entre toutes les connexions qui arrivent. En fait, cela va automatiquement créer un utilisateur par défaut et désactiver toutes les fonctionnalités de connexion, d'enregistrement de compte, ...


## Peut-on le mettre sur Internet comme ça ?

Non ! Il est primordial que vous n'exposiez pas votre happyDomain sur Internet sans authentification.
En effet, tout son contenu serait accessible à n'importe qui, et pourrait prendre possession de votre/vos domaines.

Utilisez systématiquement un reverse-proxy tel que nginx, Apache, HAproxy, Treafik, ... en ajoutant une étape de filtrage ou d'authentification basique.

Par exemple, vous pouvez filtrer les IP qui peuvent accéder au service.
Voici un exemple de configuration pour nginx, filtrant les IP (directive [`allow`](http://nginx.org/en/docs/http/ngx_http_access_module.html)) :

```
server {
    listen 80;
    listen [::]:80;

    server_name happydomain.example.com;

    location / {
        allow 42.42.42.42;
        deny all;

        proxy_pass	http://localhost:8081;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

}

```

Vous pouvez aussi ajouter une authentification à base de [mot de passe simple](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html).
