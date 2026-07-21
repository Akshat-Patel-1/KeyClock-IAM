# Project 5: Privileged Access Management Basics

**Tools:** Keycloak 26, Docker Desktop  
**Concepts:** PAM, privileged account separation, service accounts, MFA for admins, least privilege

---

## Objective

Implement basic Privileged Access Management controls in Keycloak by creating a dedicated privileged service account, separating it from regular users, and enforcing stricter authentication requirements. This mirrors the account separation and credential hardening work that PAM tools like CyberArk, BeyondTrust, and HashiCorp Vault handle in enterprise environments.

---

## Why PAM matters

Privileged accounts are the highest value target for attackers. Compromising an admin account means game over for an organisation. A single stolen admin credential can lead to full domain takeover, mass data exfiltration, and ransomware deployment across an entire network.

The core PAM principle is simple: privileged access should be separate, monitored, and harder to obtain than regular access. Admins should never use their privileged account for day to day work like browsing the web or reading email.

---

## Account separation model

TechCorp Ltd uses three tiers of accounts:

| Account type | Example | Access level |
| --- | --- | --- |
| Regular user | j.smith, s.jones, k.fage | Department resources only |
| IAM admin | im-admin | Keycloak administration |
| Privileged service account | svc-admin-techcorp | Full realm admin, break glass use only |

Each tier has different authentication requirements and is subject to different audit and review processes.

---

## Implementation

### Step 1: Created a dedicated privileged service account

Created `svc-admin-techcorp` as a dedicated privileged account separate from the regular user population. This account is not tied to any individual person. It is a service account used only for break glass scenarios where normal admin access is unavailable.

- Username: `svc-admin-techcorp`
- Email: `svcadmin@techcorp.com`
- Role: admin (realm level)
- Password: strong, non-temporary credential

### Step 2: Enforced MFA directly on the privileged account

Added **Configure OTP** as a required action on `svc-admin-techcorp`. This enforces MFA at the account level rather than relying solely on realm-wide policy. Even if authentication flow policies change, this account always requires a second factor.

This is a defence in depth approach. Realm policy handles MFA for regular users. Account level enforcement handles MFA for privileged accounts as an additional layer.

### Step 3: Created a Privileged-Accounts group

Created a dedicated `Privileged-Accounts` group and added `svc-admin-techcorp` to it. This separation enables:

- Targeted audit log filtering for privileged account activity
- Separate access review cycles for admin accounts
- Policy application scoped specifically to privileged users
- Clear visual separation in the user directory

### Step 4: Documented account separation

The user list shows clear separation between regular users, the IAM admin account, and the privileged service account. In a real environment this separation would extend to separate directories, separate workstations for admin tasks, and session recording for all privileged access.

---

## Key concepts demonstrated

**Break glass accounts:** Emergency accounts used only when normal admin access fails. Credentials are stored securely, often in a sealed envelope or vault, and access is logged and reviewed after every use.

**Service account hygiene:** Service accounts should have non-expiring but strong credentials, MFA where possible, and access scoped to exactly what they need. They should never be used for interactive day to day tasks.

**Defence in depth:** MFA enforced at the realm level covers all users. MFA enforced at the account level covers privileged accounts even if realm policy is misconfigured. Two independent controls protecting the same high value target.

**Account tiering:** Separating accounts by privilege level is a foundational PAM control. It limits lateral movement if a regular user account is compromised and ensures privileged activity is visible and auditable.

---

## Production considerations

In a mature PAM environment privileged credentials would be stored in a vault such as CyberArk or HashiCorp Vault, checked out for a session, and automatically rotated after use. All privileged sessions would be recorded for forensic review. Just in time access would replace standing admin rights entirely, granting elevated access only when needed and revoking it automatically after the task is complete.

---
