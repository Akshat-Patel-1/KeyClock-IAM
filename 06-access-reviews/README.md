# Project 6: Access Reviews and Audit Readiness

**Tools:** Keycloak 26, Docker Desktop  
**Concepts:** Access reviews, segregation of duties, toxic combinations, audit evidence, remediation workflow

---

## Objective

Conduct a structured access review across TechCorp Ltd users, identify a segregation of duties violation, remediate it, and produce audit ready evidence. This mirrors the quarterly or annual access certification process that compliance teams run in regulated environments.

---

## Why access reviews matter

Access accumulates over time. Users change roles, get promoted, move departments, and pick up permissions along the way that nobody ever removes. Left unchecked this creates access sprawl where people have far more access than their current job requires.

Access reviews are the control that catches this. Frameworks including ISO 27001, SOC 2, PCI DSS, and Cyber Essentials all require periodic reviews of who has access to what and evidence that excessive access was removed.

---

## Access review findings

Reviewed all user accounts in the TechCorp Ltd realm against their expected access based on department and role.

| User | Department | Expected role | Actual roles found | Finding |
| ------ | ------------ | --------------- | ------------------- | --------- |
| j.smith | HR | hr-staff | hr-staff, finance-staff | Violation |
| s.jones | HR | hr-staff | hr-staff | Clean |
| k.fage | Finance | finance-staff | finance-staff | Clean |
| svc-admin-techcorp | Privileged | admin | admin | Clean |
| im-admin | IAM | admin | admin | Clean |

---

## Violation identified: Segregation of duties breach

**User:** j.smith  
**Finding:** finance-staff role assigned in addition to hr-staff  
**Risk:** An HR employee with Finance access creates a toxic combination. This individual could potentially view or manipulate financial data they have no business reason to access. In a regulated environment this would breach segregation of duties controls and could result in audit findings or regulatory penalties.  
**Severity:** High  
**Action required:** Immediate removal of finance-staff role

---

## Remediation

Removed the `finance-staff` role from j.smith during the access review. Post-remediation role mapping confirms j.smith now holds only `hr-staff`, which is appropriate for their department and job function.

Root cause: Role was directly assigned outside of the group-based RBAC model, bypassing the access control design established in Project 2. This highlights the importance of enforcing role assignment exclusively through groups rather than allowing direct user role assignments.

**Recommendation:** Disable direct role assignment for non-admin users and enforce all access through group membership. This prevents one-off assignments that bypass the RBAC model and makes access reviews simpler by ensuring the group structure always reflects actual access.

---

## Audit evidence produced

All actions are captured in the Keycloak event log providing a tamper-evident audit trail:

- Role assignment event showing when finance-staff was added to j.smith
- Role unassignment event showing when finance-staff was removed
- Timestamps on all events for chronological audit trail

This evidence pack would satisfy an auditor asking to see proof that access reviews are conducted and that findings are remediated in a timely manner.

---

## Key concepts demonstrated

**Segregation of duties:** No single user should hold access that creates a conflict of interest or bypasses internal controls. HR and Finance access in the same account is a classic SoD violation.

**Toxic combinations:** Certain role pairings are inherently risky regardless of intent. Identifying and preventing these combinations is a core IAM control.

**Audit trail:** Every access change is logged with a timestamp and actor. This is not optional in regulated environments. If you cannot prove who changed what and when, the control does not exist from an auditor's perspective.

**Remediation workflow:** Spot the violation, document it, remove the access, verify the removal, log the evidence. This five step process is repeatable and auditable.

---

## Production considerations

In a mature environment access reviews would be automated using an IGA (Identity Governance and Administration) platform such as SailPoint, Saviynt, or Microsoft Entra Identity Governance. The system would automatically flag toxic combinations, route certification tasks to managers, and record decisions without manual IAM team involvement. Reviews would run quarterly for standard users and monthly for privileged accounts.

---