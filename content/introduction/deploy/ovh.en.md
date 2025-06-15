---
data: 2024-07-15T11:33:09+02:00
title: OVH API configuration
weight: 50
---

In order to manage domains hosted by OVH, an additional configuration step is required.

Authentication to the OVH API works in 2 stages:

1. First, you need to register an ***application*** (e.g. *happyDomain*). An application has an identifier and a secret that must be entered as a happyDomain parameter.
2. For each OVH account you wish to manage, the happyDomain interface redirects you to the OVH page for creating the *Consumer key*.

The *application* must be created from an existing OVH account, regardless of whether it has domains or not; it is a matter of identifying the person responsible for implementing the designated application.
Access to account data, and in particular to the domains they manage, is via the *Consumer key*.

For more information, see this page: <https://docs.ovh.com/gb/en/api/api-rights-delegation/#application-registration>


## Register your happyDomain instance with OVH

1. Go to <https://api.ovh.com/createApp/>.
2. Fill in the form with a name and description that reflects the name and use of your application, e.g. *happyDomain*.

Once the form has been validated, you'll get a key and a secret.
This information must be specified in the happyDomain configuration:

`ovh-application-key`
: Application key pour l'API d'OVH

`ovh-application-secret`
: Clef secrète pour l'API d'OVH
