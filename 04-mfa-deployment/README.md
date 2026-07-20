# Project 4: Multi-Factor Authentication Deployment

**Tools:** Keycloak 26, Docker Desktop, Google Authenticator / Microsoft Authenticator  
**Concepts:** MFA enforcement, OTP, authentication flows, credential hardening, identity threat reduction

---

## Objective

Enforce multi-factor authentication across the TechCorp Ltd realm by modifying the Keycloak browser authentication flow to require OTP as a second factor. This simulates a real MFA rollout, one of the most common and impactful identity hardening initiatives in any organisation.

---

## Why MFA matters

Passwords alone are not sufficient. Credential stuffing, phishing, and password spraying attacks succeed every day against accounts protected only by a password. MFA adds a second factor that an attacker cannot obtain just by stealing a password. Even if credentials are compromised, the account remains protected.

In regulated environments such as finance, healthcare, and government, MFA is not optional. It is a baseline control required by frameworks including ISO 27001, SOC 2, Cyber Essentials, and PCI DSS.

---

## Implementation

### Step 1: Reviewed the default browser authentication flow

Opened the built in browser flow in Keycloak and reviewed the existing authentication steps:

- Username Password Form: Required
- Browser - Conditional 2FA: Conditional (default)
- OTP Form: Required within the conditional flow

The default configuration only enforces OTP for users who have already configured it. Users without OTP set up could bypass the second factor entirely.

### Step 2: Enforced MFA for all users

Changed the **Browser - Conditional 2FA** flow from **Conditional** to **Required**. This means every user must complete a second factor regardless of whether they have previously configured one. Users without OTP set up are forced to enrol before they can access any application.

This is the difference between optional MFA and enforced MFA. Organisations that leave MFA as optional consistently see low adoption rates until a breach occurs.

### Step 3: Triggered OTP enrolment for j.smith

Added **Configure OTP** as a required action on j.smith's account. This forces OTP setup on next login without requiring an admin to be present during enrolment. The user self-serves the setup through the Keycloak account console.

### Step 4: Validated the full MFA login flow

Tested the end to end flow in an incognito browser window:

1. Navigated to the Keycloak account console
2. Entered j.smith credentials
3. Was prompted to set up Mobile Authenticator
4. Scanned the QR code using an authenticator app
5. Entered the one time code to complete enrolment
6. Successfully authenticated with MFA

The QR code setup screen confirms Keycloak is issuing TOTP secrets correctly and the authenticator app is generating valid codes against the shared secret.

---

## Key concepts demonstrated

**TOTP (Time-based One Time Password):** The OTP code changes every 30 seconds based on a shared secret and the current timestamp. Even if an attacker captures a code it is useless within seconds.

**Authentication flow modification:** Keycloak's flow system allows granular control over how users authenticate. Changing a single step from Conditional to Required is the difference between optional and enforced MFA across the entire realm.

**Required actions:** Assigning Configure OTP as a required action is how an IAM team rolls out MFA to existing users without resetting their accounts. Users are guided through self-service enrolment on their next login.

**Incognito testing:** Testing in an incognito window ensures no existing session cookies interfere with the authentication flow, giving a clean end to end validation of the new policy.

---

## Production considerations

In a production rollout MFA enforcement would typically be phased by risk level. Privileged accounts and those with access to sensitive data get enforced first, followed by the broader user population. Grace periods allow users time to enrol before enforcement kicks in. Hardware security keys such as YubiKeys would be considered for the highest risk accounts as a stronger alternative to TOTP.

---
