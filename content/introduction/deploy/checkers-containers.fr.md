---
data: 2026-08-01T10:00:00+02:00
title: Exécuter les vérificateurs dans des conteneurs séparés
weight: 17
---

{{% notice style="warning" title="Ce n'est pas le déploiement par défaut" %}}
Vous n'avez très probablement pas besoin de cette page. Le fichier
`docker compose` décrit dans [Avec Docker]({{% relref "docker" %}}) vous donne
déjà **tous les vérificateurs que nous distribuons**, avec exactement les mêmes
résultats. Les éclater en conteneurs n'ajoute aucune fonctionnalité : cela
change seulement l'endroit où ils s'exécutent. (Ajouter un vérificateur qui ne
fait *pas* partie d'happyDomain est une autre question ; la réponse est
généralement un [greffon]({{% relref "/reference/plugins" %}}).)
{{% /notice %}}


## À quoi ça sert

happyDomain doit **connaître** un vérificateur avant de pouvoir l'exécuter. Il
n'existe que deux façons de l'enregistrer :

- **intégré** (*built-in*), compilé dans le binaire happyDomain. Tous les
  vérificateurs que nous distribuons sont disponibles ainsi, et c'est le
  comportement par défaut ;
- **greffon** (*plugin*), une bibliothèque partagée chargée au démarrage depuis
  un répertoire indiqué par `-plugins-directory`. C'est le seul moyen
  d'enregistrer un vérificateur qui ne fait pas partie d'happyDomain, par
  exemple développé en interne. Cela nécessite une version `-cgo` d'happyDomain
  et une plateforme où Go prend en charge les greffons.

Une fois le vérificateur enregistré, vous décidez **où son travail s'exécute** :
dans le processus happyDomain (par défaut), ou dans un **conteneur autonome**
auquel il délègue en HTTP. C'est l'objet de cette page.

{{% notice style="note" title="Un conteneur n'enregistre pas un vérificateur" %}}
Faire tourner `happydomain/checker-<quelque-chose>` et définir son point d'accès
ne fonctionne que parce que ce vérificateur est déjà connu d'happyDomain.
Pointer un point d'accès vers un vérificateur dont le serveur n'a jamais
entendu parler ne produit rien : un vérificateur maison doit d'abord être
chargé sous forme de greffon, et seulement ensuite il pourra être délégué à un
conteneur.
{{% /notice %}}

Prenons [`checker-ping`](https://framagit.org/happyDomain/checker-ping), qui
vérifie que chaque adresse IP d'une zone répond au ping dans un délai donné. Il
est déjà présent dans votre binaire happyDomain, quel que soit le déploiement
choisi. Si vous faites également tourner le conteneur
`happydomain/checker-ping` et définissez `HAPPYDOMAIN_CHECKER_PING_ENDPOINT`, le
vérificateur intégré se contente de transmettre le travail à ce conteneur au
lieu de l'effectuer lui-même. Mêmes vérifications, mêmes règles, mêmes
résultats.


## Quand cela vaut le coup

Le déploiement éclaté prend son sens lorsque :

- vous surveillez **des milliers de zones** et souhaitez mettre à l'échelle
  horizontalement un vérificateur très sollicité, indépendamment du reste
  d'happyDomain ;
- vous avez besoin d'une **forte isolation des processus**, par exemple parce
  qu'un vérificateur requiert des capacités réseau élevées (`NET_RAW` pour
  l'ICMP) que vous ne voulez pas accorder au processus principal ;
- vous voulez **figer, mettre à jour ou revenir en arrière sur un seul
  vérificateur**, sur son propre rythme de publication.


## Ce que cela coûte

- **La mémoire.** Une trentaine de conteneurs supplémentaires, chacun avec son
  propre environnement d'exécution, pour un ensemble de fonctionnalités
  strictement identique à celui du conteneur unique.
- **L'exploitation.** Une trentaine d'images de plus à récupérer, surveiller et
  mettre à jour.
- **Le débogage.** Quand une vérification échoue, la cause peut désormais être
  le vérificateur, le conteneur, le réseau entre les deux, ou une variable
  d'environnement obsolète. Cette investigation est nettement plus pénible que
  la lecture d'un unique flux de journaux.

Si aucune des raisons ci-dessus ne vous concerne, revenez à
[Avec Docker]({{% relref "docker" %}}).


## Comment fonctionne la délégation

`HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT` est une option **d'un vérificateur
enregistré**, et non un moyen d'en déclarer un. Pour chaque vérificateur
qu'happyDomain connaît (intégré ou chargé depuis un greffon), définir cette
variable lui fait transmettre la collecte à l'URL indiquée au lieu de faire le
travail lui-même ; la laisser vide maintient le vérificateur dans le processus
happyDomain. `<ID>` est l'identifiant propre du vérificateur : s'il ne
correspond à aucun vérificateur enregistré, la variable est simplement ignorée.

Les deux approches se mélangent librement : ne déléguez que les vérificateurs
que vous souhaitez réellement isoler et laissez les autres s'exécuter dans le
processus.

Indépendamment de ce choix, deux vérificateurs s'appuient sur des services tiers
supplémentaires :

- **Zonemaster** (`checker-zonemaster`) interroge le service `zonemaster/backend`.
  La variable `HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL` indique au
  vérificateur l'adresse de ce service.
- **Matrix federation tester** (`checker-matrix`) interroge le service
  `matrixdotorg/federation-tester-backend`. La variable
  `HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER` pointe vers son point
  d'accès de rapport.


## Fichier `docker compose` complet

```yaml
name: happydomain

services:
  happydomain:
    image: happydomain/happydomain
    ports:
      - "8080:8081"
    environment:
      # Décommentez pour un usage mono-utilisateur / test
      # HAPPYDOMAIN_NO_AUTH: "1"

      # Configuration mail (obligatoire en production multi-utilisateurs)
      # HAPPYDOMAIN_MAIL_SMTP_HOST: "mailer"

      # ── DNS / DNSSEC ─────────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_DNSVIZ_ENDPOINT: "http://checker-dnsviz:8080"
      HAPPYDOMAIN_CHECKER_DNSSEC_ENDPOINT: "http://checker-dnssec:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ENDPOINT: "http://checker-zonemaster:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL: "http://zonemaster:5000"
      HAPPYDOMAIN_CHECKER_DELEGATION_ENDPOINT: "http://checker-delegation:8080"
      HAPPYDOMAIN_CHECKER_AUTHORITATIVE_CONSISTENCY_ENDPOINT: "http://checker-authoritative-consistency:8080"
      HAPPYDOMAIN_CHECKER_ALIAS_ENDPOINT: "http://checker-alias:8080"
      HAPPYDOMAIN_CHECKER_LEGACY_RECORDS_ENDPOINT: "http://checker-legacy-records:8080"
      HAPPYDOMAIN_CHECKER_NS_RESTRICTIONS_ENDPOINT: "http://checker-ns-restrictions:8080"
      HAPPYDOMAIN_CHECKER_RESOLVER_PROPAGATION_ENDPOINT: "http://checker-resolver-propagation:8080"
      HAPPYDOMAIN_CHECKER_REVERSE_ZONE_ENDPOINT: "http://checker-reverse-zone:8080"
      HAPPYDOMAIN_CHECKER_PTR_ENDPOINT: "http://checker-ptr:8080"
      HAPPYDOMAIN_CHECKER_DANGLING_ENDPOINT: "http://checker-dangling:8080"

      # ── Sécurité / Certificats ────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_TLS_ENDPOINT: "http://checker-tls:8080"
      HAPPYDOMAIN_CHECKER_DANE_ENDPOINT: "http://checker-dane:8080"
      HAPPYDOMAIN_CHECKER_CAA_ENDPOINT: "http://checker-caa:8080"
      HAPPYDOMAIN_CHECKER_BLACKLIST_ENDPOINT: "http://checker-blacklist:8080"

      # ── E-mail ────────────────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_SMTP_ENDPOINT: "http://checker-smtp:8080"
      HAPPYDOMAIN_CHECKER_EMAIL_AUTOCONFIG_ENDPOINT: "http://checker-email-autoconfig:8080"
      HAPPYDOMAIN_CHECKER_OPENPGPKEY_SMIMEA_ENDPOINT: "http://checker-email-keys:8080"

      # ── Web & Protocoles ──────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_HTTP_ENDPOINT: "http://checker-http:8080"
      HAPPYDOMAIN_CHECKER_SSH_ENDPOINT: "http://checker-ssh:8080"
      HAPPYDOMAIN_CHECKER_PING_ENDPOINT: "http://checker-ping:8080"
      HAPPYDOMAIN_CHECKER_SRV_ENDPOINT: "http://checker-srv:8080"

      # ── Collaboration / Messagerie ────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_MATRIXIM_ENDPOINT: "http://checker-matrix:8080"
      HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER: "http://matrixfederationtester:8080/api/report?server_name=%s"
      HAPPYDOMAIN_CHECKER_XMPP_ENDPOINT: "http://checker-xmpp:8080"
      HAPPYDOMAIN_CHECKER_SIP_ENDPOINT: "http://checker-sip:8080"

      # ── Annuaire & Authentification ───────────────────────────────────────
      HAPPYDOMAIN_CHECKER_LDAP_ENDPOINT: "http://checker-ldap:8080"
      HAPPYDOMAIN_CHECKER_KERBEROS_ENDPOINT: "http://checker-kerberos:8080"
      HAPPYDOMAIN_CHECKER_STUNTURN_ENDPOINT: "http://checker-stun-turn:8080"

      # ── CalDAV / CardDAV ──────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_CALDAV_ENDPOINT: "http://checker-caldav:8080"
      HAPPYDOMAIN_CHECKER_CARDDAV_ENDPOINT: "http://checker-carddav:8080"

      # ── Optionnel : intégration happyDeliver ──────────────────────────────
      # HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT: "http://checker-happydeliver:8080"

    dns:
      - ${HAPPYDOMAIN_DNS_IP:-172.28.0.53}
    restart: unless-stopped
    volumes:
      - storage:/var/lib/happydomain:rw

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
        ipv4_address: ${HAPPYDOMAIN_DNS_IP:-172.28.0.53}

  # ── Vérificateurs DNS / DNSSEC ─────────────────────────────────────────────

  checker-dnsviz:
    image: happydomain/checker-dnsviz
    restart: unless-stopped

  checker-dnssec:
    image: happydomain/checker-dnssec
    restart: unless-stopped

  checker-zonemaster:
    image: happydomain/checker-zonemaster
    restart: unless-stopped

  zonemaster:
    image: zonemaster/backend
    command: full
    restart: unless-stopped

  checker-delegation:
    image: happydomain/checker-delegation
    restart: unless-stopped

  checker-authoritative-consistency:
    image: happydomain/checker-authoritative-consistency
    restart: unless-stopped

  checker-alias:
    image: happydomain/checker-alias
    restart: unless-stopped

  checker-legacy-records:
    image: happydomain/checker-legacy-records
    restart: unless-stopped

  checker-ns-restrictions:
    image: happydomain/checker-ns-restrictions
    restart: unless-stopped

  checker-resolver-propagation:
    image: happydomain/checker-resolver-propagation
    restart: unless-stopped

  checker-reverse-zone:
    image: happydomain/checker-reverse-zone
    restart: unless-stopped

  checker-ptr:
    image: happydomain/checker-ptr
    restart: unless-stopped

  checker-dangling:
    image: happydomain/checker-dangling
    restart: unless-stopped

  # ── Vérificateurs Sécurité / Certificats ───────────────────────────────────

  checker-tls:
    image: happydomain/checker-tls
    restart: unless-stopped

  checker-dane:
    image: happydomain/checker-dane
    restart: unless-stopped

  checker-caa:
    image: happydomain/checker-caa
    restart: unless-stopped

  checker-blacklist:
    image: happydomain/checker-blacklist
    restart: unless-stopped

  # ── Vérificateurs e-mail ────────────────────────────────────────────────────

  checker-smtp:
    image: happydomain/checker-smtp
    restart: unless-stopped

  checker-email-autoconfig:
    image: happydomain/checker-email-autoconfig
    restart: unless-stopped

  checker-email-keys:
    image: happydomain/checker-email-keys
    restart: unless-stopped

  # ── Vérificateurs Web & Protocoles ─────────────────────────────────────────

  checker-http:
    image: happydomain/checker-http
    restart: unless-stopped

  checker-ssh:
    image: happydomain/checker-ssh
    restart: unless-stopped

  checker-ping:
    image: happydomain/checker-ping
    restart: unless-stopped
    cap_add:
      - NET_RAW  # requis pour l'ICMP

  checker-srv:
    image: happydomain/checker-srv
    restart: unless-stopped

  # ── Vérificateurs Collaboration / Messagerie ────────────────────────────────

  checker-matrix:
    image: happydomain/checker-matrix
    restart: unless-stopped

  matrixfederationtester:
    image: matrixdotorg/federation-tester-backend
    environment:
      BIND_ADDRESS: "0.0.0.0:8080"
    restart: unless-stopped

  checker-xmpp:
    image: happydomain/checker-xmpp
    restart: unless-stopped

  checker-sip:
    image: happydomain/checker-sip
    restart: unless-stopped

  # ── Vérificateurs Annuaire & Authentification ───────────────────────────────

  checker-ldap:
    image: happydomain/checker-ldap
    restart: unless-stopped

  checker-kerberos:
    image: happydomain/checker-kerberos
    restart: unless-stopped

  checker-stun-turn:
    image: happydomain/checker-stun-turn
    restart: unless-stopped

  # ── Vérificateurs CalDAV / CardDAV ─────────────────────────────────────────

  checker-caldav:
    image: happydomain/checker-caldav
    restart: unless-stopped

  checker-carddav:
    image: happydomain/checker-carddav
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
          access-control: ${HAPPYDOMAIN_SUBNET:-172.28.0.0/24} allow

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
        - subnet: ${HAPPYDOMAIN_SUBNET:-172.28.0.0/24}
```


## Mettre à jour la pile

La procédure est la même que pour le déploiement standard
([Mettre à jour la pile]({{% relref "docker#mettre-à-jour-la-pile" %}})) :
vérifiez d'abord si le fichier dont vous êtes parti a changé, puis

```
docker compose up -d --pull always
```

Attention, ici la première étape vous incombe : ce fichier n'est pas généré.
Lorsque nous publions un **nouveau vérificateur**, sa version intégrée arrive
avec l'image happyDomain, mais rien n'ajoute pour autant le conteneur
correspondant ni sa variable `_ENDPOINT` à votre fichier. Relisez cette page
après chaque mise à jour et reportez-y les nouveaux services que vous souhaitez
continuer à déléguer ; un vérificateur que vous ne listez pas continue
simplement de s'exécuter dans le processus, ce qui reste parfaitement valable.


## Optionnel : happyDeliver

Si vous exploitez une instance [happyDeliver](https://happydeliver.io) pour
surveiller les flux e-mail, décommentez la ligne
`HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT` et ajoutez le service correspondant :

```yaml
  checker-happydeliver:
    image: happydomain/checker-happydeliver
    restart: unless-stopped
```


## Optionnel : clés API pour checker-blacklist

Le service `checker-blacklist` fonctionne sans clé API (il utilise des listes
de blocage DNS par défaut), mais vous pouvez activer des sources supplémentaires
(Google Safe Browsing, VirusTotal, abuse.ch URLhaus) en configurant les options
d'administration correspondantes depuis l'interface d'administration de
happyDomain une fois la pile démarrée.
