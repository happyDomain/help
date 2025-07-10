---
data: 2024-07-15T11:33:09+02:00
title: Configuration pour l'API OVH
weight: 50
---

Afin de pouvoir gérer des domaines hébergés par OVH, il est nécessaire de passer par une étape de configuration supplémentaire.

En effet, le principe de fonctionnement de l'authentification à l'API d'OVH se fait en 2 étapes :

1. Tout d'abord il faut enregistrer une ***application*** (par exemple *happyDomain*). Une application dispose d'un identifiant et d'un secret qu'il convient de renseigner en tant que paramètre d'happyDomain.
2. Pour chaque compte OVH que l'on souhaite gérer, l'interface d'happyDomain redirige vers la page d'OVH permettant de créer la *Consumer key*.

L'*application* doit être créée à partir d'un compte OVH existant, peu importe qu'il dispose de domaines ou non, il s'agit d'identifier la personne responsable de la mise en œuvre de l'application désignée.
L'accès aux données d'un compte, notamment aux domaines qu'ils gèrent, se fait au travers de la *Consumer key*.

Pour plus d'information, voir cette page : <https://docs.ovh.com/gb/en/api/api-rights-delegation/#application-registration>


## Enregistrer votre instance d'happyDomain auprès d'OVH

1. Rendez-vous sur la page <https://api.ovh.com/createApp/>.
2. Remplissez le formulaire avec un nom et une description qui refléte le nom et l'usage de votre application, par exemple *happyDomain*.

Une fois le formulaire validé, vous obtiendrez une clef et un secret.
Ces informations sont à indiquer dans la configuration d'happyDomain :

`ovh-application-key`
: Application key pour l'API d'OVH

`ovh-application-secret`
: Clef secrète pour l'API d'OVH
