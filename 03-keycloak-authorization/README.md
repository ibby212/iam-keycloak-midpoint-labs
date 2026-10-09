# Lab 3: Fine-grained authorization with Keycloak Authorization Services

**Goal:** model access to an app's resources (`/dashboard`, `/reports`, `/documents`) with roles, scopes, permissions and policies.

> This lab was configuration plus scenario reasoning. There was no client app to test enforcement against.

## What I did
- Created a confidential OIDC client `resource-app` with Authorization enabled.
- Created realm roles Administrator / Editor / Viewer and three users, one per role.
- Defined `read` / `write` scopes and three resources.
- Created a role-based policy (Editor + Administrator) and a scope-based permission granting `write` on Dashboard through that policy.

## Screenshots
[Client capabilities](screenshots/01-client-capabilities.png) · [Scopes](screenshots/02-scopes.png) · [Resources](screenshots/03-resources.png) · [Write permission on Dashboard](screenshots/04-write-permission-dashboard.png)

## Takeaway
Roles answer *who*. Resources and scopes answer *what* and *which action*. Policies add conditions, such as roles or time of day, and can be combined with AND/OR logic. That avoids creating a new role for every edge case ("role explosion").

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | Using a low-privilege account to reach actions it shouldn't have (for example a Viewer writing to `/dashboard`), or tampering with the client config itself. |
| **What this lab showed** | Access is decided per resource **and** per action (scope). A permission with policies attached only grants what those policies allow. |
| **What I would monitor** | Keycloak admin events changing clients, policies or permissions: a policy quietly widened to include another role is a privilege escalation. |
| **Hardening I would add** | Restrict and log admin-console access, and rotate client secrets. |
