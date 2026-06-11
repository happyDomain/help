---
date: 2026-06-10T12:00:00+02:00
title: Notifications
weight: 2300
description: "Recevoir des alertes lorsque vos contrôles de supervision changent d'état, et configurer où et quand ces alertes sont délivrées"
---

happyDomain peut vous prévenir lorsque quelque chose change sur vos domaines. Les notifications reposent sur le système de [supervision et contrôles]({{% relref "checks" %}}) : chaque fois qu'un contrôle change de statut (par exemple de **OK** à **Avertissement**, ou un retour à **OK** après un problème), happyDomain peut envoyer une alerte vers les canaux que vous avez configurés.

Vous gérez les notifications depuis la page **Notifications** des paramètres de votre compte. Elle s'organise en trois onglets : **Canaux**, **Préférences** et **Historique**.

<!-- TODO: screenshot of the notifications page with its three tabs -->


## Ce qui déclenche une notification

Les notifications sont liées à vos contrôles. happyDomain surveille le statut rapporté par chaque contrôle et envoie une notification lorsque :

- un contrôle **se dégrade** jusqu'à (ou au-delà d') une gravité qui vous importe, par exemple en atteignant **Avertissement**, **Critique** ou **Erreur** ;
- un contrôle **se rétablit**, en revenant à un état sain, si vous avez demandé à être prévenu des rétablissements.

Chaque notification décrit donc une transition de statut (l'ancien statut et le nouveau) pour une cible donnée. Les transitions qui vous parviennent, et par quels canaux, sont entièrement déterminées par vos **préférences** (voir ci-dessous).


## Canaux

Un **canal** est une destination vers laquelle les notifications sont envoyées. Ouvrez l'onglet **Canaux** pour les gérer.

Pour en ajouter un, cliquez sur **Ajouter**, choisissez un **Type**, donnez-lui un **Nom** (afin de le reconnaître par la suite) et renseignez les champs propres à ce type. Un canal peut être **activé** ou désactivé à l'aide d'un interrupteur, sans avoir à le supprimer.

happyDomain propose les types de canaux suivants d'origine :

### E-mail

Envoie les notifications à une adresse e-mail.

- **Adresse e-mail** : le destinataire. Si le champ est laissé vide, la notification est envoyée à l'adresse e-mail de votre compte.

### Webhook

Envoie une requête HTTP vers une URL de votre choix, ce qui est utile pour intégrer happyDomain à des outils de discussion, des plateformes d'automatisation ou vos propres services.

- **URL du webhook** (obligatoire) : le point d'accès qui recevra la notification.
- **En-têtes du webhook** : des en-têtes HTTP personnalisés facultatifs (paires nom/valeur) à ajouter à la requête, par exemple un en-tête d'autorisation.
- **Secret du webhook** : un secret facultatif servant à signer la requête, afin que le destinataire puisse vérifier qu'elle provient bien de happyDomain.

### UnifiedPush

Délivre des notifications push sur vos appareils via un distributeur [UnifiedPush](https://unifiedpush.org/).

- **Point d'accès UnifiedPush** (obligatoire) : l'URL du point d'accès fournie par votre application UnifiedPush.

{{% notice style="info" title="Autres types de canaux" icon="circle-info" %}}
La liste des types disponibles dépend de ce que l'administrateur de l'instance a activé. Pour les types pour lesquels happyDomain ne propose pas de formulaire dédié, l'éditeur bascule sur un champ de configuration JSON brut.
{{% /notice %}}

### Tester un canal

Une fois un canal créé et activé, utilisez le bouton d'envoi/test situé à côté de lui pour délivrer une notification de test. Cela confirme que la configuration fonctionne avant de compter dessus pour de vraies alertes.

<!-- TODO: screenshot of the channels list with the test button -->


## Préférences

Les canaux indiquent *où* vont les notifications ; les **préférences** indiquent *quoi* est envoyé et *quand*. Ouvrez l'onglet **Préférences** puis cliquez sur **Ajouter** pour créer une règle.

Une préférence combine les réglages suivants :

### Portée

Choisissez l'étendue d'application de la règle :

- **Globale** : s'applique à l'ensemble de vos domaines et services.
- **Domaine** : s'applique à un seul domaine que vous sélectionnez.
- **Service** : s'applique à un service précis (vous sélectionnez le domaine et fournissez l'identifiant du service).

### Canaux

Sélectionnez un ou plusieurs de vos canaux configurés pour recevoir les notifications correspondant à cette préférence. Si vous n'avez pas encore de canal, créez-en un d'abord dans l'onglet **Canaux**.

### Statut minimum

Choisissez la gravité la plus basse qui doit déclencher une notification. Les niveaux disponibles, par gravité croissante, sont **OK**, **Info**, **Avertissement**, **Critique** et **Erreur**. Seuls les changements de statut qui atteignent ce niveau (ou un niveau supérieur) sont notifiés. Par exemple, choisir **Avertissement** signifie que vous êtes alerté pour les états Avertissement, Critique et Erreur, mais pas pour les changements purement informatifs.

### Notifier le rétablissement

Lorsque cette option est activée, vous recevez également une notification quand un contrôle auparavant dégradé revient à un état sain, afin de savoir qu'un problème a été résolu.

### Heures calmes

Vous pouvez définir une période durant laquelle les notifications sont retenues, par exemple la nuit. Indiquez une heure de **début** et une heure de **fin** (de 0 à 23). Lorsque les heures calmes sont actives, les alertes survenant dans cette plage ne sont pas envoyées immédiatement.

### Activée

Chaque préférence peut être activée ou désactivée à l'aide d'un interrupteur, ce qui permet de suspendre temporairement une règle sans la supprimer.

<!-- TODO: screenshot of the preference editor -->

{{% notice style="tip" title="Commencez simplement" icon="lightbulb" %}}
Une configuration courante consiste en une unique préférence **Globale**, pointant vers un canal, avec un statut minimum d'**Avertissement** et les notifications de rétablissement activées. Vous pourrez ensuite ajouter des règles plus spécifiques par domaine ou par service au fur et à mesure de vos besoins.
{{% /notice %}}


## Historique

L'onglet **Historique** liste les notifications que happyDomain a tenté d'envoyer. Pour chaque entrée, vous pouvez voir :

- la cible concernée ;
- la transition de statut (ancien statut → nouveau statut) ;
- si l'envoi a **réussi** ou **échoué** (avec le message d'erreur en cas d'échec).

Utilisez **Charger plus** pour remonter dans les entrées plus anciennes. Cette vue est l'endroit où vérifier pourquoi une alerte attendue ne vous est pas parvenue, par exemple un canal mal configuré provoquant des échecs répétés.

<!-- TODO: screenshot of the notification history list -->
