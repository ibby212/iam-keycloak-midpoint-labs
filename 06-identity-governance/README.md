# Lab 6: Identity governance in midPoint

**Goal:** use the controls auditors look for: access requests, approvals, Segregation of Duties (SoD), access reviews and audit trails.

## What I did
- **Requestable roles:** a role only appears in the self-service catalog once it's marked requestable.
- **Approval workflow:** added a policy rule on the *Auditor* role so that requests go to an approver (frank). One request was approved and another rejected.
- **SoD (Auditor ✕ Accountant)**, tested in all three modes:
  - *Enforcement:* the request is blocked.
  - *Approval:* the conflict is allowed only after an approver accepts it.
  - *Prune:* the new role is granted and the conflicting old role is removed.
- **Access certification:** launched a campaign over all user assignments, accepted or revoked items, closed the stage, ran remediation, and exported a report.
- **Audit log:** filtered by initiator to reconstruct which approver granted which role to whom, then exported it to CSV.

## Screenshots
[Approver work item](screenshots/01-approver-work-item.png) · [SoD conflict in cart](screenshots/02-sod-conflict-in-cart.png) · [Conflict solver](screenshots/03-sod-conflict-solver.png) · [Enforcement refusal](screenshots/04-sod-enforcement-refusal.png) · [Certification campaign](screenshots/05-certification-campaign.png) · [Audit log](screenshots/06-audit-log-filtered.png)

## Takeaway
Review and remediation are separate steps. A campaign where everything gets "accepted" without checking creates a paper trail of approving bad access, which is worse than having no campaign.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | An insider collects conflicting roles (for example Accountant + Auditor) or quietly gains a sensitive role. **Account Manipulation, T1098.** |
| **What this lab showed** | Enforcement blocked the conflicting request outright. Approval mode allowed it only with frank's sign-off. The audit log, filtered by initiator, showed exactly who approved which role for whom, and could be exported as evidence. |
| **What I would monitor** | SoD exceptions granted in approval mode, roles assigned outside the request workflow, and approvers approving their own requests. |
| **Weak point** | A single approver is a single point of compromise. If frank's account is taken over, every Auditor request gets approved. |
