# Passkeys Advanced Patterns and Enterprise Architecture

## 1. Multi-Tenant Passkey Architecture

In multi-tenant SaaS applications, passkey architecture must account for the relationship between tenants, domains, and RP IDs. The RP ID is bound to a single registrable domain, meaning all tenants sharing a domain (e.g., `tenant1.app.com`, `tenant2.app.com`) will share the same RP ID (`app.com`). This has critical implications for credential isolation.

### 1.1 Tenant Isolation Strategies

If tenants are accessed via subdomains of a shared domain, passkeys registered under one tenant are technically valid for all tenants (since they share the RP ID). The relying party must implement application-level isolation by associating each credential with both a user ID and a tenant ID, and rejecting assertions where the credential's tenant does not match the requested tenant.

If tenants have custom domains (e.g., `login.customer1.com`), each custom domain constitutes a separate RP ID, providing natural cryptographic isolation. However, this means a user who accesses multiple tenants must register separate passkeys for each custom domain.

### 1.2 Related Origins (WebAuthn Level 3)

WebAuthn Level 3 introduces the "related origins" concept, allowing a relying party to specify additional origins that are authorized to use credentials registered under the primary RP ID. This is configured via a `.well-known/webauthn` file hosted at the RP ID's domain. This feature enables SSO-like experiences where a passkey registered at `example.com` can be used to authenticate at `auth.example.com` or `login.example.com`.

## 2. Passkeys with Risk-Based Authentication

Advanced deployments combine passkeys with risk-based authentication (RBA) engines to dynamically adjust security requirements based on contextual signals.

### 2.1 Signal Collection

During a passkey authentication, the relying party collects contextual signals including: the IP address and geolocation, the device fingerprint, the time of day, the authenticator model (via AAGUID), whether the credential is synced or device-bound (via BE/BS flags), and the user's historical authentication patterns.

### 2.2 Dynamic Policy Enforcement

Based on these signals, the RBA engine can make real-time decisions:

If the user is authenticating from a recognized device, at a normal time, from a familiar location, the passkey assertion alone is sufficient. If the user is authenticating from a new device or unusual location, the system may require step-up authentication (a second passkey assertion with `userVerification: required`) or trigger an out-of-band verification (push notification to a registered device).

If the RBA engine detects indicators of compromise (e.g., the assertion originates from a known malicious IP, or the authenticator model has a known vulnerability published in the FIDO MDS), the system can reject the authentication entirely and lock the account pending manual review.

## 3. Passkeys in Zero Trust Architecture

In a Zero Trust security model, no user or device is inherently trusted, regardless of network location. Passkeys serve as a foundational component of Zero Trust by providing continuous, phishing-resistant identity verification.

### 3.1 Continuous Authentication

Traditional session-based authentication verifies identity once and then trusts the session cookie for the duration. Zero Trust architectures require periodic re-verification, particularly before accessing sensitive resources.

Passkeys enable frictionless continuous authentication because the biometric gesture is fast and familiar. Applications can silently request a passkey assertion at defined intervals (e.g., every 30 minutes) or before specific actions, without significantly disrupting the user's workflow.

### 3.2 Device Trust Integration

In enterprise Zero Trust deployments, passkey authentication is often combined with device trust signals from MDM (Mobile Device Management) or EDR (Endpoint Detection and Response) solutions. The authentication decision considers not only the passkey assertion but also whether the device is managed, compliant with security policies (encrypted, patched, no jailbreak detected), and free of active threats.

## 4. Advanced Attestation Workflows

### 4.1 Enterprise Attestation

Enterprise attestation is a special attestation mode available on managed devices. When enabled via MDM policy, the authenticator includes the device's unique identifier in the attestation statement, allowing the relying party to verify that the credential was created on a specific, organization-managed device.

This is particularly valuable for organizations that need to maintain an inventory of authorized authenticators and ensure that credentials are only created on approved hardware.

### 4.2 Attestation Verification Pipeline

For relying parties that enforce attestation, the verification pipeline must:

1. Extract the attestation format from the attestation object (packed, tpm, android-key, apple, fido-u2f, none).
2. Parse the attestation statement according to the format-specific rules defined in the WebAuthn specification.
3. Verify the attestation signature using the appropriate trust anchor (manufacturer root certificate, FIDO MDS root, or Apple WebAuthn root CA).
4. Validate the certificate chain, checking for revocation via CRL or OCSP.
5. Extract the AAGUID and query the FIDO Metadata Service to verify the authenticator's certification level and security status.
6. Apply organizational policy (e.g., reject if certification level is below L2, or if the authenticator model has a published vulnerability).

## 5. Passkeys and Account Linking

### 5.1 The Duplicate Account Problem

When organizations introduce passkeys alongside existing authentication methods, they must handle the scenario where a user has multiple accounts (e.g., one created with email/password and another created via social login). Passkey registration must be integrated with account linking workflows to prevent credential fragmentation.

### 5.2 Linking Strategy

The recommended approach is to require identity verification before allowing passkey registration. If a user authenticates via a social provider (Google, Apple) and then attempts to register a passkey, the system should first verify that the social identity is linked to the correct internal account. This prevents scenarios where a user accidentally registers a passkey against the wrong account.

## 6. Passkeys for Machine-to-Machine Authentication

While passkeys are primarily designed for human authentication (requiring biometric verification), emerging patterns extend the concept to machine-to-machine (M2M) scenarios using the underlying FIDO2 infrastructure.

### 6.1 Hardware-Bound Service Credentials

In high-security environments, service accounts can be authenticated using hardware security modules (HSMs) or TPMs that implement the CTAP protocol. The service's private key is stored in the HSM, and authentication is performed programmatically without human interaction. This provides the same phishing-resistance and non-exportability guarantees as human passkeys, but for automated systems.

### 6.2 Attestation for Supply Chain Security

The attestation mechanism can be repurposed for software supply chain security. Build systems can use hardware-bound credentials to sign build artifacts, providing cryptographic proof that the artifact was produced on a specific, trusted build machine with a verified secure enclave.

## 7. Performance Benchmarks

Based on production deployments, the following performance characteristics are typical:

| Operation | Latency (P50) | Latency (P95) | Notes |
|-----------|---------------|---------------|-------|
| Registration (platform authenticator) | 2-4 seconds | 6-8 seconds | Includes biometric prompt |
| Authentication (platform, conditional UI) | 1-2 seconds | 3-5 seconds | Fastest path |
| Authentication (cross-device/hybrid) | 8-15 seconds | 20-30 seconds | Includes QR scan + BLE |
| Server-side verification | 1-3 ms | 5-10 ms | ES256 signature verification |
| Challenge generation + storage | < 1 ms | 2-3 ms | Redis-backed |
| Credential lookup by ID | 1-2 ms | 5-8 ms | Indexed PostgreSQL |

## 8. Disaster Recovery and Business Continuity

### 8.1 IdP Outage Scenarios

If the organization's Identity Provider experiences an outage, passkey authentication will fail for applications that delegate authentication. Mitigation strategies include maintaining a local authentication fallback (direct WebAuthn endpoints that bypass the IdP) or implementing IdP redundancy with automatic failover.

### 8.2 Cloud Keychain Outage

If Apple iCloud, Google Password Manager, or another cloud keychain provider experiences an outage, users with synced passkeys may be unable to authenticate. The relying party cannot control this scenario but must ensure that fallback authentication methods (password, backup codes, hardware keys) remain available during such events.

### 8.3 Database Recovery

If the credentials database is corrupted or lost, all registered passkeys become permanently unusable (the server can no longer verify assertions without the stored public keys). This makes the credentials table one of the most critical datasets in the application. It must be included in all backup strategies, with point-in-time recovery capability and regular backup verification testing.
