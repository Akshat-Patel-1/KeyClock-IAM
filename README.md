# Project 1: User Lifecycle Management

**Tools:** Keycloak 26, Docker Desktop, Windows 11  
**Concepts:** Joiner/Mover/Leaver workflows, least privilege, access provisioning, audit logging

---

## What this project covers

This project simulates the day-to-day identity lifecycle work that IAM analysts handle across enterprise environments. I deployed Keycloak locally using Docker, configured a secure admin account, and ran through a full joiner, mover, and leaver workflow across three fictional departments.

---

## Environment setup

- Deployed Keycloak 26 via Docker on a local Windows machine
- Created a permanent admin account (`im-admin`) with the admin realm role assigned
- Deleted the default temporary admin account to remove standing privileged access
- Enabled user event logging for audit trail capture

This step alone reflects real hardening tasks on a new IAM platform deployment.

---

## Scenario: TechCorp Ltd

Fictional company with three departments: Engineering, HR, and Finance.

### Joiner: James Smith

Provisioned a new user account for a software engineer joining the Engineering department.

- Created user `j.smith` with appropriate email and profile attributes
- Assigned to the `Engineering` group
- Set a non-temporary password

### Mover: James Smith promoted to HR

James moved departments. Access was updated to reflect his new role.

- Removed from `Engineering` group
- Added to `HR` group
- No admin intervention required beyond group reassignment, which is how RBAC is supposed to work

### Leaver: Kai fage offboarded

Michael left the company. Rather than deleting the account immediately, I disabled it first. This is standard practice in regulated environments where access history needs to be preserved for audit purposes.

- Located `k.fage` in the user directory
- Disabled the account via the Details tab
- Account retained for audit trail, not deleted

---

## Audit logging

Event logging was enabled on the Keycloak realm to capture all identity actions. This produces a tamper-evident log of who did what and when, which is the kind of evidence compliance teams ask for during audits.

Screenshots of the event log are in the `/screenshots` folder.

---

## What this demonstrates

- Provisioning and deprovisioning user accounts in an enterprise IAM platform
- Group-based access control aligned to job roles
- Following least privilege by scoping access to department only
- Disabling rather than deleting accounts to preserve audit history
- Configuring and reading audit event logs
