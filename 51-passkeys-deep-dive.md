# Passkeys Deep Dive: Internals and Architecture

## 1. Cryptographic Foundations

### 1.1 Public Key Cryptography in WebAuthn

Passkeys are built on asymmetric (public key) cryptography. During registration, the authenticator generates a key pair: a private key that never leaves the secure hardware, and a public key that is sent to the relying party for storage.

During authentication, the relying party sends a challenge. The authenticator signs the challenge (along with additional context data) using the private key. The relying party verifies the signature using the stored public key. If the signature is valid, the user is authenticated.

The critical security property is that possession of the public key provides zero advantage in forging signatures. Even if the relying party's database is completely compromised, the attacker cannot impersonate users because they lack the private keys.

### 1.2 Supported Algorithms

**ES256 (ECDSA with P-256 and SHA-256):** The most widely supported algorithm. Uses the NIST P-256 elliptic curve. Key size: 256-bit private key, 512-bit public key (x + y coordinates). Signature size: ~64 bytes. Performance: fast on hardware with ECC acceleration.

**RS256 (RSASSA-PKCS1-v1_5 with SHA-256):** Legacy algorithm primarily supported for backward compatibility with older Windows Hello implementations. Key size: 2048-bit minimum. Signature size: 256 bytes. Performance: slower than ES256, larger keys and signatures.

**EdDSA (Ed25519):** Modern algorithm with excellent performance and small key/signature sizes. Not yet universally supported by all authenticators but gaining adoption. Key size: 256-bit. Signature size: 64 bytes. Performance: fastest of the three.

### 1.3 The Signature Verification Process

During authentication, the signed data is constructed as:

```
signedData = authenticatorData || SHA-256(clientDataJSON)
```

The `authenticatorData` contains the RP ID hash (proving the credential is for this domain), flags (UP, UV, BE, BS), and the sign counter. The `clientDataJSON` contains the challenge, origin, and type. By signing both together, the authenticator cryptographically binds the authentication to a specific challenge from a specific origin.

The server verifies:
1. Parse `clientDataJSON` and verify `type`, `challenge`, and `origin`
2. Compute `SHA-256(clientDataJSON)`
3. Concatenate `authenticatorData || hash`
4. Verify the signature over this concatenation using the stored public key

## 2. Authenticator Architecture

### 2.1 Platform Authenticators

Platform authenticators are built into the device's operating system and hardware. They leverage the device's secure enclave (Apple), Trusted Platform Module (Windows), or Titan M chip (Android) to generate and store private keys.

**Apple Secure Enclave:** A dedicated hardware security processor present in all modern Apple devices. Private keys are generated inside the Secure Enclave and never leave it. The Secure Enclave performs all cryptographic operations internally, only outputting the resulting signature. Even the main processor cannot access the raw key material.

**Windows TPM (Trusted Platform Module):** A dedicated security chip (or firmware-based equivalent) that stores keys in hardware-protected storage. Windows Hello uses the TPM to generate and protect passkey private keys. The TPM provides attestation capabilities that can prove the key was generated in certified hardware.

**Android Hardware Security Module:** Modern Android devices use a dedicated security chip (Titan M on Pixel, Samsung Knox Vault on Galaxy) or ARM TrustZone to protect key material. The Android Keystore API provides access to hardware-backed key storage.

### 2.2 Roaming Authenticators

Roaming authenticators are external devices that communicate with the client via USB, NFC, or BLE. The most common examples are YubiKeys and other FIDO2 security keys.

**CTAP2 Protocol:** Communication between the client (browser/OS) and the roaming authenticator uses the Client to Authenticator Protocol version 2 (CTAP2). This protocol defines commands for making credentials, getting assertions, managing PINs, and querying authenticator capabilities.

**Transport Protocols:**
- USB HID: Direct wired connection, fastest and most reliable
- NFC: Contactless, requires physical proximity (< 4cm)
- BLE: Wireless, used for hybrid/cross-device flows
- Internal: Platform authenticator (no external transport)

### 2.3 Synced vs. Device-Bound Credentials

**Device-Bound Credentials:** The private key exists on exactly one device and cannot be exported or backed up. If the device is lost, the credential is permanently lost. Provides the highest security guarantee (AAL3 eligible) but worst recovery story.

**Synced Credentials (Passkeys):** The private key is encrypted and synchronized across devices via a cloud service (iCloud Keychain, Google Password Manager, or third-party managers). Provides excellent usability (survives device loss) but the security depends on the cloud account's protection.

The `BE` (Backup Eligible) flag in authenticatorData indicates whether the credential CAN be synced. The `BS` (Backup State) flag indicates whether it IS currently backed up. These flags allow relying parties to make policy decisions based on the credential's backup status.

## 3. The Hybrid Transport Protocol (Cross-Device)

### 3.1 Protocol Overview

The hybrid transport protocol enables authentication using a smartphone when the user is signing in on a different device (e.g., a laptop). The protocol uses a combination of QR codes, BLE advertisements, and a cloud relay to establish a secure channel.

### 3.2 Flow Sequence

1. **QR Code Generation:** The client device (laptop) generates a QR code containing a one-time pairing key and the cloud relay's endpoint URL.

2. **QR Code Scanning:** The user scans the QR code with their smartphone's camera.

3. **BLE Proximity Verification:** The smartphone broadcasts a BLE advertisement. The laptop detects this advertisement, confirming that both devices are in physical proximity (preventing remote relay attacks).

4. **Cloud Relay Connection:** Both devices connect to the cloud relay service via WebSocket. The relay facilitates encrypted communication between the devices.

5. **CTAP2 Tunnel:** A CTAP2 session is established over the encrypted tunnel. The laptop sends the WebAuthn request through the tunnel to the smartphone.

6. **User Verification:** The smartphone prompts the user for biometric verification (Face ID, fingerprint).

7. **Assertion Return:** The signed assertion is sent back through the tunnel to the laptop, which forwards it to the relying party.

### 3.3 Security Properties

The hybrid protocol provides:
- **Proximity binding:** BLE ensures devices are physically close
- **End-to-end encryption:** The cloud relay cannot read the CTAP2 messages
- **One-time pairing:** Each QR code is single-use
- **No persistent pairing required:** (Unlike legacy BLE pairing)

## 4. Conditional UI (Autofill) Internals

### 4.1 Browser Implementation

When a page calls `navigator.credentials.get()` with `mediation: "conditional"`, the browser enters a special mode:

1. The browser does NOT immediately show a modal dialog.
2. Instead, it queries the credential manager for discoverable credentials matching the RP ID.
3. If matching credentials exist, they are added to the autofill dropdown of any input field with `autocomplete="username webauthn"`.
4. The Promise remains pending until the user either selects a credential from the dropdown or the page calls `abort()`.
5. When the user selects a credential, the browser prompts for biometric verification and resolves the Promise with the assertion.

### 4.2 Implementation Requirements

For Conditional UI to work correctly:
- The `navigator.credentials.get()` call must be made on page load (not in a click handler)
- Only one conditional request can be active at a time
- The input field must have the `webauthn` token in its `autocomplete` attribute
- The page must check `PublicKeyCredential.isConditionalMediationAvailable()` before attempting

### 4.3 Abort Controller Pattern

```javascript
let abortController = new AbortController();

// Start conditional UI on page load
async function startConditionalUI() {
  try {
    const assertion = await navigator.credentials.get({
      publicKey: { challenge, rpId, userVerification: "required" },
      mediation: "conditional",
      signal: abortController.signal
    });
    // User selected a passkey from autofill
    await verifyAssertion(assertion);
  } catch (e) {
    if (e.name === "AbortError") {
      // Request was aborted (user clicked explicit login button)
    }
  }
}

// If user clicks "Sign in with passkey" button explicitly
function onExplicitPasskeyClick() {
  abortController.abort(); // Cancel conditional UI
  abortController = new AbortController(); // Create new controller
  // Start modal (non-conditional) WebAuthn flow
  navigator.credentials.get({
    publicKey: { challenge, rpId, userVerification: "required" },
    signal: abortController.signal
  });
}
```

## 5. Attestation Internals

### 5.1 Attestation Object Structure

The attestation object returned during registration is a CBOR-encoded map:

```cbor
{
  "fmt": "packed",           // Attestation format
  "attStmt": {              // Attestation statement (format-specific)
    "alg": -7,             // Algorithm used to sign
    "sig": bytes,          // Signature over authData || clientDataHash
    "x5c": [bytes]         // Certificate chain (optional)
  },
  "authData": bytes         // Authenticator data (contains public key)
}
```

### 5.2 Attestation Formats

**Packed:** The most common format for FIDO2 authenticators. Can be self-attestation (no certificate, the credential key signs itself) or full attestation (includes a manufacturer certificate chain).

**TPM:** Used by Windows Hello when the device has a TPM. Includes TPM-specific structures (TPMS_ATTEST, TPMT_SIGNATURE) that prove the key was generated in a certified TPM.

**Android Key:** Used by Android devices with hardware-backed keystores. Includes a certificate chain rooted in Google's hardware attestation root CA.

**Apple:** Used by Apple devices. Includes a certificate chain rooted in Apple's WebAuthn root CA. The attestation proves the key was generated in the Secure Enclave.

**FIDO U2F:** Legacy format for backward compatibility with U2F authenticators. Contains a simple ECDSA signature and a single attestation certificate.

**None:** No attestation provided. The relying party receives no proof of the authenticator's provenance. This is the recommended default for consumer applications.

## 6. The FIDO Alliance Ecosystem

### 6.1 Standards Hierarchy

```
W3C WebAuthn (Browser API)
    ↕
FIDO2 (Umbrella specification)
    ├── CTAP2 (Client to Authenticator Protocol)
    ├── CTAP2.1 (PIN/UV, credential management)
    └── CTAP2.2 (Hybrid transport, enterprise attestation)
    
FIDO UAF (Legacy mobile biometric)
FIDO U2F (Legacy second-factor, predecessor to FIDO2)
```

### 6.2 Certification Levels

**Level 1 (L1):** Software-only implementation. No hardware security requirements. Suitable for platform authenticators on devices without secure enclaves.

**Level 2 (L2):** Restricted Operating Environment. The authenticator must run in a trusted execution environment (TEE) or equivalent isolation.

**Level 3 (L3):** Hardware-based security. The authenticator must use dedicated security hardware (secure element, TPM) for key storage and cryptographic operations.

**Level 3+ (L3+):** Enhanced hardware security with additional physical attack resistance (side-channel protection, tamper detection).

## 7. WebAuthn Level 3 New Features

### 7.1 Signal Methods

WebAuthn Level 3 introduces signal methods that allow relying parties to communicate credential state changes to the client/authenticator:

- `signalUnknownCredential()`: Tells the credential manager that a credential ID is not recognized by the RP (useful for cleaning up stale credentials)
- `signalAllAcceptedCredentials()`: Tells the credential manager which credentials the RP still recognizes for a user (allows cleanup of revoked credentials)
- `signalCurrentUserDetails()`: Updates the credential manager with the user's current name/displayName

### 7.2 PRF Extension (Pseudo-Random Function)

The PRF extension allows the relying party to derive symmetric keys from the passkey authentication process. This enables use cases like end-to-end encryption where the encryption key is derived from the passkey itself, without the relying party ever seeing the key.

### 7.3 Supplemental Public Keys

Allows an authenticator to generate additional key pairs during registration that can be used for purposes other than authentication (e.g., signing documents, encrypting data).

## 8. Performance Characteristics

### 8.1 Cryptographic Operation Timing

| Operation | Hardware (Secure Enclave) | Software (WebCrypto) |
|-----------|--------------------------|---------------------|
| ES256 Key Generation | 50-200ms | 5-20ms |
| ES256 Sign | 20-100ms | 2-10ms |
| ES256 Verify | N/A (server-side) | 1-5ms |
| RS256 Key Generation | 500-2000ms | 100-500ms |
| RS256 Sign | 50-200ms | 10-50ms |
| RS256 Verify | N/A (server-side) | 1-3ms |

### 8.2 End-to-End Latency Breakdown

| Phase | Duration | Notes |
|-------|----------|-------|
| Options request (network) | 50-200ms | Server generates challenge |
| Browser UI rendering | 100-500ms | Modal or conditional UI |
| User biometric gesture | 500-3000ms | Depends on user speed |
| Authenticator crypto | 50-200ms | Key generation or signing |
| Assertion response (network) | 50-200ms | Server verifies |
| Server verification | 1-10ms | Signature check + DB lookup |
| **Total (platform, happy path)** | **1-4 seconds** | |
| **Total (cross-device/hybrid)** | **8-30 seconds** | Includes QR scan + BLE |

## 9. Credential Lifecycle Management

### 9.1 State Machine

```
[Created] → [Active] → [Revoked]
                ↑           ↓
                └── [Suspended] (temporary disable)
```

### 9.2 Lifecycle Events

| Event | Trigger | Action |
|-------|---------|--------|
| Creation | User completes registration | Store credential, set active |
| Authentication | User signs in | Update last_used_at, increment sign_count |
| Rename | User changes device name | Update device_name |
| Suspend | Admin action or policy | Set suspended flag, reject assertions |
| Revoke | User deletes, admin action, or compromise | Set revoked_at, permanently reject |
| Cleanup | Credential unused for 12+ months | Notify user, suggest removal |

### 9.3 Credential Recovery

When a user loses access to all their passkeys (lost all devices, cloud account compromised), the relying party must provide an alternative recovery path. Common approaches:

1. **Recovery codes:** Pre-generated one-time codes stored offline by the user
2. **Email/SMS verification:** Lower security but widely available
3. **Identity verification:** Manual process involving ID documents (highest friction, highest assurance)
4. **Trusted contact:** Another verified user vouches for the account owner

The recovery flow must be carefully designed to resist social engineering while remaining accessible to legitimate users who have genuinely lost access.
