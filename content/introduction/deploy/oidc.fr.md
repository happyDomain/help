---
data: 2024-06-07T20:50:55+02:00
title: OpenID Connect
weight: 25
---

happyDomain supporte l'authentification d'utilisateurs via le protocole OpenID Connect. Si vous avez un prestataire pour l'authentification (Auth0, Okta, ...) ou que vous avez un logiciel dit "Identity Provider" (IdP) tel que Keycloak, Authentik, Authelia, ... vous pouvez l'utiliser avec happyDomain et vous passez, éventuellement, du système d'inscription et d'authentification embarqué.


## Configuration

Pour activer l'authentification OpenID Connect, vous aurez besoin de définir les options suivantes :

```
HAPPYDOMAIN_OIDC_PROVIDER_URL=https://auth.example.com/
HAPPYDOMAIN_OIDC_CLIENT_ID=youClientId
HAPPYDOMAIN_OIDC_CLIENT_SECRET=0a1b2c3d4e6f7A8B9C0D
```

L'option `PROVIDER_URL` correspond à l'URL de base du service d'authentification. Celui-ci doit mettre à disposition une page de découverte de ses paramètres (accessible à `/.well-known/openid-configuration`).


### Paramétrage du fournisseur OpenID Connect

Vous aurez besoin de créer une application dans votre fournisseur d'authentification, avec les paramètres suivants :

- Provider type: OIDC ou OAuth2
- Grant type: `Authorization Code`
- Application type: `Web` ou `PWA`
- Client type: `private`
- Scopes: `openid`, `profile`, `email`

Définissez également une adresse de retour autorisée :
- `https://yourHappyDomain.example.com/auth/callback`
