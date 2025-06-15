---
data: 2024-06-07T20:50:55+02:00
title: OpenID Connect
weight: 25
---

happyDomain supports user authentication via the OpenID Connect protocol. If you have an authentication provider (Auth0, Okta, ...) or Identity Provider (IdP) software such as Keycloak, Authentik, Authelia, ... you can use it with happyDomain, and possibly dispense with the embedded registration and authentication system.


## Configuration

To enable OpenID Connect, you'll need to set the following options:

```
HAPPYDOMAIN_OIDC_PROVIDER_URL=https://auth.example.com/
HAPPYDOMAIN_OIDC_CLIENT_ID=youClientId
HAPPYDOMAIN_OIDC_CLIENT_SECRET=0a1b2c3d4e6f7A8B9C0D
```

The `PROVIDER_URL` setting should be defined to the base URL of your authentication service.
The service should expose a settings discovery endpoint (at `/.well-known/openid-configuration`).


### OpenID Connect provider settings

You'll need to setup a new application in your authentication provider, with the following settings:

- Provider type: OIDC ou OAuth2
- Grant type: `Authorization Code`
- Application type: `Web` ou `PWA`
- Client type: `private`
- Scopes: `openid`, `profile`, `email`

Also define the allowed callback URL to:
- `https://yourHappyDomain.example.com/auth/callback`
