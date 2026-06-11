---
date: 2026-04-05T12:00:00+02:00
title: Surveillance & Vérifications
author: nemunaire
weight: 35
description: "Mettez en place des vérifications automatiques pour surveiller vos domaines et vos services"
---

happyDomain intègre un système de surveillance qui permet de lancer des vérifications automatiques sur vos domaines et vos services. Les vérificateurs collectent périodiquement des données (temps de réponse au ping, date d'expiration d'un domaine ou résultats d'audit DNS, par exemple), les évaluent au regard des seuils que vous définissez et rapportent un statut clair : OK, Avertissement, Critique ou Erreur.

## Parcourir les vérificateurs disponibles

Vous pouvez consulter l'ensemble des vérificateurs disponibles depuis la page **Vérifications**, accessible par le menu principal.

<!-- TODO: screenshot of the checkers list page -->

Chaque vérificateur indique la portée à laquelle il s'applique :

- **Niveau domaine** : vérifications qui concernent le domaine lui-même, indépendamment des enregistrements DNS (par exemple l'expiration du domaine via WHOIS).
- **Niveau zone** : vérifications qui nécessitent le contenu complet de la zone (par exemple la validation DNSSEC).
- **Niveau service** : vérifications qui ciblent un service précis sur un sous-domaine (par exemple un ping ou une vérification HTTP).

Utilisez la barre de recherche pour filtrer les vérificateurs par nom. Cliquez sur un vérificateur pour afficher sa description et ses options de configuration.


## Configurer un vérificateur pour votre domaine

Pour mettre en place un vérificateur sur l'un de vos domaines :

1. Rendez-vous sur la page de votre domaine et ouvrez l'onglet **Vérifications**.
2. Un tableau présente les vérificateurs disponibles pour ce domaine. Cliquez sur **Configurer** en regard du vérificateur souhaité.

<!-- TODO: screenshot of the domain checks list -->

La page de configuration regroupe plusieurs sections :

### Options du vérificateur

Les options sont réparties par catégorie :

- **Options d'administration** : paramètres globaux contrôlés par l'administrateur (en lecture seule pour les utilisateurs ordinaires).
- **Configuration** : préférences au niveau de l'utilisateur, comme les seuils d'avertissement et critique. Ce sont les principaux réglages que vous ajusterez.
- **Paramètres propres au domaine** : valeurs renseignées automatiquement à partir du domaine vérifié (par exemple le nom de domaine lui-même).
- **Paramètres du vérificateur** : paramètres d'exécution supplémentaires.

Renseignez ou ajustez les options selon vos besoins, puis cliquez sur **Enregistrer**.

<!-- TODO: screenshot of the checker configuration page with options -->

### Règles

Chaque vérificateur s'accompagne d'une ou plusieurs **règles** qui évaluent différents aspects des données collectées. Un vérificateur de ping peut par exemple comporter des règles distinctes pour la latence et la perte de paquets.

Vous pouvez activer ou désactiver chaque règle à l'aide des interrupteurs. Chaque règle peut également disposer de ses propres options à configurer.

<!-- TODO: screenshot of the rules section with toggles -->


## Planifier des vérifications automatiques

Une fois un vérificateur configuré, vous pouvez planifier son exécution automatique à intervalle régulier.

Dans la section **Planification** de la page de configuration du vérificateur :

1. Activez **Exécuter automatiquement** pour mettre en place la planification.
2. Définissez l'**intervalle de vérification** (en heures) : il détermine la fréquence d'exécution.
3. L'heure de la **prochaine exécution planifiée** est affichée afin que vous sachiez quand aura lieu la prochaine vérification.
4. Cliquez sur **Enregistrer** pour appliquer la planification.

<!-- TODO: screenshot of the schedule card -->

{{% notice style="info" title="Limites d'intervalle" icon="clock" %}}
Chaque vérificateur définit un intervalle minimal et maximal afin d'éviter de surcharger les services externes. L'interface respecte ces bornes lorsque vous configurez la planification.
{{% /notice %}}

Lorsque la planification est désactivée, le vérificateur peut tout de même être lancé manuellement à tout moment.


## Lancer une vérification manuellement

Vous pouvez déclencher une vérification à tout moment, sans attendre la planification :

1. Rendez-vous sur la page **Exécutions** du vérificateur (depuis la liste des vérifications du domaine, cliquez sur **Voir les résultats**).
2. Cliquez sur **Lancer la vérification**.
3. Une fenêtre s'ouvre, dans laquelle vous pouvez :
   - Sélectionner les règles à exécuter pour cette exécution.
   - Remplacer temporairement certaines options (ces remplacements ne sont pas enregistrés).
4. Cliquez sur **Lancer la vérification** pour démarrer l'exécution.

<!-- TODO: screenshot of the Run Check modal -->

Un message de confirmation apparaît avec l'identifiant de l'exécution. Le résultat s'affiche dans la liste des exécutions une fois la vérification terminée.


## Consulter les résultats des vérifications

Toutes les exécutions passées sont répertoriées sur la page **Exécutions** du vérificateur, accessible depuis l'onglet Vérifications de votre domaine en cliquant sur **Voir les résultats**.

<!-- TODO: screenshot of the executions list -->

Le tableau des exécutions présente :

| Colonne | Description |
|--------|-------------|
| **Exécutée le** | Date à laquelle la vérification a été lancée |
| **Statut** | Le résultat : OK, Avertissement, Critique, Erreur ou Inconnu |
| **Message** | Un résumé des constats |
| **Durée** | Le temps qu'a pris la vérification |
| **Type** | S'il s'agit d'une exécution planifiée ou manuelle |

Cliquez sur **Voir** sur une exécution pour en afficher les résultats détaillés.


## Comprendre le détail d'une exécution

La page de détail d'une exécution présente les résultats sous la forme la plus adaptée à ce que fournit le vérificateur.

### Métriques

Lorsqu'un vérificateur produit des métriques (temps de réponse, pourcentage de perte de paquets, etc.), la page de détail affiche :

- Un **graphique en courbes** qui trace l'évolution des valeurs de la métrique au fil des exécutions récentes.
- Un **tableau de données** qui répertorie chaque métrique avec son nom, sa valeur et son unité.

<!-- TODO: screenshot of the metrics chart and table -->

### Rapports HTML

Certains vérificateurs génèrent des rapports HTML détaillés (par exemple les résultats d'un audit DNS). Ces rapports sont affichés directement dans la page.

### Données brutes

Vous pouvez à tout moment basculer vers la vue **JSON brut** pour examiner l'ensemble des données d'observation collectées durant l'exécution. Utilisez le sélecteur de vue en haut de la section du rapport pour passer de Métriques à Rapport HTML et JSON brut.

### Évaluation des règles

Chaque règle exécutée durant l'exécution rapporte son propre statut et son propre message. Le détail règle par règle permet de comprendre quel aspect précis a déclenché un statut d'avertissement ou critique.

<!-- TODO: screenshot of rule evaluation results -->


## Vérifications au niveau service

Les vérificateurs qui s'appliquent au niveau service se configurent depuis la page du service concerné, et non depuis l'onglet Vérifications du domaine.

1. Rendez-vous sur votre domaine, puis sur le sous-domaine et le service concernés.
2. Ouvrez l'onglet **Vérifications** de ce service.
3. Le déroulement est identique à celui des vérificateurs de niveau domaine : configurer les options, mettre en place la planification, lancer des vérifications et consulter les résultats.

Les vérificateurs de niveau service reçoivent automatiquement les informations relatives au service auquel ils sont rattachés (comme les adresses IP d'un service Serveur), ce qui réduit la configuration manuelle.


## Gérer les résultats des vérifications

Depuis la liste des exécutions, vous pouvez :

- **Supprimer un résultat** en cliquant sur l'action de suppression d'une exécution précise.
- **Supprimer tous les résultats** d'un vérificateur grâce à l'option de suppression groupée (une fenêtre de confirmation apparaît).

Cela peut être utile pour faire le ménage dans les anciennes données ou repartir de zéro après des changements de configuration.


## Référence des statuts

Les vérificateurs rapportent l'un des statuts suivants, par ordre de gravité :

| Statut | Signification |
|--------|---------|
| **OK** | Tout se situe dans les paramètres acceptables |
| **Information** | Constat informatif, aucune action nécessaire |
| **Avertissement** | Un seuil est sur le point d'être atteint ; une attention est recommandée |
| **Critique** | Un seuil a été dépassé ; une action est requise |
| **Erreur** | La vérification elle-même a échoué (erreur de collecte, mauvaise configuration) |
| **Inconnu** | La vérification n'a pas pu déterminer de résultat |
