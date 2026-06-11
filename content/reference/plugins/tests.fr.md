---
title: "Écrire un plugin de vérification happyDomain"
description: "Guide technique pour développer des plugins checker pour happyDomain"
---

happyDomain peut être étendu par des **plugins de vérification** (*checkers*) externes : des bibliothèques partagées (fichiers `.so`) qui ajoutent des diagnostics automatisés sur les zones, les domaines, les services ou les utilisateurs. Un plugin checker est chargé dans le processus happyDomain au démarrage. L'opérateur dépose simplement un fichier `.so` dans un répertoire configuré, sans recompiler le serveur.

Un checker comporte deux moitiés. Il **collecte** d'abord des données brutes sur une cible (une observation). Il **évalue** ensuite ces données au regard d'un ensemble de règles afin de produire un statut. Les résultats sont stockés puis affichés dans l'interface de happyDomain, à côté du domaine ou du service concerné.

{{< notice style="warning" >}}
Un plugin `.so` est chargé dans le processus happyDomain et s'exécute avec les mêmes privilèges que le serveur. Le répertoire des plugins doit être considéré comme un emplacement de confiance : happyDomain refuse de charger des plugins depuis un répertoire qui ne l'est pas (voir [Sécurité et déploiement](#sécurité-et-déploiement)).
{{< /notice >}}

---

## Ce qu'un plugin checker doit exporter

Les plugins sont construits avec le module **`checker-sdk-go`**, publié séparément du cœur de happyDomain. Dans cette page, `checker` désigne le paquet `git.happydns.org/checker-sdk-go/checker`.

Le chargeur de happyDomain recherche un unique symbole exporté nommé `NewCheckerPlugin`, avec cette signature exacte :

```go
func NewCheckerPlugin() (*checker.CheckerDefinition, checker.ObservationProvider, error)
```

Les deux valeurs de retour décrivent les deux moitiés d'un checker :

- **`*CheckerDefinition`** décrit le checker : son identifiant, son nom, sa version, les clés d'observation dont il dépend, les options qu'il accepte, ses règles, un agrégateur optionnel, un intervalle de planification, et s'il expose des rapports HTML ou des métriques. Voir le [tableau des champs](#champs-de-checkerdefinition) ci-dessous.
- **`ObservationProvider`** est la moitié chargée de la collecte. Elle expose une méthode `Key()` (la clé d'observation que les règles consultent) et une méthode `Collect(ctx, opts)` qui renvoie la charge utile brute de l'observation. happyDomain sérialise ce résultat en JSON et le met en cache pour chaque contexte d'observation.
- Renvoyez une `error` non nulle si le plugin ne peut pas s'initialiser (variable d'environnement manquante, dépendance cgo cassée, …). L'hôte journalise l'erreur et ignore le fichier, sans interrompre le démarrage.

Un même fichier `.so` peut exporter plusieurs types de plugins. Le chargeur applique chaque chargeur connu à chaque fichier, puis ignore tout symbole qu'il ne reconnaît pas. Un binaire peut donc fournir plusieurs plugins.

---

## Exemple minimal

Voici le plus petit plugin qui se charge. Il collecte une observation fixe et ne déclare aucune règle. On peut l'adapter à partir de [`checker-dummy`](https://git.happydns.org/checker-dummy), l'implémentation de référence.

```go
// Command plugin is the happyDomain plugin entrypoint for the dummy checker.
//
// Build with:
//   go build -buildmode=plugin -o checker-dummy.so ./plugin
package main

import (
	"context"

	"git.happydns.org/checker-sdk-go/checker"
)

type dummyProvider struct{}

func (dummyProvider) Key() checker.ObservationKey { return "dummy.observation" }

func (dummyProvider) Collect(ctx context.Context, opts checker.CheckerOptions) (any, error) {
	return map[string]string{"hello": "world"}, nil
}

// NewCheckerPlugin is the symbol resolved by happyDomain at startup.
func NewCheckerPlugin() (*checker.CheckerDefinition, checker.ObservationProvider, error) {
	def := &checker.CheckerDefinition{
		ID:              "com.example.dummy",
		Name:            "Dummy checker",
		Version:         "0.1.0",
		ObservationKeys: []checker.ObservationKey{"dummy.observation"},
		// Add Rules / Aggregator / Options here in a real plugin.
	}
	return def, dummyProvider{}, nil
}
```

{{< notice style="warning" >}}
Un plugin Go et le processus hôte partagent le même *runtime*. Ils **doivent** être compilés avec la même version de la chaîne d'outils Go et les mêmes versions de chaque dépendance partagée. Toute divergence produit une erreur bloquante au chargement. Voir [Contraintes de build](#contraintes-de-build).
{{< /notice >}}

---

## Champs de `CheckerDefinition`

Le `*CheckerDefinition` renvoyé par `NewCheckerPlugin` est la description de votre checker :

| Champ | Type | Description |
|---|---|---|
| `ID` | `string` | **Requis.** Identifiant stable et persistant. Choisissez une valeur préfixée par un espace de noms (`com.example.dnssec-freshness`, et non `dnssec`) et ne la changez jamais : elle indexe les résultats stockés et la configuration utilisateur. |
| `Name` | `string` | Nom lisible affiché dans l'interface. |
| `Version` | `string` | Version du plugin (par exemple `"1.0.0"`). |
| `Availability` | `CheckerAvailability` | Déclare les portées auxquelles le checker s'applique et d'éventuelles restrictions de fournisseur ou de service (voir ci-dessous). |
| `Options` | `CheckerOptionsDocumentation` | Documente les options acceptées par le checker, regroupées par portée (voir ci-dessous). |
| `Rules` | `[]CheckRule` | Les règles évaluées sur l'observation collectée. |
| `Aggregator` | `CheckAggregator` | Optionnel. Combine les `CheckState` produits par chaque règle en un état de synthèse unique. |
| `Interval` | `*CheckIntervalSpec` | Bornes de planification optionnelles (durées `Min`, `Max`, `Default`). |
| `HasHTMLReport` | `bool` | À activer lorsque le provider implémente `CheckerHTMLReporter`. |
| `HasMetrics` | `bool` | À activer lorsque le provider implémente `CheckerMetricsReporter`. |
| `ObservationKeys` | `[]ObservationKey` | Les clés d'observation que ce checker lit. |

### Disponibilité

`CheckerAvailability` contrôle l'endroit où le checker est proposé dans l'interface :

| Champ | Type | Description |
|---|---|---|
| `ApplyToDomain` | `bool` | Le checker peut s'exécuter sur un domaine entier. |
| `ApplyToZone` | `bool` | Le checker peut s'exécuter sur une zone. |
| `ApplyToService` | `bool` | Le checker peut s'exécuter sur un service précis. |
| `LimitToProviders` | `[]string` | Restreint à certains identifiants de fournisseurs DNS (vide = aucune restriction). |
| `LimitToServices` | `[]string` | Restreint à certains identifiants de types de services, par exemple `"abstract.MatrixIM"` (vide = aucune restriction). |

### Options

Les options sont déclarées par **portée**, c'est-à-dire selon qui les définit et combien de temps elles persistent. Chaque portée est une tranche de `CheckerOptionDocumentation` :

| Portée | Qui la définit | Usage habituel |
|---|---|---|
| `AdminOpts` | Administrateur | Réglages valables pour toute l'instance, identifiants partagés. |
| `UserOpts` | Utilisateur | Préférences personnelles (par exemple la langue). |
| `DomainOpts` | Utilisateur | Configuration au niveau du domaine. |
| `ServiceOpts` | Utilisateur | Configuration au niveau du service. |
| `RunOpts` | Utilisateur, au moment de l'exécution | Paramètres propres à une invocation. |

happyDomain fusionne les valeurs des différentes portées, de la moins spécifique (administrateur) à la plus spécifique (exécution), avant d'appeler `Collect`. Le provider reçoit donc une unique table `CheckerOptions` à plat. Chaque option est un `CheckerOptionField` avec des champs comme `Id`, `Type`, `Label`, `Default`, `Choices`, `Required`, `Secret`, `Description` et `AutoFill`. On lit les valeurs typées de la table à l'aide des fonctions du SDK `checker.GetOption`, `checker.GetIntOption`, `checker.GetBoolOption`, …

{{< notice style="info" >}}
Lorsque happyDomain enregistre un checker externalisable, il ajoute automatiquement une option d'administration `endpoint`. L'administrateur peut ainsi déléguer la collecte à un point d'accès HTTP distant plutôt que de l'exécuter dans le processus. Laissée vide, elle fait fonctionner le checker localement.
{{< /notice >}}

---

## L'`ObservationProvider`

Le provider est la moitié du checker chargée de la collecte :

```go
type ObservationProvider interface {
	Key() ObservationKey
	Collect(ctx context.Context, opts CheckerOptions) (any, error)
}
```

- `Key()` renvoie la clé d'observation que ce provider remplit. Elle doit correspondre à l'une des `ObservationKeys` déclarées dans la définition.
- `Collect` effectue le travail réel (une requête DNS, un appel HTTP, …) et renvoie n'importe quelle valeur sérialisable en JSON. happyDomain la convertit en JSON et la met en cache ; les règles la relisent ensuite.

Un provider peut implémenter des interfaces supplémentaires du SDK pour étendre son comportement :

| Interface | Rôle |
|---|---|
| `CheckerHTMLReporter` | `GetHTMLReport(ctx ReportContext)` rend l'observation stockée sous forme de document HTML. |
| `CheckerMetricsReporter` | `ExtractMetrics(ctx ReportContext, collectedAt)` produit des métriques temporelles. |
| `CheckEnabler` | `IsEligible(ctx, opts)` décide, à partir des données réelles de la cible, s'il est pertinent d'exécuter le checker. |
| `DiscoveryPublisher` | `DiscoverEntries(data)` publie des enregistrements `DiscoveryEntry` que d'autres checkers peuvent consommer. |

Les règles renvoient des valeurs `CheckState` dont le `Status` vaut `StatusOK`, `StatusInfo`, `StatusWarn`, `StatusCrit`, `StatusError` ou `StatusUnknown`.

### Optionnel : serveur autonome

Le SDK fournit aussi `checker.Server`, une ossature HTTP pour exécuter un checker comme un point d'accès distant plutôt que (ou en plus) d'un plugin chargé dans le processus. Elle expose les routes `/health` et `/collect`, ainsi que `/definition`, `/evaluate` et `/report` lorsque le provider implémente les interfaces optionnelles correspondantes. Un provider qui implémente `CheckerInteractive` (`RenderForm` / `ParseForm`) dispose en outre d'un formulaire `/check` destiné aux humains, utilisable en dehors de happyDomain. Voir le [README du SDK](https://git.happydns.org/checker-sdk-go) pour les détails ; le mode plugin décrit plus haut n'en a pas besoin.

---

## Contraintes de build

Le paquet `plugin` de Go est intransigeant. Pour se charger correctement, votre plugin doit être compilé avec :

- la **même version de la chaîne d'outils Go** que happyDomain, jusqu'au même niveau de correctif ;
- les **mêmes versions de chaque dépendance partagée** (à figer dans votre `go.mod`, en vendorisant les versions exactes que happyDomain embarque) ;
- `CGO_ENABLED=1` ;
- les mêmes `GOOS` et `GOARCH` que le binaire hôte.

Si l'un de ces points ne correspond pas, `plugin.Open` échoue avec une erreur parfois obscure, du type *« plugin was built with a different version of package … »*. L'hôte la journalise et ignore le fichier.

Le paquet `plugin` de Go ne fonctionne que sur **linux**, **darwin** et **freebsd**. Sur les autres plateformes, happyDomain est construit sans prise en charge des plugins et les répertoires configurés sont ignorés, avec un avertissement journalisé au démarrage.

---

## Sécurité et déploiement

### Permissions du répertoire et des fichiers

Charger un fichier `.so` revient à exécuter du code arbitraire avec les droits du processus happyDomain. Le chargeur impose donc des règles strictes de propriété avant de toucher au moindre fichier :

- Le répertoire des plugins **ne doit pas être un lien symbolique** : happyDomain refuse de le suivre, pour éviter qu'il soit redirigé vers un chemin contrôlé par un attaquant.
- Le répertoire des plugins **ne doit pas être accessible en écriture au groupe ni à tous**. Un répertoire modifiable par quelqu'un d'autre que son propriétaire est traité comme une erreur de configuration bloquante et interrompt le chargement.
- Tout fichier `.so` **accessible en écriture au groupe ou à tous est ignoré** (journalisé puis écarté), même dans un répertoire par ailleurs verrouillé.

En pratique : conservez le répertoire détenu par l'utilisateur happydomain, en mode `0755`, et les fichiers de plugins en mode `0644`.

```bash
sudo install -d -m 0755 -o happydomain /var/lib/happydomain/plugins
sudo install -m 0644 -o happydomain checker-dummy.so /var/lib/happydomain/plugins/
```

### Construire le plugin

```bash
CGO_ENABLED=1 go build -buildmode=plugin -o checker-dummy.so ./plugin
```

### Indiquer le répertoire à happyDomain

Le répertoire se configure avec l'option **`--plugins-directory`**, qui **peut être répétée** pour analyser plusieurs répertoires :

```bash
happydomain --plugins-directory /var/lib/happydomain/plugins
```

La variable d'environnement équivalente est `HAPPYDOMAIN_PLUGINS_DIRECTORY`.

Le chargeur analyse chaque répertoire configuré et tente de charger tous les fichiers `.so` qu'il y trouve. Un plugin qui échoue au chargement (mauvaise compilation, symboles absents, panique dans sa fabrique) est journalisé puis ignoré, sans interrompre le démarrage : un seul `.so` défectueux n'empêche jamais le chargement des autres.

### Redémarrer et vérifier les journaux

```bash
sudo systemctl restart happydomain
```

En cas de chargement réussi, happyDomain journalise :

```
Plugin com.example.dummy (/var/lib/happydomain/plugins/checker-dummy.so) loaded
```

---

## Licence

Les plugins checker n'importent que `git.happydns.org/checker-sdk-go/checker`, sous licence **Apache-2.0**. Le SDK a été délibérément détaché du cœur de happyDomain (sous AGPL-3.0) pour offrir une API publique réduite et stable aux checkers tiers.

Un plugin construit avec ce SDK n'est donc **pas** une œuvre dérivée de happyDomain. Vous pouvez distribuer votre `.so` sous la licence de votre choix (MIT, Apache, propriétaire ou AGPL, selon vos besoins).

---

## Implémentation de référence

[`checker-dummy`](https://git.happydns.org/checker-dummy) est le modèle complet et documenté dont s'inspire cette page. Partez-en pour écrire votre propre checker.
