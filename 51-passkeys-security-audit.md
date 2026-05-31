# Passkeys Security Audit Guide

## 1. Threat Model

### 1.1 Threats Mitigated by Passkeys

**Phishing:** Passkeys are cryptographically bound to the RP ID (domain). Even if a user visits a convincing phishing site (`examp1e.com`), the browser will not offer credentials registered for `example.com`. The private key never leaves the authenticator, so there is nothing to steal.

**Credential Stuffing:** Passkeys eliminate shared secrets. There is no password to reuse across sites. Each credential is unique to the relying party.

**Server-Side Breach:** The server stores only public keys. If the credentials database is compromised, attackers obtain public keys that cannot be used to impersonate users. This is fundamentally different from password hashes, which can be cracked offline.

**Man-in-the-Middle:** The `origin` field in `clientDataJSON` is set by the browser and cannot be spoofed. The server verifies that the origin matches its expected value, detecting any interception.

**Replay Attacks:** Each authentication ceremony uses a fresh, single-use challenge. Replaying a captured assertion will fail because the challenge will not match.

### 1.2 Residual Threats

**Social Engineering:** An attacker may convince a user to register a passkey on the attacker's device (e.g., by gaining physical access during registration). Mitigate with session binding and out-of-band confirmation.

**Malware on Endpoint:** If the user's device is compromised with a keylogger or screen-capture malware, the attacker may be able to observe the authentication flow. However, they still cannot extract the private key from the secure enclave.

**Synced Passkey Compromise:** If a user's cloud account (Apple ID, Google Account) is compromised, the attacker gains access to all synced passkeys. Mitigate by requiring device-bound credentials for high-risk operations.

**Authenticator Vulnerabilities:** Hardware or firmware vulnerabilities in specific authenticator models may allow key extraction. Monitor the FIDO MDS for security advisories.

## 2. Registration Security Checklist

| # | Check | Risk if Missing | Priority |
|---|-------|-----------------|----------|
| 1 | Challenge is cryptographically random (≥16 bytes) | Replay attacks | CRITICAL |
| 2 | Challenge is single-use (deleted after verification) | Replay attacks | CRITICAL |
| 3 | Challenge has short TTL (60-300 seconds) | Extended attack window | HIGH |
| 4 | RP ID is correctly set to registrable domain | All credentials bound to wrong domain | CRITICAL |
| 5 | User ID is opaque (not email or username) | Information leakage | MEDIUM |
| 6 | `excludeCredentials` prevents duplicate registration | Credential confusion | MEDIUM |
| 7 | Origin is verified against expected values | MitM attacks | CRITICAL |
| 8 | Attestation is verified if required by policy | Unauthorized authenticator types | VARIES |
| 9 | User is authenticated before registering additional credentials | Credential hijacking | HIGH |
| 10 | Transports are stored for future `allowCredentials` | UX degradation | LOW |
| 11 | Backup eligibility flags (BE/BS) are stored | Cannot enforce device-bound policies | MEDIUM |
| 12 | Public key algorithm is validated against allowed list | Algorithm downgrade | HIGH |

## 3. Authentication Security Checklist

| # | Check | Risk if Missing | Priority |
|---|-------|-----------------|----------|
| 1 | Challenge is fresh and single-use | Replay attacks | CRITICAL |
| 2 | Origin matches expected value(s) | MitM, phishing | CRITICAL |
| 3 | RP ID hash in authenticatorData matches expected | Cross-site credential use | CRITICAL |
| 4 | Signature is cryptographically verified | Complete authentication bypass | CRITICAL |
| 5 | User Presence (UP) flag is set | Automated attacks without user | HIGH |
| 6 | User Verification (UV) flag is set (if required) | Authentication without biometric | HIGH |
| 7 | Sign count is validated (for device-bound credentials) | Credential cloning | MEDIUM |
| 8 | Credential is not revoked | Use of compromised credential | HIGH |
| 9 | User handle matches expected user | Account confusion | CRITICAL |
| 10 | Type field is "public-key" | Protocol confusion | LOW |

## 4. Configuration Security

### 4.1 RP ID Configuration

The RP ID MUST be set to the most restrictive value possible. If the application is served exclusively from `app.example.com`, the RP ID should be `app.example.com`, not `example.com`. Using the broader domain allows any subdomain to potentially use the credentials, expanding the attack surface.

However, if the application needs credentials to work across multiple subdomains (e.g., `app.example.com` and `admin.example.com`), the RP ID must be set to the common parent domain (`example.com`).

### 4.2 User Verification Policy

| Context | Recommended Setting | Rationale |
|---------|-------------------|-----------|
| Standard login | `required` | Ensures biometric/PIN verification |
| Step-up authentication | `required` | Must prove user presence |
| Low-risk operations | `preferred` | Allows graceful degradation |
| Never use | `discouraged` | Removes the multi-factor benefit |

### 4.3 Attestation Policy

| Context | Recommended Setting | Rationale |
|---------|-------------------|-----------|
| Consumer applications | `none` | Maximum compatibility, no privacy concerns |
| Enterprise (standard) | `none` or `indirect` | Balance of security and compatibility |
| Enterprise (high security) | `direct` | Verify authenticator provenance |
| Government/regulated | `enterprise` | Full device identification |

## 5. Data Protection

### 5.1 Stored Credential Data Classification

| Field | Sensitivity | Protection Required |
|-------|-------------|-------------------|
| credential_id | LOW | Standard database security |
| public_key | LOW | Standard database security (public by definition) |
| user_id (handle) | MEDIUM | Should be opaque, not PII |
| sign_count | LOW | Standard database security |
| transports | LOW | Standard database security |
| aaguid | LOW | Reveals authenticator model |
| created_at / last_used_at | MEDIUM | Usage patterns, encrypt at rest |

### 5.2 Data Minimization

The relying party should store only the minimum data required for authentication. Do not store the full attestation object after verification (unless required for audit). Do not store clientDataJSON after verification. Do not log raw assertion data in application logs.

### 5.3 Backup and Recovery

The credentials table is the single most critical table for authentication. If lost, ALL users lose access via passkeys. Requirements:
- Point-in-time recovery (PITR) enabled
- Cross-region replication for disaster recovery
- Regular backup verification (restore and test)
- Encrypted backups at rest and in transit
- Retention policy aligned with credential lifecycle

## 6. Token and Session Security

### 6.1 Post-Authentication Session

After successful passkey verification, the server issues a session token. This session must be:
- Bound to the client (IP address, user agent, or device fingerprint)
- Short-lived with refresh capability
- Invalidated on logout, password change, or credential revocation
- Stored securely (HttpOnly, Secure, SameSite=Strict cookies)

### 6.2 API Token Security for WebAuthn Endpoints

The registration endpoint MUST require an existing authenticated session (the user must already be logged in to add a passkey). The authentication endpoint is public by nature but must be rate-limited to prevent enumeration attacks.

## 7. Rate Limiting and Abuse Prevention

| Endpoint | Rate Limit | Rationale |
|----------|-----------|-----------|
| `/webauthn/authenticate/options` | 10 req/min per IP | Prevent challenge exhaustion |
| `/webauthn/authenticate/verify` | 5 req/min per credential | Prevent brute-force |
| `/webauthn/register/options` | 3 req/min per user | Prevent credential spam |
| `/webauthn/register/verify` | 3 req/min per user | Prevent credential spam |

## 8. Compliance Mapping

### 8.1 NIST SP 800-63B (Digital Identity Guidelines)

| AAL Level | Passkey Configuration | Requirements Met |
|-----------|----------------------|-----------------|
| AAL1 | Any passkey, UV: discouraged | Single-factor cryptographic |
| AAL2 | Passkey with UV: required | Multi-factor cryptographic (something you have + something you are) |
| AAL3 | Device-bound passkey + attestation + hardware key | Hardware-bound multi-factor with verifier impersonation resistance |

### 8.2 PCI DSS 4.0

Passkeys satisfy PCI DSS 4.0 Requirement 8.3 (multi-factor authentication) when configured with `userVerification: required`. The biometric constitutes "something you are" and the private key in the secure enclave constitutes "something you have."

### 8.3 GDPR Considerations

Passkeys are privacy-preserving by design. The credential ID is a random value that does not reveal user identity across relying parties. No biometric data leaves the device. However, the relying party must still document the processing of credential metadata (creation time, last used time, authenticator model) in their Records of Processing Activities.

## 9. Incident Response Procedures

### 9.1 Suspected Credential Compromise

1. Immediately revoke the affected credential (set `revoked_at` timestamp)
2. Notify the user via out-of-band channel (email, SMS)
3. Force re-authentication via alternative method
4. Prompt user to register new credentials
5. Review access logs for unauthorized activity during the compromise window
6. If clone detection triggered: investigate all sessions authenticated with that credential

### 9.2 Mass Credential Database Breach

1. Assess impact: public keys alone cannot be used for authentication
2. However, credential IDs and user mappings are exposed
3. Rotate all active sessions
4. Notify affected users (regulatory requirement in most jurisdictions)
5. Consider: if attestation data was stored, authenticator models are exposed
6. No need to force credential re-registration (public keys are not secrets)

### 9.3 Authenticator Model Vulnerability (FIDO MDS Advisory)

1. Query the FIDO MDS for the affected AAGUID
2. Identify all credentials in the database with the matching AAGUID
3. Assess severity based on the MDS advisory
4. For critical vulnerabilities: proactively notify affected users and prompt re-registration
5. For moderate vulnerabilities: add monitoring and require step-up for sensitive operations
