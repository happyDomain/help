---
date: 2026-06-10T12:00:00+02:00
title: Disponibilité et recherches de domaines
weight: 1800
description: "Vérifier si un domaine est disponible à l'enregistrement, et inspecter n'importe quel domaine grâce aux outils WHOIS et résolveur DNS"
---

happyDomain embarque quelques outils de diagnostic qui fonctionnent sur n'importe quel nom de domaine, qu'il soit ou non géré dans happyDomain. Ils permettent de vérifier si un nom est disponible à l'enregistrement, de consulter ses informations d'enregistrement (WHOIS), et d'interroger directement ses enregistrements DNS (résolveur).

## Vérificateur de disponibilité

La page **Disponibilité** vous permet de surveiller un ou plusieurs noms de domaine que vous souhaiteriez enregistrer, et d'être prévenu dès que l'un d'eux devient disponible.

<!-- TODO: screenshot of the availability page -->

### Surveiller un domaine

Pour commencer à surveiller un nom :

1. Saisissez le domaine qui vous intéresse (par exemple `mondomaine.example`) dans le champ situé en haut de la page.
2. Cliquez sur **Ajouter**.

Le nom est ajouté à votre liste de surveillance et une première vérification est lancée immédiatement en arrière-plan, afin de ne pas avoir à attendre la prochaine vérification automatique pour connaître son état actuel.

### Lire le statut

Chaque domaine surveillé affiche un badge qui reflète le résultat de la dernière vérification :

| Badge | Signification |
|-------|---------------|
| **Disponible** | Le nom semble libre et peut probablement être enregistré. |
| **Enregistré** | Le nom est déjà pris. |
| _(message d'erreur)_ | La vérification n'a pas pu aboutir (par exemple l'extension n'est pas prise en charge, ou le registre n'a pas pu être contacté). |
| **Jamais vérifié** | Aucun résultat n'est encore disponible. |

La date de la dernière vérification est indiquée à côté du statut.

### Revérifier et supprimer

- Cliquez sur **Vérifier maintenant** à côté d'un domaine pour déclencher immédiatement une nouvelle vérification. Une animation est affichée pendant que le contrôle s'exécute en arrière-plan, et le statut se met à jour automatiquement une fois terminé.
- Cliquez sur l'icône de corbeille pour cesser de surveiller un domaine. Une confirmation est demandée avant la suppression.

{{% notice style="tip" title="La disponibilité reste indicative" icon="lightbulb" %}}
Le résultat de disponibilité n'est qu'une indication. Certaines extensions ne peuvent pas être vérifiées de manière fiable, et un nom qui semble libre peut tout de même être réservé ou bloqué au moment de l'enregistrement. Confirmez toujours auprès d'un bureau d'enregistrement avant de compter sur un nom.
{{% /notice %}}


## Recherche WHOIS

L'outil **WHOIS** affiche les informations publiques d'enregistrement d'un domaine : son bureau d'enregistrement, ses dates importantes et son statut actuel.

<!-- TODO: screenshot of the WHOIS lookup page -->

Saisissez un nom de domaine puis lancez la recherche. happyDomain affiche alors un résumé clair des informations collectées :

- **Statut** : les statuts d'enregistrement renvoyés pour le domaine (par exemple `active`, `clientTransferProhibited`, un état `hold`, etc.), présentés sous forme de badges colorés.
- **Date de création** et **Date d'expiration** : la date du premier enregistrement du domaine et celle à laquelle son enregistrement doit expirer. L'expiration est accompagnée d'une barre de progression et d'un compte à rebours, qui passe à l'orange puis au rouge à l'approche de l'échéance.
- **Bureau d'enregistrement** : l'organisme auprès duquel le domaine est enregistré, avec un lien vers son site lorsqu'il est disponible.
- **Serveurs de noms** : les serveurs de noms faisant autorité déclarés pour le domaine.

Si le nom n'est pas enregistré, un message vous indique que le domaine est introuvable. Si la recherche elle-même échoue, l'erreur est affichée.

{{% notice style="info" title="WHOIS et vérificateur de disponibilité" icon="circle-info" %}}
Le vérificateur de disponibilité s'appuie sur les mêmes données d'enregistrement que la recherche WHOIS. Si vous souhaitez la vue complète de l'enregistrement d'un nom, utilisez l'outil WHOIS ; si vous voulez simplement savoir si un nom est libre (et être prévenu ultérieurement), ajoutez-le à votre liste de surveillance de disponibilité.
{{% /notice %}}


## Résolveur DNS

Le **résolveur DNS** vous permet d'interroger le DNS en direct de n'importe quel domaine, exactement tel qu'il est publié sur Internet en cet instant. C'est pratique pour confirmer qu'une modification s'est bien propagée, ou pour inspecter un domaine que vous ne gérez pas.

<!-- TODO: screenshot of the resolver page -->

Saisissez un nom de domaine puis lancez la requête. Par défaut, tous les types d'enregistrement sont demandés (`ANY`). Ouvrez les options **Avancées** pour affiner la requête :

- **Champ (type d'enregistrement)** : restreignez la requête à un seul type d'enregistrement (`A`, `AAAA`, `MX`, `TXT`, etc.).
- **Résolveur** : choisissez le résolveur DNS auquel envoyer la requête. Plusieurs résolveurs publics réputés sont proposés, et vous pouvez sélectionner **Personnalisé** pour saisir l'adresse du résolveur de votre choix.
- **Afficher les enregistrements DNSSEC** : inclure les enregistrements liés à DNSSEC (`RRSIG`, `NSEC`, `NSEC3`) dans les résultats, masqués par défaut.

Si vous gérez des domaines dans happyDomain, leurs noms vous sont suggérés au fur et à mesure que vous saisissez le domaine à interroger.

Les résultats sont regroupés par type d'enregistrement, et chaque entrée affiche ses champs ainsi que son TTL. Lorsque le domaine existe mais ne possède aucun enregistrement du type demandé, un message l'indique explicitement.

{{% notice style="tip" title="Résolveur et votre zone dans happyDomain" icon="lightbulb" %}}
Le résolveur montre ce qui est **actuellement publié** par les serveurs faisant autorité, ce qui peut différer d'une zone en cours d'édition dans happyDomain tant que vous n'avez pas [publié vos modifications]({{% relref "publish-changes" %}}).
{{% /notice %}}
