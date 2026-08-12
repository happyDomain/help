---
date: 2026-08-12T10:00:00+02:00
title: Signaler un problème
author: nemunaire
weight: 30
---

**Dites-le nous en une phrase, cela suffit.** Nous reviendrons vers vous si nous avons besoin de plus.

Étant donné la diversité des configurations DNS et des hébergeurs, nous ne pouvons pas tester toutes les situations possibles : vos signalements sont ce qui permet à happyDomain de s'améliorer pour tout le monde.
Ne passez pas votre soirée à enquêter avant de nous écrire, et n'ayez pas peur de signaler quelque chose qui s'avérerait ne pas être un bug : nous préférons de loin lire un signalement trop court que ne jamais entendre parler du problème.

Où signaler :

- depuis happyDomain lui-même : **Signaler un problème**, dans le menu utilisateur, ou directement sur un message d'erreur inattendue, auquel cas l'erreur en question accompagne votre signalement. happyDomain remplit les détails techniques pour vous, vous n'avez qu'à décrire ce que vous faisiez ;
- directement sur la forge que vous préférez, nous les lisons toutes : [GitHub](https://github.com/happyDomain/happydomain/issues), [GitLab](https://gitlab.com/happyDomain/happydomain/-/issues), [Framagit](https://framagit.org/happyDomain/happydomain/-/issues) ou [Codeberg](https://codeberg.org/happyDomain/happyDomain/issues) ;
- par email à [contact@happydomain.org](mailto:contact@happydomain.org), si vous n'avez de compte sur aucune forge (la fenêtre **Signaler un problème** peut préparer cet email pour vous) ;
- sur le [canal Matrix](https://matrix.to/#/#happyDNS:matrix.org), si vous préférez simplement en discuter ;
- **uniquement pour les failles de sécurité**, en privé : consultez [notre politique de sécurité](https://github.com/happyDomain/happydomain/blob/master/SECURITY.md).


## Le bouton *Signaler un problème*

happyDomain garde en mémoire les erreurs qu'il rencontre au fil de votre utilisation.
Lorsque vous ouvrez **Signaler un problème**, il prépare un rapport contenant votre version, votre navigateur et ces erreurs, puis ouvre un ticket pré-rempli sur la forge de votre choix, ou copie le tout dans votre presse-papier si vous préférez le coller ailleurs.

Vous pouvez relire et modifier ces détails avant de les envoyer : ils ne contiennent ni mot de passe, ni clé d'API, ni nom de domaine.


## Pour aller plus loin

Tout ce qui suit est facultatif, et nous ne vous le demanderons que si le signalement en a besoin.

### Les journaux d'happyDomain

De nombreux bugs laissent une trace côté serveur, et une erreur comme `NetworkError` dans votre navigateur correspond souvent à un plantage d'happyDomain.
Le texte complet d'un tel plantage nous indique exactement quelle ligne a cassé.

Avec Docker Compose, depuis le répertoire contenant votre `docker-compose.yml` :

```
docker compose logs -f happydomain
```

Avec un simple conteneur :

```
docker container logs -f happydomain
```

Avec l'exécutable autonome, les journaux s'affichent dans le terminal qui l'exécute (ou sont récupérés par votre gestionnaire de services : `journalctl -u happydomain`).

Reproduisez le problème pendant que les journaux défilent, puis copiez ce qui est apparu.
Vérifiez qu'il ne s'y trouve pas de clé d'API ou de mot de passe avant de les coller.

### Les erreurs de votre navigateur

1. ouvrez les outils de développement avec `F12` (ou `Ctrl+Maj+I`, `Cmd+Option+I` sur macOS) ;
2. sélectionnez l'onglet **Console**, et videz-le ;
3. reproduisez le problème ;
4. copiez les messages apparus et, dans l'onglet **Réseau**, la requête en échec.

Une capture d'écran fait aussi l'affaire. Du texte copié est plus facile à rechercher et à vous citer, mais que cela ne vous empêche jamais de nous signaler le problème.


## Vérifier une correction

Lorsque nous vous annonçons qu'un correctif est disponible, mettez à jour votre instance avant de tester à nouveau.

Avec Docker Compose :

```
docker compose up -d --pull always
```

Avec un simple conteneur, récupérez l'image puis recréez le conteneur :

```
docker image pull happydomain/happydomain
```

Les corrections arrivent sur l'image `master` et sur <https://get.happydomain.org/master/> dès qu'elles sont fusionnées, avant la sortie de la version suivante.
Si le problème persiste, dites-le nous : nous préférons en entendre parler deux fois plutôt que de le laisser en l'état.
