# Lab 4: Identity lifecycle (Joiner / Mover / Leaver)

**Goal:** have HR data drive access end to end: HR CSV → midPoint → LDAP (Finance / R&D) → Nextcloud (Finance / R&D).

## What I did
- Connected two Nextcloud instances to two LDAP directories and shared folders by group:
  - `fin-employee` → Accounting, `fin-manager` → Accounting + Budget
  - `rd-employee` → Knowledge-base, `rd-manager` → Knowledge-base + Projects
- Added the Nextcloud LDAP filter `(&(objectClass=inetOrgPerson)(!(midPointAccountStatus=disabled)))` so that accounts disabled in midPoint can't log in.
- Imported the LDAP resources and four business roles into midPoint and enabled **auto-assignment** on `title` + `organizationalUnit`.
- Added the HR CSV as an inbound resource, with an import task and a live-sync task.
- Ran the lifecycle by editing the CSV:
  - **Joiner:** new line → role auto-assigned, LDAP account created, folders visible.
  - **Mover:** department changed → account moved from one LDAP to the other.
  - **Promotion:** title changed to Manager → extra folder.
  - **Leaver:** line removed → account deprovisioned.
- **Emergency disable:** disabled the user directly in midPoint, and Nextcloud login failed with no HR change needed.

## Screenshots
[Finance LDAP groups](screenshots/01-ldap-fin-groups.png) · [Folder share permissions](screenshots/02-accounting-share-permissions.png) · [Account provisioned by midPoint](screenshots/03-midpoint-provisioned-ldap-account.png) · [Users imported from HR](screenshots/04-hr-csv-import-users.png) · [HR-sourced user in Nextcloud](screenshots/05-nextcloud-as-hr-user.png)

## Takeaway
Once HR is the source of truth, access follows the job automatically. The flip side is that **bad HR data becomes bad access everywhere**. The emergency disable path exists because HR updates are too slow for incidents.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | Using a former employee's still-active account, or an account that kept old access after a move. **Valid Accounts, T1078.** |
| **What this lab showed** | Leaver: removing the HR record deprovisioned the LDAP account. Mover: the account moved to the new department's directory and lost the old folders. Emergency disable cut Nextcloud access through the `midPointAccountStatus` filter, without waiting for HR. |
| **What I would monitor** | Logins by accounts flagged disabled or departed. Gaps between an HR leave date and the actual deprovisioning time. |
| **Weak point** | HR is trusted blindly. Anyone who can edit the HR feed can grant themselves a manager role, and the automation will provision it. The HR source needs the same protection as an admin console. |
