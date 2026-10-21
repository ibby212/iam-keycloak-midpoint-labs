# IAM Labs: Keycloak, midPoint, LDAP, Entra ID

Seven Identity & Access Management labs from the **IAM Fundamentals** course at EPITA (2026). Each lab ran locally in Docker. The lab environments and instructions were provided by the course. The screenshots, observations and notes in this repo are my own.

## What's covered

| # | Lab | Stack | Key concepts |
|---|-----|-------|--------------|
| 1 | [IGA + directory + access management](01-midpoint-ldap-keycloak/) | midPoint, OpenLDAP, Keycloak | Identity vs account, provisioning, role → group mapping, LDAP federation |
| 2 | [MFA and brute-force protection](02-keycloak-mfa/) | Keycloak | TOTP MFA flow, login event logging, account lockout |
| 3 | [Fine-grained authorization](03-keycloak-authorization/) | Keycloak Authorization Services | RBAC, resources, scopes, permissions, policies |
| 4 | [Identity lifecycle (Joiner / Mover / Leaver)](04-identity-lifecycle-jml/) | HR CSV, midPoint, 2× LDAP, 2× Nextcloud | HR as source of truth, attribute-based role assignment, deprovisioning, emergency disable |
| 5 | [OIDC, SSO and social login](05-oidc-sso-federation/) | Keycloak, OIDC client app, Google | Confidential OIDC client, identity brokering, SSO across two apps |
| 6 | [Identity governance](06-identity-governance/) | midPoint | Access requests, approval workflow, Segregation of Duties, access certification, audit log |
| 7 | [Cloud federation](07-keycloak-entra-federation/) | Keycloak, Microsoft Entra ID, ngrok | Keycloak as identity broker for Entra ID over OIDC |

## Architecture across the labs

```
HR (CSV) ──► midPoint (IGA) ──► OpenLDAP ──► Nextcloud / apps
   source        decides who         stores         enforce access
   of truth      gets which role     accounts +     from LDAP groups
                                     groups
                                        │
                                        ▼
            Keycloak (AM): authenticates users, MFA, issues OIDC tokens
                                        │
                                        ▼
            External IdPs: Google, Microsoft Entra ID (brokered via OIDC)
```

## Security takeaways

- The four jobs stay separate: IGA decides access, the directory stores it, the access manager authenticates, and the app enforces. Mixing them is how orphaned and over-privileged accounts appear.
- Disabling a user in midPoint cut their app access immediately through an LDAP filter, without waiting for the HR feed. How fast a leaver is cut off is itself a control.
- MFA on its own did not stop guessing. With brute-force detection off, no number of wrong OTPs locked the account. With it on, the account locked after 3 failures and Keycloak logged `LOGIN_ERROR` events.
- Segregation of Duties has three modes: block the request, require approval, or remove the old conflicting role (prune). Each fits a different risk level.
- The midPoint audit log answers who granted an access: which approver granted which role to whom, and when.

## Identity attacks these controls address

Every lab folder ends with an **Attacker view / Defender view** section. In summary:

| Lab | Attack technique (MITRE ATT&CK) | Control I configured | Signal a SOC would watch |
|---|---|---|---|
| 1. IGA + LDAP + Keycloak | Orphaned accounts (T1078 Valid Accounts) | Every LDAP account is provisioned from an identity in midPoint | LDAP writes by anything other than midPoint's service account |
| 2. MFA | Password / OTP guessing and spraying (T1110 Brute Force) | TOTP MFA + temporary lockout after 3 failures | `LOGIN_ERROR` bursts, a lockout followed by success from a new IP |
| 3. Authorization | Privilege escalation through over-broad policies | Per-resource, per-scope permissions | Admin events changing policies or permissions |
| 4. Lifecycle (JML) | Ex-employee access (T1078 Valid Accounts) | HR-driven deprovisioning + emergency disable | Logins by disabled or departed accounts |
| 5. OIDC / SSO | Token theft (T1528 Steal Application Access Token) | Confidential clients, a central IdP session | Logins through a new IdP, a session used from two IPs |
| 6. Governance | Insider role abuse, SoD violations (T1098 Account Manipulation) | Approval workflow, SoD rules, access certification, audit log | SoD exceptions, roles granted outside the request workflow |
| 7. Entra federation | Malicious trusted IdP (T1484.002 Trust Modification) | Single-tenant app registration, a broker in Keycloak | IdP created or edited, redirect URI changes |

## Skills
Keycloak (realms, OIDC clients, authentication flows, MFA, brute-force detection, authorization services, identity brokering) · midPoint (resources, roles, inducements, auto-assignment, approvals, SoD, certification, audit) · OpenLDAP · Nextcloud LDAP integration · Microsoft Entra ID app registration · OIDC / OAuth 2.0 · Docker Compose

## Notes
- This repo contains no course handouts, configuration files or credentials. The screenshots were chosen to exclude secrets, tokens, QR codes and tenant identifiers.
