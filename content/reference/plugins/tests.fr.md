---
title: "Écrire un plugin happyDomain"
description: "Guide technique pour développer des plugins de test pour happyDomain"
---

happyDomain prend en charge des **plugins de test** externes — des bibliothèques partagées (fichiers `.so`) qui ajoutent des vérifications de santé sur les domaines ou les services d'une instance en cours d'exécution. Les plugins sont chargés au démarrage sans recompiler le serveur ; l'opérateur dépose simplement un fichier `.so` dans un répertoire configuré.

## Fonctionnement

Un plugin reçoit un ensemble d'options assemblées depuis plusieurs portées de configuration, exécute une vérification (appel HTTP, requête DNS, …) et renvoie un résultat avec un niveau de statut et un rapport détaillé optionnel. Les résultats sont stockés et affichés dans l'interface happyDomain aux côtés du domaine ou du service concerné.

Au démarrage, happyDomain parcourt chaque répertoire listé dans l'option de configuration `plugins-directories`. Pour chaque fichier trouvé, il :

1. Ouvre la bibliothèque partagée.
2. Recherche le symbole exporté `NewTestPlugin`.
3. Appelle `NewTestPlugin()` pour obtenir une valeur de plugin.
4. Enregistre le plugin sous chaque nom renvoyé par `PluginEnvName()`.

Si le fichier n'est pas un plugin Go valide, si `NewTestPlugin` est absent ou s'il retourne une erreur, un avertissement est journalisé et le fichier est ignoré. Le serveur démarre toujours, quels que soient les échecs de chargement individuels.

---

## L'interface `TestPlugin`

Tout plugin doit implémenter quatre méthodes :

```go
type TestPlugin interface {
    PluginEnvName() []string
    Version()          PluginVersionInfo
    AvailableOptions() PluginOptionsDocumentation
    RunTest(PluginOptions, map[string]string) (*PluginResult, error)
}
```

---

## Structure du projet

Un plugin est un module Go autonome compilé avec `-buildmode=plugin`. Il doit être dans `package main` et exporter exactement un symbole :

```go
func NewTestPlugin() (happydns.TestPlugin, error)
```

Organisation recommandée :

```
myplugin/
├── go.mod
├── Makefile
└── plugin.go       # (ou réparti sur plusieurs fichiers .go)
```

### go.mod

```
module git.happydns.org/happyDomain/plugins/myplugin

go 1.25

require git.happydns.org/happyDomain v0.0.0
replace git.happydns.org/happyDomain => ../../
```

La directive `replace` pointe vers votre dépôt local happyDomain, garantissant que le plugin est compilé avec exactement les mêmes types que le serveur.

{{< notice style="warning" >}}
Un plugin Go et le processus hôte partagent le même environnement d'exécution. Ils **doivent** être compilés avec la même version de la chaîne d'outils Go et les mêmes versions de toutes les dépendances partagées. Tout écart provoque une erreur fatale au chargement.
{{< /notice >}}

---

## Point d'entrée

```go
package main

import "git.happydns.org/happyDomain/model"

func NewTestPlugin() (happydns.TestPlugin, error) {
    return &MyPlugin{}, nil
}
```

Le constructeur est l'endroit idéal pour effectuer une initialisation unique (ouvrir des fichiers de configuration, créer un client HTTP, …). Retournez une erreur si le plugin ne peut pas fonctionner.

---

## Nommage — `PluginEnvName()`

Renvoie un ou plusieurs identifiants courts en minuscules. Ces noms sont utilisés pour retrouver le plugin via l'API et pour indexer sa configuration stockée.

```go
func (p *MyPlugin) PluginEnvName() []string {
    return []string{"myplugin"}
}
```

Choisissez des noms peu susceptibles d'entrer en conflit (ex. `"zonemaster"`, `"matrixim"`) et gardez-les **stables entre les versions**, car ils sont persistés avec la configuration utilisateur. Si deux plugins chargés revendiquent le même nom, le second est ignoré et un conflit est journalisé.

---

## Version et disponibilité — `Version()`

Décrit le plugin et contrôle l'endroit où il apparaît dans l'interface :

```go
func (p *MyPlugin) Version() happydns.PluginVersionInfo {
    return happydns.PluginVersionInfo{
        Name:    "My Plugin",
        Version: "1.0",
        AvailableOn: happydns.PluginAvailability{
            ApplyToDomain:    true,
            ApplyToService:   false,
            LimitToProviders: nil,  // nil ou vide = tous les fournisseurs
            LimitToServices:  []string{"abstract.MatrixIM"},
        },
    }
}
```

| Champ | Type | Description |
|---|---|---|
| `ApplyToDomain` | `bool` | Le plugin peut être exécuté sur un domaine entier |
| `ApplyToService` | `bool` | Le plugin peut être exécuté sur un service DNS spécifique |
| `LimitToProviders` | `[]string` | Restreint à certains identifiants de fournisseurs DNS (vide = aucune restriction) |
| `LimitToServices` | `[]string` | Restreint à certains types de services, ex. `"abstract.MatrixIM"` (vide = aucune restriction) |

`ApplyToDomain` et `ApplyToService` peuvent être tous les deux `true` simultanément.

---

## Options — `AvailableOptions()`

Les options sont des paires clé/valeur (`map[string]any`) qui configurent chaque exécution de test. Elles sont déclarées regroupées par **portée**, c'est-à-dire qui les définit et combien de temps elles persistent :

```go
func (p *MyPlugin) AvailableOptions() happydns.PluginOptionsDocumentation {
    return happydns.PluginOptionsDocumentation{
        RunOpts:     []happydns.PluginOptionDocumentation{ /* … */ },
        ServiceOpts: []happydns.PluginOptionDocumentation{ /* … */ },
        DomainOpts:  []happydns.PluginOptionDocumentation{ /* … */ },
        UserOpts:    []happydns.PluginOptionDocumentation{ /* … */ },
        AdminOpts:   []happydns.PluginOptionDocumentation{ /* … */ },
    }
}
```

### Portées des options

| Portée | Qui la définit | Clé de stockage | Usage typique |
|---|---|---|---|
| `RunOpts` | L'utilisateur, au moment du test | _(transitoire)_ | Paramètres propres à l'exécution |
| `ServiceOpts` | L'utilisateur | plugin + utilisateur + domaine + service | Configuration au niveau du service |
| `DomainOpts` | L'utilisateur | plugin + utilisateur + domaine | Configuration au niveau du domaine |
| `UserOpts` | L'utilisateur | plugin + utilisateur | Préférences personnelles (ex. langue) |
| `AdminOpts` | L'administrateur | plugin | Paramètres d'instance, identifiants partagés |

Avant l'appel à `RunTest`, happyDomain fusionne toutes les valeurs par portée, de la moins spécifique (admin) à la plus spécifique (exécution). Les valeurs plus spécifiques écrasent silencieusement les moins spécifiques. `RunTest` reçoit toujours une map plate unique et n'a pas besoin de savoir de quelle portée provient chaque valeur.

### Champs d'une option

Chaque option est un `PluginOptionDocumentation` (un alias pour `Field`) :

| Champ | Type | Description |
|---|---|---|
| `Id` | `string` | **Obligatoire.** Clé utilisée dans la map `PluginOptions` dans `RunTest` |
| `Type` | `string` | Type de saisie : `"string"`, `"select"` |
| `Label` | `string` | Libellé lisible affiché dans l'interface |
| `Placeholder` | `string` | Texte indicatif du champ de saisie |
| `Default` | `any` | Valeur par défaut pré-remplie dans le formulaire |
| `Choices` | `[]string` | Options pour les saisies de type `"select"` |
| `Required` | `bool` | Indique si le champ doit être rempli avant l'exécution |
| `Secret` | `bool` | Marque le champ comme sensible (ex. une clé API) |
| `Hide` | `bool` | Masque entièrement le champ à l'utilisateur |
| `Textarea` | `bool` | Affiche une zone de texte multiligne |
| `Description` | `string` | Texte d'aide affiché sous le champ |
| `AutoFill` | `string` | Remplit le champ automatiquement depuis le contexte (voir ci-dessous) |

### Remplissage automatique

Lorsque `AutoFill` est défini, happyDomain remplit le champ à partir du contexte du test ; l'utilisateur n'est pas sollicité :

| Constante | Valeur chaîne | Rempli avec |
|---|---|---|
| `happydns.AutoFillDomainName` | `"domain_name"` | FQDN du domaine testé, ex. `"example.com."` |
| `happydns.AutoFillSubdomain` | `"subdomain"` | Sous-domaine relatif à la zone, ex. `"www"` — tests à portée service uniquement |
| `happydns.AutoFillServiceType` | `"service_type"` | Identifiant du type de service, ex. `"abstract.MatrixIM"` — tests à portée service uniquement |

```go
{
    Id:       "domainName",
    Type:     "string",
    Label:    "Nom de domaine",
    AutoFill: happydns.AutoFillDomainName,
    Required: true,
}
```

---

## Exécuter la vérification — `RunTest()`

`RunTest` reçoit la map d'options fusionnée et une map de métadonnées (réservée à un usage futur), effectue la vérification et renvoie un `PluginResult`.

Convertissez toujours les valeurs d'options vers un type concret avant de les utiliser — la map contient des valeurs de type `any` :

```go
func (p *MyPlugin) RunTest(opts happydns.PluginOptions, _ map[string]string) (*happydns.PluginResult, error) {
    domain, ok := opts["domainName"].(string)
    if !ok || domain == "" {
        return nil, fmt.Errorf("l'option domainName est obligatoire")
    }

    // … effectuer la vérification …

    return &happydns.PluginResult{
        Status:     happydns.PluginResultStatusOK,
        StatusLine: "Tout est bon",
        Report:     myStructuredReport,
    }, nil
}
```

Retournez une **erreur non nulle** uniquement pour les échecs inattendus (erreurs réseau, configuration invalide). Pour les échecs de vérification attendus — le service surveillé est indisponible, les enregistrements DNS sont incorrects — retournez un `PluginResult` avec un statut approprié et un `StatusLine` lisible par un humain.

### Champs du résultat

| Champ | Type | Description |
|---|---|---|
| `Status` | `PluginResultStatus` | Niveau de résultat global (voir ci-dessous) |
| `StatusLine` | `string` | Résumé court affiché dans l'interface |
| `Report` | `any` | Toute valeur sérialisable en JSON, stockée comme données de diagnostic structurées |

### Niveaux de statut (du pire au meilleur)

| Constante | Signification |
|---|---|
| `PluginResultStatusKO` | La vérification a échoué |
| `PluginResultStatusWarn` | La vérification a réussi avec des avertissements |
| `PluginResultStatusInfo` | Informatif, aucune action requise |
| `PluginResultStatusOK` | La vérification a entièrement réussi |

---

## Compilation

```bash
go build -buildmode=plugin -o happydomain-plugin-test-myplugin.so \
    git.happydns.org/happyDomain/plugins/myplugin
```

`Makefile` minimal :

```makefile
PLUGIN_NAME=myplugin
TARGET=../happydomain-plugin-test-$(PLUGIN_NAME).so

all: $(TARGET)

$(TARGET): *.go
	go build -buildmode=plugin -o $@ git.happydns.org/happyDomain/plugins/$(PLUGIN_NAME)
```

Le préfixe `happydomain-plugin-test-` est une convention ; happyDomain charge tous les fichiers présents dans les répertoires de plugins, quel que soit leur nom.

---

## Déploiement

### 1. Copier le fichier `.so`

```bash
cp happydomain-plugin-test-myplugin.so /usr/lib/happydomain/plugins/
```

### 2. Indiquer le répertoire à happyDomain

`happydomain.conf` :

```
plugins-directories=/usr/lib/happydomain/plugins
```

Variable d'environnement :

```bash
HAPPYDOMAIN_PLUGINS_DIRECTORIES=/usr/lib/happydomain/plugins
```

Plusieurs répertoires peuvent être listés en les séparant par des virgules.

### 3. Vérifier les journaux

En cas de chargement réussi :

```
Plugin My Plugin loaded (version 1.0)
```

En cas de conflit de nom ou d'erreur de chargement, un avertissement est journalisé avec le nom du fichier et la raison.

---

## Implémentations de référence

Deux plugins sont fournis dans ce répertoire :

- **`matrix/`** — interroge l'API de test de fédération Matrix. Illustre `ApplyToService` avec `LimitToServices` et `AdminOpts` pour l'URL du serveur tiers.
- **`zonemaster/`** — pilote l'API JSON-RPC de Zonemaster, attend la fin du test et agrège les résultats par niveau de sévérité. Illustre `AutoFillDomainName`, `UserOpts` pour la sélection de la langue et la gestion de statuts multi-niveaux.
