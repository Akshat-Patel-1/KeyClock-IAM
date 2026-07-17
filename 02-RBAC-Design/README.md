# Project 2: Role-Based Access Control Design

**Tools:** Keycloak 26, Docker Desktop  
**Concepts:** RBAC, least privilege, access restriction, sensitive data classification, audit readiness

---

## Objective

Design and implement a role-based access control model for TechCorp Ltd that gives each department exactly the access they need and nothing more. This reduces access sprawl and limits the blast radius if any account is compromised.

---

## Role Design

| Role | Department | Access | Sensitive Data |
| ------ | ------------ | -------- | ---------------- |
| engineer | Engineering | Code repositories, cloud infrastructure, deployment pipelines | Yes |
| hr-staff | HR | Employee records, payroll system, recruitment tools | Yes |
| finance-staff | Finance | Financial reporting, invoice management, budget tools | Yes |

All three roles touch sensitive data and therefore require MFA enforcement in a production environment.

---

## Access Restriction Matrix

This defines what each role explicitly cannot access. In a real audit this is as important as what they can access.

| Role | Blocked From |
| ------ | ------------- |
| engineer | Payroll, employee records, invoices, budgets |
| hr-staff | Code repositories, cloud infrastructure, deployment pipelines, financial reporting |
| finance-staff | Code repositories, employee records, payroll, deployment pipelines |

No cross-department access is permitted. A finance employee cannot view HR records. An engineer cannot view payroll. This is least privilege applied at the role level.

---

## Design Decisions

**Why roles instead of direct user assignment?**  
Assigning permissions directly to users creates an unmanageable mess at scale. When someone changes departments you update one role assignment, not twenty individual permissions. Keycloak groups map users to roles, roles map to permissions. Clean separation.

**Why flag sensitive data access?**  
Roles that touch sensitive data need stricter controls: mandatory MFA, shorter session timeouts, more frequent access reviews. Flagging it at the design stage means those controls get built in rather than bolted on later.

**Why document what roles cannot access?**  
Auditors do not just check that the right people have access. They check that the wrong people do not. Having an explicit restriction matrix shows the access model was thought through, not just clicked together.

---

## Implementation in Keycloak

Roles created in Keycloak master realm:

- `engineer`
- `hr-staff`
- `finance-staff`

Each role assigned to the corresponding department group. Users inherit role permissions through group membership, not direct assignment.

---
