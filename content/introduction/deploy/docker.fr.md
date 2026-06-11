---
data: 2023-01-19T19:31:08+02:00
title: Avec Docker
weight: 15
---

happyDomain est sponsorisé par Docker.
Vous trouverez notre image officielle sur [le Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

L'image exécute happyDomain en tant que processus unique avec une base de données LevelDB stockée sur le disque — aucune base de données supplémentaire à configurer.


## Versions, étiquettes et architectures supportées

Toutes les étiquettes (*tags*) sont construites pour `amd64`, `arm64` et `arm/v7` et sont basées sur Alpine.

Les étiquettes actuellement disponibles :

- `latest` : la version la plus récente, correspondant à la branche `master` de notre dépôt.


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


## Déploiement complet avec tous les vérificateurs

happyDomain intègre tous les vérificateurs (*checkers*) en natif, mais certains
d'entre eux s'appuient sur des outils externes (DNSViz, Zonemaster, Matrix
federation tester) distribués dans leurs propres images Docker. Les faire
tourner comme des services séparés vous donne l'expérience complète des
vérificateurs et une meilleure isolation.

L'approche recommandée est `docker compose`. Enregistrez le fichier suivant
sous le nom `docker-compose.yml` et lancez `docker compose up -d`.

```yaml
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

    restart: unless-stopped
    volumes:
      - storage:/var/lib/happydomain:rw

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

volumes:
  storage:
```

### Comment ça fonctionne

Chaque vérificateur tourne comme un service HTTP autonome. happyDomain lui délègue
les demandes de vérification via la variable d'environnement
`HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT`. Si aucun point d'accès n'est configuré, le
vérificateur correspondant s'exécute localement dans le processus happyDomain.

Deux vérificateurs s'appuient sur des services tiers supplémentaires :

- **Zonemaster** (`checker-zonemaster`) interroge le service `zonemaster/backend`.
  La variable `HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL` indique au
  vérificateur l'adresse de ce service.
- **Matrix federation tester** (`checker-matrix`) interroge le service
  `matrixdotorg/federation-tester-backend`. La variable
  `HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER` pointe vers son point
  d'accès de rapport.

### Optionnel : happyDeliver

Si vous exploitez une instance [happyDeliver](https://happydeliver.io) pour
surveiller les flux e-mail, décommentez la ligne
`HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT` et ajoutez le service correspondant :

```yaml
  checker-happydeliver:
    image: happydomain/checker-happydeliver
    restart: unless-stopped
```

### Optionnel : clés API pour checker-blacklist

Le service `checker-blacklist` fonctionne sans clé API (il utilise des listes
de blocage DNS par défaut), mais vous pouvez activer des sources supplémentaires
— Google Safe Browsing, VirusTotal, abuse.ch URLhaus — en configurant les
options d'administration correspondantes depuis l'interface d'administration de
happyDomain une fois la pile démarrée.


## Interface d'administration

happyDomain expose des commandes d'administration à travers un socket Unix.
Le conteneur inclut l'utilitaire `hadmin` :

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

`hadmin` est une surcouche légère autour de `curl` — commencez par le chemin
d'URL, puis ajoutez les options `curl` après.


## Utilisation d'un fichier de configuration

Plutôt que des variables d'environnement, vous pouvez placer un fichier de
configuration dans `/data/happydomain.conf` (dans le volume de données) ou le
monter directement sur `/etc/happydomain.conf` :

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```
