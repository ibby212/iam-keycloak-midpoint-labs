# Lab 5: OIDC, SSO and Google identity brokering

**Goal:** delegate an app's login to Keycloak over OIDC, add Google as an external IdP, and get SSO across two apps.

## What I did
- Registered two confidential OIDC clients in Keycloak, one per instance of a sample app.
- Logged into app 1 through Keycloak and received an access token.
- Added **Google** as an identity provider in Keycloak, using an OAuth client created in Google Cloud Console.
- Logged into app 1, then opened app 2 and got a token **without logging in again**. That is SSO through the Keycloak session.

## Screenshots
[Keycloak login page with a Google option](screenshots/01-keycloak-login-with-google.png) · [App receives a token](screenshots/02-sample-app-token.png)

## Takeaway
The app never sees a password. It trusts tokens signed by Keycloak. Keycloak in turn can trust Google, and the user's Keycloak session is what makes the second app skip the login.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | Stealing tokens or authorization codes. **Steal Application Access Token, T1528.** One stolen SSO session gives access to every connected app. |
| **What this lab showed** | Logging into app 1 gave silent access to app 2, so the Keycloak session is a single high-value target. Social login means trusting Google's account security too (the lab enabled *Trust Email*). |
| **What I would monitor** | `CODE_TO_TOKEN` errors, logins via a new identity provider for an existing user, and sessions used from two IPs at once. |
| **Hardening I would add** | Exact redirect URIs (no wildcards like `http://localhost:8080/*`), short token lifetimes, PKCE. |
