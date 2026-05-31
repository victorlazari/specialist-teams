# Passkeys Configuration Schemas Reference

## 1. PublicKeyCredentialCreationOptions Schema

The complete JSON schema for the registration options object:

```json
{
  "publicKey": {
    "rp": {
      "id": "string (registrable domain, e.g., 'example.com')",
      "name": "string (human-readable RP name, e.g., 'Example Corp')"
    },
    "user": {
      "id": "BufferSource (max 64 bytes, opaque user handle)",
      "name": "string (username or email for display)",
      "displayName": "string (friendly name, e.g., 'Jane Doe')"
    },
    "challenge": "BufferSource (min 16 bytes, cryptographically random)",
    "pubKeyCredParams": [
      { "type": "public-key", "alg": -7 },
      { "type": "public-key", "alg": -257 },
      { "type": "public-key", "alg": -8 }
    ],
    "timeout": "unsigned long (milliseconds, recommended: 60000-300000)",
    "excludeCredentials": [
      {
        "type": "public-key",
        "id": "BufferSource (credential ID)",
        "transports": ["internal", "hybrid", "usb", "ble", "nfc"]
      }
    ],
    "authenticatorSelection": {
      "authenticatorAttachment": "platform | cross-platform | (omit for any)",
      "residentKey": "required | preferred | discouraged",
      "requireResidentKey": "boolean (deprecated, use residentKey)",
      "userVerification": "required | preferred | discouraged"
    },
    "attestation": "none | indirect | direct | enterprise",
    "attestationFormats": ["packed", "tpm", "android-key", "apple", "fido-u2f", "none"],
    "extensions": {
      "credProps": true,
      "minPinLength": true,
      "credBlob": "BufferSource (max 32 bytes)",
      "largeBlob": { "support": "required | preferred" },
      "prf": { "eval": { "first": "BufferSource", "second": "BufferSource" } }
    }
  }
}
```

## 2. PublicKeyCredentialRequestOptions Schema

```json
{
  "publicKey": {
    "challenge": "BufferSource (min 16 bytes, cryptographically random)",
    "rpId": "string (registrable domain, defaults to current origin's effective domain)",
    "timeout": "unsigned long (milliseconds)",
    "allowCredentials": [
      {
        "type": "public-key",
        "id": "BufferSource (credential ID)",
        "transports": ["internal", "hybrid", "usb", "ble", "nfc"]
      }
    ],
    "userVerification": "required | preferred | discouraged",
    "extensions": {
      "appid": "string (legacy U2F AppID for backward compatibility)",
      "getCredBlob": true,
      "largeBlob": { "read": true },
      "prf": { "eval": { "first": "BufferSource", "second": "BufferSource" } }
    }
  },
  "mediation": "conditional | optional | required | silent"
}
```

## 3. Credential Storage Schema (Database)

### 3.1 PostgreSQL Schema

```sql
CREATE TABLE webauthn_credentials (
    -- Primary key
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Foreign key to users table
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- WebAuthn credential data
    credential_id BYTEA NOT NULL,
    public_key BYTEA NOT NULL,
    public_key_algorithm INTEGER NOT NULL DEFAULT -7,
    sign_count BIGINT NOT NULL DEFAULT 0,
    
    -- Transport hints
    transports TEXT[] NOT NULL DEFAULT '{}',
    
    -- Backup state (WebAuthn Level 2+)
    backup_eligible BOOLEAN NOT NULL DEFAULT false,
    backup_state BOOLEAN NOT NULL DEFAULT false,
    
    -- Authenticator metadata
    authenticator_attachment TEXT CHECK (authenticator_attachment IN ('platform', 'cross-platform')),
    aaguid UUID,
    
    -- User-facing metadata
    device_name TEXT NOT NULL DEFAULT 'Unknown Device',
    
    -- Lifecycle timestamps
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_used_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ,
    
    -- Constraints
    CONSTRAINT unique_credential_id UNIQUE (credential_id),
    CONSTRAINT valid_algorithm CHECK (public_key_algorithm IN (-7, -8, -35, -36, -257, -258, -259))
);

-- Performance indexes
CREATE INDEX idx_cred_user_id ON webauthn_credentials(user_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_cred_credential_id ON webauthn_credentials(credential_id) WHERE revoked_at IS NULL;
CREATE INDEX idx_cred_last_used ON webauthn_credentials(last_used_at) WHERE revoked_at IS NULL;
```

### 3.2 MongoDB Schema

```javascript
{
  $jsonSchema: {
    bsonType: "object",
    required: ["userId", "credentialId", "publicKey", "algorithm", "signCount", "createdAt"],
    properties: {
      userId: { bsonType: "objectId" },
      credentialId: { bsonType: "binData" },
      publicKey: { bsonType: "binData" },
      algorithm: { bsonType: "int", enum: [-7, -8, -35, -36, -257, -258, -259] },
      signCount: { bsonType: "long", minimum: 0 },
      transports: { bsonType: "array", items: { bsonType: "string" } },
      backupEligible: { bsonType: "bool" },
      backupState: { bsonType: "bool" },
      authenticatorAttachment: { bsonType: "string", enum: ["platform", "cross-platform"] },
      aaguid: { bsonType: "string" },
      deviceName: { bsonType: "string" },
      createdAt: { bsonType: "date" },
      lastUsedAt: { bsonType: "date" },
      revokedAt: { bsonType: "date" }
    }
  }
}
```

## 4. COSE Key Format Reference

### 4.1 ES256 (ECDSA with P-256 and SHA-256)

```cbor
{
  1: 2,        // kty: EC2
  3: -7,       // alg: ES256
  -1: 1,       // crv: P-256
  -2: bytes,   // x-coordinate (32 bytes)
  -3: bytes    // y-coordinate (32 bytes)
}
```

### 4.2 RS256 (RSASSA-PKCS1-v1_5 with SHA-256)

```cbor
{
  1: 3,        // kty: RSA
  3: -257,     // alg: RS256
  -1: bytes,   // n (modulus, 256 bytes for RSA-2048)
  -2: bytes    // e (exponent, typically 3 bytes: 01 00 01)
}
```

### 4.3 EdDSA (Ed25519)

```cbor
{
  1: 1,        // kty: OKP
  3: -8,       // alg: EdDSA
  -1: 6,       // crv: Ed25519
  -2: bytes    // x (public key, 32 bytes)
}
```

## 5. AuthenticatorData Format

```
+------------------+------+------+------+------+------------------+------------------+
| rpIdHash (32B)   | flags| signCount    | attestedCredData   | extensions (CBOR) |
+------------------+------+------+------+------+------------------+------------------+

Flags byte (bit field):
  Bit 0 (UP): User Present
  Bit 2 (UV): User Verified
  Bit 3 (BE): Backup Eligible
  Bit 4 (BS): Backup State
  Bit 6 (AT): Attested Credential Data present
  Bit 7 (ED): Extension Data present

Attested Credential Data (only in registration):
  +----------+-------------------+-------------------+
  | aaguid   | credentialIdLen   | credentialId      | credentialPublicKey (COSE) |
  | (16B)    | (2B, big-endian)  | (variable)        | (variable CBOR)            |
  +----------+-------------------+-------------------+
```

## 6. ClientDataJSON Schema

```json
{
  "type": "webauthn.create | webauthn.get",
  "challenge": "base64url-encoded challenge",
  "origin": "https://example.com",
  "crossOrigin": false,
  "tokenBinding": {
    "status": "present | supported | not-supported",
    "id": "base64url-encoded token binding ID"
  }
}
```

## 7. Server Configuration Templates

### 7.1 Node.js (SimpleWebAuthn)

```javascript
// config/webauthn.js
module.exports = {
  rpName: process.env.WEBAUTHN_RP_NAME || 'My Application',
  rpID: process.env.WEBAUTHN_RP_ID || 'example.com',
  origin: process.env.WEBAUTHN_ORIGIN || 'https://example.com',
  challengeTTL: parseInt(process.env.CHALLENGE_TTL || '120', 10),
  timeout: parseInt(process.env.WEBAUTHN_TIMEOUT || '60000', 10),
  attestation: process.env.WEBAUTHN_ATTESTATION || 'none',
  userVerification: process.env.WEBAUTHN_UV || 'required',
  residentKey: process.env.WEBAUTHN_RK || 'required',
  algorithms: [-7, -257], // ES256, RS256
};
```

### 7.2 Python (py_webauthn)

```python
# config/webauthn.py
import os

WEBAUTHN_CONFIG = {
    "rp_id": os.getenv("WEBAUTHN_RP_ID", "example.com"),
    "rp_name": os.getenv("WEBAUTHN_RP_NAME", "My Application"),
    "origin": os.getenv("WEBAUTHN_ORIGIN", "https://example.com"),
    "challenge_ttl": int(os.getenv("CHALLENGE_TTL", "120")),
    "timeout": int(os.getenv("WEBAUTHN_TIMEOUT", "60000")),
    "attestation": os.getenv("WEBAUTHN_ATTESTATION", "none"),
    "user_verification": os.getenv("WEBAUTHN_UV", "required"),
    "resident_key": os.getenv("WEBAUTHN_RK", "required"),
}
```

### 7.3 Go (go-webauthn)

```go
// config/webauthn.go
package config

import (
    "github.com/go-webauthn/webauthn/webauthn"
)

func NewWebAuthnConfig() *webauthn.Config {
    return &webauthn.Config{
        RPDisplayName: getEnv("WEBAUTHN_RP_NAME", "My Application"),
        RPID:          getEnv("WEBAUTHN_RP_ID", "example.com"),
        RPOrigins:     []string{getEnv("WEBAUTHN_ORIGIN", "https://example.com")},
        Timeouts: webauthn.TimeoutsConfig{
            Login: webauthn.TimeoutConfig{
                Enforce: true,
                Timeout: 60 * time.Second,
            },
            Registration: webauthn.TimeoutConfig{
                Enforce: true,
                Timeout: 60 * time.Second,
            },
        },
    }
}
```

## 8. Well-Known Files

### 8.1 Related Origins (WebAuthn Level 3)

File: `https://<rp-id>/.well-known/webauthn`

```json
{
  "origins": [
    "https://login.example.com",
    "https://auth.example.com",
    "https://app.example.com"
  ]
}
```

### 8.2 Apple App Site Association

File: `https://<rp-id>/.well-known/apple-app-site-association`

```json
{
  "webcredentials": {
    "apps": [
      "TEAMID.com.example.myapp"
    ]
  }
}
```

### 8.3 Android Digital Asset Links

File: `https://<rp-id>/.well-known/assetlinks.json`

```json
[
  {
    "relation": ["delegate_permission/common.get_login_creds"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.myapp",
      "sha256_cert_fingerprints": [
        "AA:BB:CC:DD:EE:FF:..."
      ]
    }
  }
]
```

## 9. FIDO Metadata Service (MDS) Response Schema

```json
{
  "aaguid": "00000000-0000-0000-0000-000000000000",
  "description": "Authenticator Model Name",
  "authenticatorVersion": 5,
  "protocolFamily": "fido2",
  "schema": 3,
  "upv": [{ "major": 1, "minor": 1 }],
  "authenticationAlgorithms": ["secp256r1_ecdsa_sha256_raw"],
  "publicKeyAlgAndEncodings": ["cose"],
  "attestationTypes": ["basic_full"],
  "userVerificationDetails": [
    [{ "userVerificationMethod": "fingerprint_internal" }],
    [{ "userVerificationMethod": "passcode_internal" }]
  ],
  "keyProtection": ["hardware", "secure_element"],
  "matcherProtection": ["on_chip"],
  "cryptoStrength": 128,
  "attachmentHint": ["internal"],
  "tcDisplay": [],
  "attestationRootCertificates": ["MIIBfz..."],
  "statusReports": [
    {
      "status": "FIDO_CERTIFIED_L2",
      "effectiveDate": "2024-01-15",
      "certificationDescriptor": "FIDO2 L2"
    }
  ]
}
```
