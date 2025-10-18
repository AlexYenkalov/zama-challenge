# Security

## Table of Contents

### 1. Authentication & Authorization Options
- [Options Comparison](#authentication--authorization-options)
- [Decision Reference](#decision-reference)

### 2. Multi-Factor Authentication (MFA) Options
- [MFA Options Comparison](#mfa-options-comparison)
- [Decision Reference](#decision-reference-1)

---

## 1. Authentication & Authorization Options

| **Option** | **Initial Setup** | **Cost** | **Security** | **Access Management** | **Identity Sources** | **Maintenance** | **Compliance** | **Audit Logging** |
|------------|------------------|----------|-------------|---------------------|-------------------|-----------------|-------------------|-------------|
| **Self-hosted OIDC/OAuth 2.0 with Better-Auth** | ✅ Simple (Better-Auth plugin integration) | ✅ Low (infrastructure hosting only) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP) | ✅ Simple (Better-Auth handles OIDC server, user management) | ❌  Low (no compliance certifications yet) | ❌ N/A (not in-built yet, manual implementation required) |
| **Self-hosted OIDC/OAuth 2.0 Services** | ⚠️ Medium (Docker/K8s setup, configuration, modern tooling helps) | ✅ Low (infrastructure hosting only) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP, certificate-based) | ⚠️ Medium (server maintenance, modern solutions help) | ✅ High (SOC2, GDPR, HIPAA with proper configuration) | ✅ High (comprehensive audit logs, enterprise-grade) |
| **Managed OIDC/OAuth 2.0 Services** | ✅ Simple (Auth0/AWS Cognito/Azure AD integration) | ⚠️ Medium (per-user pricing, usage-based) | ✅ High (OIDC/OAuth 2.0 standard, secure, token refresh, token expiration, enterprise-grade, compliance) | ✅ High (OAuth scopes, roles, token revocation) | ✅ High (SAML, LDAP, social login, magic links, SMS/Email OTP, certificate-based, enterprise SSO) | ✅ Simple (managed service, automatic updates) | ✅ High (SOC2, GDPR, HIPAA, PCI DSS certified) | ✅ High (built-in audit logs, compliance reporting) |
| **HMAC API Keys** | ✅ Simple (Custom implementation) | ✅ Low (infrastructure hosting only) | ✅ High (HMAC signatures, replay protection, tamper detection, non-repudiation) | ❌ Low (no user context) | ❌ N/A (no identity sources, just API keys) | ⚠️ Medium (custom key management, manual rotation) | ❌ Low (no compliance features, extensive custom work required) | ❌ N/A (manual implementation required) |
| **Better-Auth API Keys** | ✅ Simple (Better-Auth library integration) | ✅ Low (infrastructure hosting only) | ⚠️ Medium (API keys, rate limiting, expiration, no HMAC/replay protection) | ⚠️ Medium (scopes, custom metadata, user context, but API key level not full OAuth 2.0) | ❌ N/A (no identity sources, just API keys) | ✅ Simple (Better-Auth handles key management, user management) | ❌ Low (no compliance certifications yet) | ❌ N/A (not in-built yet, manual implementation required) |

### Decision Reference
*Chosen option and rationale are detailed in [07-platform-policies.md](./07-platform-policies.md)*

---

## 2. Multi-Factor Authentication (MFA) Options

| **Option** | **Initial Setup** | **Cost** | **Security** | **Maintenance** | **Compliance** | **Audit Logging** |
|------------|------------------|----------|-------------|-----------------|-------------------|-------------|
| **MFA with Email OTP** | ✅ Simple (authentication service integration) | ✅ Low (email service costs, typically included in auth services) | ⚠️ Medium (email interception, account compromise risks) | ✅ Simple (authentication services typically handle Email OTP implementation) | ❌ Low (NIST recommends against email for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with SMS OTP** | ✅ Simple (authentication service integration) | ✅ Low (SMS service costs, typically included in auth services) | ⚠️ Medium (SMS interception, SIM swapping risks) | ✅ Simple (authentication services typically handle SMS OTP implementation) | ❌ Low (NIST recommends against SMS for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with TOTP** | ✅ Simple (authentication service integration) | ✅ None (no additional infrastructure) | ✅ High (time-based OTP, cryptographic security) | ✅ Simple (authentication services typically handle TOTP implementation) | ✅ High (RFC 6238 standard, NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with Passkeys (FIDO2/WebAuthn)** | ✅ Simple (authentication service integration) | ✅ None (no additional infrastructure) | ✅ High (hardware security, phishing-resistant) | ✅ Simple (authentication services typically handle FIDO2/WebAuthn implementation) | ✅ High (FIDO2/WebAuthn standards, NIST recommended) | ✅ High (standard MFA audit logging) |
| **MFA with Push Notifications** | ✅ Simple (authentication services integration) | ✅ Low (push service costs, typically included in auth services) | ✅ High (device-based, user confirmation) | ✅ Simple (authentication services typically handle Push Notifications implementation) | ✅ High (NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |
| **MFA with Hardware Tokens** | ⚠️ Medium (hardware distribution, setup) | ⚠️ Medium (hardware costs per token) | ✅ High (hardware-based, offline security) | ⚠️ Medium (hardware management, distribution) | ✅ High (NIST recommended for high-security applications) | ✅ High (standard MFA audit logging) |

### Decision Reference
*Chosen option and rationale are detailed in [07-platform-policies.md](./07-platform-policies.md)*
