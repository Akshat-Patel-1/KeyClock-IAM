# Project 3: Single Sign-On and Federation Setup

**Tools:** Keycloak 26, Docker Desktop, SAML 2.0  
**Concepts:** SSO, SAML federation, identity provider, service provider, token assertions, role propagation

---

## Objective

Configure Keycloak as a SAML 2.0 Identity Provider (IdP) and register a service provider client to simulate enterprise SSO. This mirrors real application integration work where an IAM team onboards a new application to the company's central identity platform so users authenticate once and access everything.

---

## What is SSO and why does it matter?

Without SSO, every application manages its own passwords. That means more credentials to phish, more password resets to handle, and no central place to revoke access when someone leaves. With SSO, the identity provider is the single source of truth. Revoke access in Keycloak and the user loses access to every connected application immediately.

SAML 2.0 is the protocol most enterprise applications use for SSO. Salesforce, Workday, Microsoft 365, ServiceNow, and hundreds of other platforms all support SAML federation.

---

## Architecture

```text
User → Service Provider (saml-test-app)
     → Redirected to Keycloak (IdP) for authentication
     → Keycloak issues signed SAML assertion
     → Assertion returned to Service Provider
     → User granted access based on roles in assertion
```

Keycloak acts as the IdP. The service provider trusts Keycloak to authenticate users and pass their roles through the SAML assertion.

---

## Implementation

### Step 1: Created a SAML client in Keycloak

Registered `saml-test-app` as a SAML 2.0 service provider in Keycloak with the following configuration:

- Client type: SAML
- Valid redirect URI: `http://localhost:8081/*`
- Master SAML processing URL: `http://localhost:8081/saml`
- Sign assertions: On
- Sign documents: On

Signing ensures the SAML assertion cannot be tampered with in transit. Any service provider receiving the assertion can verify it was issued by Keycloak using the public key in the IdP metadata.

### Step 2: Exported IdP metadata

Retrieved the Keycloak SAML metadata from:

```text
http://localhost:8080/realms/master/protocol/saml/descriptor
```

This XML file is what you hand to an application team when onboarding them to SSO. It contains the IdP entity ID, SSO endpoint URLs, and the signing certificate. The application loads this once and from that point trusts Keycloak for authentication.

### Step 3: Generated and validated a SAML assertion

Used Keycloak's built in client scope evaluator to generate a real SAML assertion for user `j.smith`. The assertion confirms:

- Subject: `j.smith` correctly identified
- Issuer: Keycloak master realm
- Audience: `saml-test-app`
- Roles passed in assertion: `hr-staff` inherited from HR group assignment in Project 2
- Session expiry timestamps present and correctly formatted

The `hr-staff` role appearing in the assertion proves the RBAC model from Project 2 feeds directly into the SSO token. When j.smith logs into any SAML connected application, that application receives his role and can make authorisation decisions without calling back to Keycloak.

---

## Key concepts demonstrated

**IdP initiated vs SP initiated SSO:** In this setup the service provider redirects the user to Keycloak for authentication. The user never enters credentials into the application itself.

**Role propagation:** RBAC roles defined in Keycloak are embedded in the SAML assertion and passed to the service provider. This means access control is centralised in the IdP, not duplicated across every application.

**Signed assertions:** Keycloak signs the assertion using its private key. The service provider verifies the signature using the public key from the IdP metadata. This prevents assertion forgery.

**Metadata exchange:** The trust relationship between IdP and SP is established through metadata exchange, not hardcoded credentials. This is the standard enterprise onboarding process for new SSO integrations.

---

## Production considerations

In a production deployment Keycloak would be hosted on a public domain rather than localhost. The IdP metadata URL would be registered with the service provider to complete the federation trust and allow end to end SSO login flows. Certificate rotation schedules and assertion encryption would also be configured for regulated environments.
