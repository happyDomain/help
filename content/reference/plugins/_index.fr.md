---
date: 2025-06-15T11:38:46+02:00
title: Plugins
author: nemunaire
archetype: chapter
weight: 40
---

Les plugins sont des morceaux de code externes (des bibliothèques partagées chargées au démarrage) qui étendent les fonctionnalités de happyDomain sans recompiler le serveur. Il suffit de déposer un fichier `.so` dans un répertoire configuré pour que happyDomain le charge automatiquement au prochain démarrage.

Cela nécessite une **version d'happyDomain compatible avec les plugins** : les binaires et l'image de conteneur par défaut sont compilés statiquement et ignorent silencieusement les répertoires de plugins. Cherchez un binaire ou une étiquette de conteneur suffixée par `-cgo`, disponible sur les plateformes où Go prend en charge les plugins (Linux, et quelques autres).

{{% children %}}
