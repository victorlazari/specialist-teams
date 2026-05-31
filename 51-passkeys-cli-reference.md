# Passkeys CLI Reference and API Commands

## 1. WebAuthn JavaScript API Reference

### 1.1 Registration (navigator.credentials.create)

```javascript
// Full PublicKeyCredentialCreationOptions specification
const options = {
  publicKey: {
    // REQUIRED: Server-generated cryptographic challenge (min 16 bytes)
    challenge: new Uint8Array(32), // Must be crypto.getRandomValues() on server

    // REQUIRED: Relying Party information
    rp: {
      id: "example.com",        // Registrable domain (NOT full URL)
      name: "Example Corp"       // Human-readable display name
    },

    // REQUIRED: User information
    user: {
      id: new Uint8Array(64),    // Opaque user handle (NOT email, max 64 bytes)
      name: "user@example.com",  // Username/email for display
      displayName: "Jane Doe"    // Friendly name for UI
    },

    // REQUIRED: Supported algorithms (ordered by preference)
    pubKeyCredParams: [
      { type: "public-key", alg: -7 },    // ES256 (ECDSA w/ SHA-256) - RECOMMENDED
      { type: "public-key", alg: -257 },   // RS256 (RSASSA-PKCS1-v1_5 w/ SHA-256)
      { type: "public-key", alg: -8 },     // EdDSA
      { type: "public-key", alg: -35 },    // ES384
      { type: "public-key", alg: -36 }     // ES512
    ],

    // OPTIONAL: Authenticator requirements
    authenticatorSelection: {
      authenticatorAttachment: "platform",  // "platform" | "cross-platform" | undefined
      residentKey: "required",             // "required" | "preferred" | "discouraged"
      requireResidentKey: true,            // Deprecated, use residentKey instead
      userVerification: "required"         // "required" | "preferred" | "discouraged"
    },

    // OPTIONAL: Credentials to exclude (prevent duplicate registration)
    excludeCredentials: [
      {
        type: "public-key",
        id: existingCredentialId,           // ArrayBuffer of existing credential
        transports: ["internal", "hybrid"]  // Hint for authenticator selection
      }
    ],

    // OPTIONAL: Timeout in milliseconds (default varies by browser)
    timeout: 60000,

    // OPTIONAL: Attestation preference
    attestation: "none",  // "none" | "indirect" | "direct" | "enterprise"

    // OPTIONAL: Attestation formats preference (Level 3)
    attestationFormats: ["packed", "tpm"],

    // OPTIONAL: Extensions
    extensions: {
      credProps: true,           // Request credential properties
      minPinLength: true,        // Request minimum PIN length
      credBlob: new Uint8Array() // Store small blob with credential
    }
  }
};

const credential = await navigator.credentials.create(options);
```

### 1.2 Authentication (navigator.credentials.get)

```javascript
// Full PublicKeyCredentialRequestOptions specification
const options = {
  publicKey: {
    // REQUIRED: Server-generated cryptographic challenge
    challenge: new Uint8Array(32),

    // OPTIONAL: Relying Party ID (defaults to current origin's effective domain)
    rpId: "example.com",

    // OPTIONAL: Allowed credentials (omit for discoverable credentials)
    allowCredentials: [
      {
        type: "public-key",
        id: credentialId,                    // ArrayBuffer
        transports: ["internal", "hybrid"]   // Performance hint
      }
    ],

    // OPTIONAL: User verification requirement
    userVerification: "required",  // "required" | "preferred" | "discouraged"

    // OPTIONAL: Timeout
    timeout: 60000,

    // OPTIONAL: Extensions
    extensions: {
      appid: "https://legacy-u2f.example.com",  // U2F backward compatibility
      getCredBlob: true                          // Retrieve stored blob
    }
  },

  // OPTIONAL: Mediation behavior
  mediation: "conditional"  // "conditional" | "optional" | "required" | "silent"
};

const assertion = await navigator.credentials.get(options);
```

### 1.3 Capability Detection (Level 3)

```javascript
// Check if WebAuthn is supported
if (window.PublicKeyCredential) {
  console.log("WebAuthn supported");
}

// Check Conditional UI support
const conditionalSupported = await PublicKeyCredential.isConditionalMediationAvailable();

// Check User Verifying Platform Authenticator availability
const platformAvailable = await PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable();

// Get client capabilities (Level 3)
const capabilities = await PublicKeyCredential.getClientCapabilities();
// Returns: { conditionalCreate: true, conditionalGet: true, hybridTransport: true, ... }

// Signal methods (Level 3)
await PublicKeyCredential.signalUnknownCredential({ rpId: "example.com", credentialId: id });
await PublicKeyCredential.signalAllAcceptedCredentials({ rpId: "example.com", userId: uid, allAcceptedCredentialIds: [...] });
await PublicKeyCredential.signalCurrentUserDetails({ rpId: "example.com", userId: uid, name: "new@email.com", displayName: "New Name" });
```

## 2. Server-Side API Endpoints

### 2.1 Registration Endpoints

```bash
# Request registration options
curl -X POST https://api.example.com/webauthn/register/options \
  -H "Authorization: Bearer <session_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "jane.doe@example.com",
    "displayName": "Jane Doe",
    "authenticatorType": "platform"
  }'

# Response:
# {
#   "challenge": "base64url_encoded_challenge",
#   "rp": { "id": "example.com", "name": "Example Corp" },
#   "user": { "id": "base64url_user_id", "name": "jane.doe@example.com", "displayName": "Jane Doe" },
#   "pubKeyCredParams": [{ "type": "public-key", "alg": -7 }],
#   "timeout": 60000,
#   "excludeCredentials": [],
#   "authenticatorSelection": { "residentKey": "required", "userVerification": "required" }
# }

# Submit registration response
curl -X POST https://api.example.com/webauthn/register/verify \
  -H "Authorization: Bearer <session_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "id": "base64url_credential_id",
    "rawId": "base64url_raw_id",
    "type": "public-key",
    "response": {
      "attestationObject": "base64url_attestation_object",
      "clientDataJSON": "base64url_client_data",
      "transports": ["internal", "hybrid"]
    },
    "clientExtensionResults": { "credProps": { "rk": true } }
  }'
```

### 2.2 Authentication Endpoints

```bash
# Request authentication options
curl -X POST https://api.example.com/webauthn/authenticate/options \
  -H "Content-Type: application/json" \
  -d '{ "username": "jane.doe@example.com" }'

# For discoverable credentials (no username required):
curl -X POST https://api.example.com/webauthn/authenticate/options \
  -H "Content-Type: application/json" \
  -d '{}'

# Submit authentication response
curl -X POST https://api.example.com/webauthn/authenticate/verify \
  -H "Content-Type: application/json" \
  -d '{
    "id": "base64url_credential_id",
    "rawId": "base64url_raw_id",
    "type": "public-key",
    "response": {
      "authenticatorData": "base64url_auth_data",
      "clientDataJSON": "base64url_client_data",
      "signature": "base64url_signature",
      "userHandle": "base64url_user_handle"
    }
  }'
```

## 3. SimpleWebAuthn Server Commands (Node.js)

```javascript
// Installation
// npm install @simplewebauthn/server @simplewebauthn/browser

// Generate Registration Options
import { generateRegistrationOptions } from '@simplewebauthn/server';

const options = await generateRegistrationOptions({
  rpName: 'Example Corp',
  rpID: 'example.com',
  userName: 'jane.doe@example.com',
  userDisplayName: 'Jane Doe',
  attestationType: 'none',
  excludeCredentials: existingCredentials.map(cred => ({
    id: cred.credentialID,
    transports: cred.transports,
  })),
  authenticatorSelection: {
    residentKey: 'required',
    userVerification: 'required',
  },
});

// Verify Registration Response
import { verifyRegistrationResponse } from '@simplewebauthn/server';

const verification = await verifyRegistrationResponse({
  response: registrationBody,
  expectedChallenge: storedChallenge,
  expectedOrigin: 'https://example.com',
  expectedRPID: 'example.com',
});

// Generate Authentication Options
import { generateAuthenticationOptions } from '@simplewebauthn/server';

const options = await generateAuthenticationOptions({
  rpID: 'example.com',
  userVerification: 'required',
  allowCredentials: [], // Empty for discoverable credentials
});

// Verify Authentication Response
import { verifyAuthenticationResponse } from '@simplewebauthn/server';

const verification = await verifyAuthenticationResponse({
  response: authenticationBody,
  expectedChallenge: storedChallenge,
  expectedOrigin: 'https://example.com',
  expectedRPID: 'example.com',
  credential: {
    id: storedCredential.credentialID,
    publicKey: storedCredential.publicKey,
    counter: storedCredential.counter,
  },
});
```

## 4. py_webauthn Server Commands (Python)

```python
# pip install webauthn

from webauthn import generate_registration_options, verify_registration_response
from webauthn import generate_authentication_options, verify_authentication_response
from webauthn.helpers.structs import (
    AuthenticatorSelectionCriteria, ResidentKeyRequirement,
    UserVerificationRequirement, PublicKeyCredentialDescriptor
)

# Generate Registration Options
options = generate_registration_options(
    rp_id="example.com",
    rp_name="Example Corp",
    user_id=b"unique_user_id_bytes",
    user_name="jane.doe@example.com",
    user_display_name="Jane Doe",
    authenticator_selection=AuthenticatorSelectionCriteria(
        resident_key=ResidentKeyRequirement.REQUIRED,
        user_verification=UserVerificationRequirement.REQUIRED,
    ),
    exclude_credentials=[
        PublicKeyCredentialDescriptor(id=cred.credential_id)
        for cred in existing_credentials
    ],
)

# Verify Registration Response
verification = verify_registration_response(
    credential=registration_response,
    expected_challenge=stored_challenge,
    expected_origin="https://example.com",
    expected_rp_id="example.com",
)

# Generate Authentication Options
options = generate_authentication_options(
    rp_id="example.com",
    user_verification=UserVerificationRequirement.REQUIRED,
)

# Verify Authentication Response
verification = verify_authentication_response(
    credential=authentication_response,
    expected_challenge=stored_challenge,
    expected_origin="https://example.com",
    expected_rp_id="example.com",
    credential_public_key=stored_credential.public_key,
    credential_current_sign_count=stored_credential.sign_count,
)
```

## 5. Database Operations

```sql
-- PostgreSQL schema for passkey credentials
CREATE TABLE webauthn_credentials (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    credential_id BYTEA NOT NULL UNIQUE,
    public_key BYTEA NOT NULL,
    public_key_algorithm INTEGER NOT NULL DEFAULT -7,
    sign_count BIGINT NOT NULL DEFAULT 0,
    transports TEXT[] DEFAULT '{}',
    backup_eligible BOOLEAN NOT NULL DEFAULT false,
    backup_state BOOLEAN NOT NULL DEFAULT false,
    authenticator_attachment TEXT,
    aaguid UUID,
    device_name TEXT DEFAULT 'Unknown Device',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_used_at TIMESTAMPTZ,
    revoked_at TIMESTAMPTZ
);

CREATE INDEX idx_credentials_user_id ON webauthn_credentials(user_id);
CREATE INDEX idx_credentials_credential_id ON webauthn_credentials(credential_id);

-- Query: Get all active credentials for a user
SELECT * FROM webauthn_credentials
WHERE user_id = $1 AND revoked_at IS NULL
ORDER BY last_used_at DESC NULLS LAST;

-- Query: Find credential by ID for authentication
SELECT * FROM webauthn_credentials
WHERE credential_id = $1 AND revoked_at IS NULL;

-- Update: Increment sign count after successful authentication
UPDATE webauthn_credentials
SET sign_count = $2, last_used_at = NOW()
WHERE credential_id = $1;

-- Revoke: Soft-delete a credential
UPDATE webauthn_credentials
SET revoked_at = NOW()
WHERE id = $1 AND user_id = $2;
```

## 6. Testing Commands

```bash
# Chrome DevTools Protocol - Virtual Authenticator
# Enable WebAuthn in Chrome DevTools
chrome://flags/#enable-web-authentication-testing-api

# Playwright virtual authenticator setup
npx playwright test --project=chromium

# FIDO Conformance Testing Tool
# Download from: https://fidoalliance.org/certification/functional-certification/conformance/
java -jar fido2-conformance-tools.jar --rp-url https://example.com

# OpenSSL - Verify ES256 signature manually
openssl dgst -sha256 -verify public_key.pem -signature signature.bin signed_data.bin

# Generate test challenge (32 bytes, base64url)
openssl rand -base64 32 | tr '+/' '-_' | tr -d '='

# Decode CBOR attestation object (using cbor-diag tool)
echo "<base64_attestation>" | base64 -d | cbor2diag.rb
```

## 7. Environment Variables

```bash
# Server configuration
WEBAUTHN_RP_ID=example.com
WEBAUTHN_RP_NAME="Example Corp"
WEBAUTHN_RP_ORIGIN=https://example.com
WEBAUTHN_TIMEOUT=60000
WEBAUTHN_ATTESTATION=none
WEBAUTHN_USER_VERIFICATION=required
WEBAUTHN_RESIDENT_KEY=required

# Challenge storage (Redis)
REDIS_URL=redis://localhost:6379/0
CHALLENGE_TTL_SECONDS=120

# FIDO Metadata Service
FIDO_MDS_URL=https://mds3.fidoalliance.org
FIDO_MDS_TOKEN=<your_mds_access_token>
FIDO_MDS_CACHE_TTL=86400
```
