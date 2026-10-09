# Lab 7: Keycloak brokering to Microsoft Entra ID

**Goal:** have a local Keycloak delegate authentication to a cloud IdP (Entra ID), with ngrok giving Keycloak the public HTTPS URL that Entra needs for redirects.

## What I did
- Exposed the local Keycloak through ngrok and set it as Keycloak's hostname.
- Registered Keycloak as an app in Entra ID (single tenant), with redirect URI `https://<public-url>/realms/<realm>/broker/entra/endpoint`.
- Added Entra as an OpenID Connect v1.0 identity provider in Keycloak, via the tenant's discovery endpoint.
- Logged in with an Entra account. Keycloak created the federated user on first login, then issued its own tokens to an OIDC client app.

## Screenshots
[Keycloak login with an Entra option](screenshots/01-keycloak-login-with-entra.png) · [Broker settings](screenshots/02-broker-advanced-settings.png)

## Flow
```
Browser → app (8081) → Keycloak → Entra ID (user authenticates)
        ← app gets Keycloak token ← Keycloak (via ngrok URL) ← Entra redirect
```
**Authentication** happens in Entra. **Token issuance** happens in Keycloak. The **app** validates Keycloak's signature.

## Takeaway
A wrong or overly broad redirect URI is the classic way to steal authorization codes. In a real deployment I'd also turn on PKCE for the broker. It was off in this lab.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | Adding or modifying a trusted identity provider gives an attacker a backdoor login path that survives password resets. **Domain or Tenant Policy Modification: Trust Modification, T1484.002.** |
| **What this lab showed** | Adding one IdP in Keycloak was enough for an external account to log in and get a local user created automatically on first login. |
| **What I would monitor** | Keycloak admin events creating or editing identity providers, plus changes to redirect URIs on the Entra app registration. |
| **Hardening I would add** | Turn on PKCE (it was off), restrict which external users can be auto-created, and never expose a dev Keycloak on a public tunnel longer than needed. |
