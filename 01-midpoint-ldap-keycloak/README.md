# Lab 1: midPoint (IGA) + OpenLDAP + Keycloak (AM)

**Goal:** see how identity governance, a directory and access management split the work in an enterprise IAM setup.

## What I did
- Ran midPoint, OpenLDAP (phpLDAPadmin) and Keycloak with Docker Compose.
- Created an identity in midPoint with no target-system assignment, and confirmed no LDAP account existed.
- Connected LDAP as a midPoint **resource** and assigned it to the user, which provisioned an `inetOrgPerson` account under `ou=people`.
- Created business roles (Admin / Manager / User) mapped to LDAP groups with inducements. Swapping Manager for Admin moved the user from `cn=managers` to `cn=admins`.
- Federated the same LDAP into a Keycloak realm (read-only) with a group → realm-role mapper, then synced users and roles.

## What I observed
| Step | Screenshot |
|---|---|
| Three admin consoles running | [01](screenshots/01-three-consoles.png) |
| Pre-existing LDAP groups | [02](screenshots/02-ldap-groups.png) |
| Identity exists in midPoint, but not yet in LDAP | [03](screenshots/03-user-not-in-ldap-yet.png) |
| Account provisioned into LDAP | [04](screenshots/04-ldap-account-provisioned.png) |
| Role swap: removed from managers, added to admins | [05](screenshots/05-removed-from-managers.png) · [06](screenshots/06-added-to-admins.png) |
| Keycloak sees the LDAP user and the group-based roles | [07](screenshots/07-keycloak-synced-user.png) · [08](screenshots/08-keycloak-realm-roles-from-ldap.png) |

## Takeaway
**Provisioning** (midPoint → LDAP) is a write: midPoint owns the account. **Federation** (Keycloak ← LDAP) is a read: Keycloak trusts LDAP and doesn't own the user. A role change in midPoint cascades into LDAP group membership, and from there into Keycloak roles.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | An account that exists in LDAP but has no owning identity in IGA (orphan) keeps working after its owner leaves. **Valid Accounts, ATT&CK T1078.** |
| **What this lab showed** | In this design every LDAP account is created by midPoint from an identity, so midPoint can always tell which identity owns it. An account created directly in LDAP would sit outside that control. |
| **What I would monitor** | LDAP accounts or group memberships created by any bind DN other than midPoint's service account. Group membership changes in privileged groups (`cn=admins`). |
| **Hardening I would add** | The lab binds as `cn=admin` with password `admin` over plain `ldap://389`. In production, use a least-privilege service account and LDAPS. |
