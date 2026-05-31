# Passkeys Troubleshooting Guide

## 1. Registration Failures

### 1.1 NotAllowedError: The operation either timed out or was not allowed

**Cause:** This is the most common error and has multiple root causes.

**Diagnosis Steps:**
1. Check if the WebAuthn call was triggered by a user gesture. Safari strictly requires a click/tap event to precede `navigator.credentials.create()`. Programmatic calls without user interaction will be rejected.
2. Verify the RP ID matches the current domain. If the page is served from `auth.example.com` but the RP ID is set to `other.com`, the browser will reject the operation.
3. Check if the user cancelled the biometric prompt. This is not an error condition but must be handled gracefully.
4. Verify the timeout has not expired. Default timeouts vary by browser (Chrome: 120s, Safari: 60s, Firefox: 120s).
5. Check if `excludeCredentials` contains a credential already present on the authenticator. The browser will throw this error to prevent duplicate registration.

**Resolution:**
- Ensure all WebAuthn calls are within a click event handler
- Verify RP ID configuration matches the serving domain
- Implement retry logic with a clear "Try Again" button
- Increase timeout for cross-device flows

### 1.2 SecurityError: The RP ID is not a registrable domain suffix of the current origin

**Cause:** The RP ID does not match the page's origin. The RP ID must be equal to or a registrable domain suffix of the page's effective domain.

**Examples:**
- Page at `https://login.example.com` with RP ID `example.com` → VALID (registrable domain suffix)
- Page at `https://example.com` with RP ID `example.com` → VALID (exact match)
- Page at `https://example.com` with RP ID `login.example.com` → INVALID (RP ID is more specific)
- Page at `https://example.com` with RP ID `other.com` → INVALID (different domain)
- Page at `http://example.com` (HTTP) → INVALID (WebAuthn requires HTTPS, except localhost)

**Resolution:** Correct the RP ID to match the registrable domain. Remember: you cannot change the RP ID after credentials are registered without invalidating all existing passkeys.

### 1.3 InvalidStateError: The authenticator was previously registered

**Cause:** The `excludeCredentials` list contains a credential ID that already exists on the authenticator being used. This is the intended behavior to prevent duplicate registration.

**Resolution:** Inform the user that they already have a passkey registered on this device. Offer to proceed to authentication instead, or allow them to manage existing credentials.

### 1.4 NotSupportedError: The algorithm is not supported

**Cause:** The `pubKeyCredParams` array contains only algorithms that the authenticator does not support.

**Resolution:** Always include ES256 (`alg: -7`) as it is universally supported by all FIDO2 authenticators. Include RS256 (`alg: -257`) as a fallback for older Windows Hello implementations.

## 2. Authentication Failures

### 2.1 No Passkey Available in Autofill (Conditional UI)

**Cause:** Conditional UI is not triggering, and no passkeys appear in the autofill dropdown.

**Diagnosis Steps:**
1. Verify `mediation: "conditional"` is set in the `navigator.credentials.get()` call.
2. Ensure the call is made on page load (not inside a click handler for conditional UI).
3. Check that the input field has `autocomplete="username webauthn"` attribute.
4. Verify the browser supports Conditional UI: `PublicKeyCredential.isConditionalMediationAvailable()`.
5. Confirm passkeys exist for this RP ID in the user's credential manager.
6. Check that no other `navigator.credentials.get()` call is already pending (only one can be active).

**Resolution:**
- Add the `webauthn` token to the input's `autocomplete` attribute
- Call `navigator.credentials.get()` immediately on page load
- Abort any pending requests before starting a new one using `AbortController`
- Provide an explicit "Sign in with Passkey" button as fallback

### 2.2 Signature Verification Failed

**Cause:** The server's cryptographic verification of the assertion signature fails.

**Diagnosis Steps:**
1. Verify the correct public key is being used (match credential ID to stored record).
2. Ensure the signed data is constructed correctly: `authenticatorData || SHA-256(clientDataJSON)`.
3. Check the algorithm matches what was used during registration (ES256 vs RS256).
4. Verify no data corruption occurred during base64url encoding/decoding.
5. Check for byte-order issues in the public key parsing.

**Resolution:**
- Log the raw authenticatorData and clientDataJSON for debugging
- Verify base64url decoding is correct (no padding, URL-safe alphabet)
- Ensure the public key was stored in the correct format (COSE vs PEM vs DER)
- Use a well-tested library rather than implementing verification manually

### 2.3 Sign Count Regression (Clone Detection)

**Cause:** The received `signCount` is less than or equal to the stored `signCount`, indicating a potential credential clone.

**Diagnosis Steps:**
1. Check if the credential is a synced passkey (backup_eligible = true). Synced passkeys often report signCount = 0 consistently.
2. If the credential is device-bound and the counter has regressed, this is a genuine security concern.

**Resolution:**
- For synced passkeys (signCount always 0): Disable clone detection for these credentials
- For device-bound passkeys with regression: Immediately revoke the credential, notify the user, and require re-registration via a high-assurance recovery flow

## 3. Cross-Device (Hybrid Transport) Issues

### 3.1 QR Code Not Appearing

**Cause:** The browser is not offering the cross-device option.

**Diagnosis Steps:**
1. Verify the device has Bluetooth hardware and it is enabled.
2. Check that the browser supports hybrid transport (Chrome 108+, Safari 16+, Edge 108+).
3. Ensure `allowCredentials` does not restrict to only `["internal"]` transports.

**Resolution:**
- Do not restrict transports in `allowCredentials` unless specifically required
- Inform users that Bluetooth must be enabled for cross-device authentication
- Provide alternative authentication methods for devices without Bluetooth

### 3.2 Cross-Device Connection Timeout

**Cause:** The BLE proximity check or cloud relay connection failed.

**Diagnosis Steps:**
1. Ensure the phone and computer are within Bluetooth range (typically 10 meters).
2. Check that both devices have internet connectivity.
3. Verify no firewall is blocking the WebSocket connection to the cloud relay.

**Resolution:**
- Increase the timeout to at least 120 seconds for cross-device flows
- Instruct users to keep devices close together
- Check corporate firewall rules for WebSocket blocking

## 4. Platform-Specific Issues

### 4.1 Safari: "This request has been cancelled by the user"

**Cause:** Safari is particularly strict about user gesture requirements and will cancel requests that are not directly triggered by user interaction.

**Resolution:** Ensure the WebAuthn call is the direct result of a click event. Do not use `setTimeout`, `Promise.then`, or `async/await` chains that break the user gesture context.

### 4.2 Android: Multiple Credential Provider Dialog

**Cause:** The user has multiple credential providers installed (Google Password Manager + 1Password + Bitwarden), and Android displays a selection dialog.

**Resolution:** This is expected behavior on Android and cannot be suppressed by the relying party. Ensure your documentation helps users understand which provider contains their passkey.

### 4.3 Windows: "Windows Hello is not set up"

**Cause:** The user's Windows device does not have Windows Hello configured (no PIN, fingerprint, or facial recognition set up).

**Resolution:** Guide the user to Settings > Accounts > Sign-in options to configure Windows Hello. Alternatively, offer cross-device authentication via their smartphone.

### 4.4 Firefox: Conditional UI Not Working

**Cause:** Firefox's Conditional UI support has historically lagged behind Chrome and Safari.

**Resolution:** Always implement both Conditional UI and an explicit "Sign in with Passkey" button. Check `isConditionalMediationAvailable()` and only activate conditional mediation if supported.

## 5. Server-Side Issues

### 5.1 Challenge Expired or Not Found

**Cause:** The challenge stored in the server's session/cache has expired (TTL exceeded) or was never stored.

**Resolution:**
- Increase challenge TTL to 120-300 seconds to accommodate slow users
- Verify Redis/session storage is healthy and accessible
- Ensure challenge is stored BEFORE returning options to the client
- Implement proper error handling that prompts the user to retry

### 5.2 Origin Mismatch

**Cause:** The `origin` in `clientDataJSON` does not match the server's expected origin.

**Common causes:**
- Server expects `https://example.com` but client sends `https://www.example.com`
- Reverse proxy stripping or modifying headers
- Development environment using `http://localhost` while production expects HTTPS

**Resolution:**
- Configure the server to accept all valid origins for your deployment
- For localhost development, accept `http://localhost:<port>` origins
- Ensure reverse proxy configuration preserves the original origin

### 5.3 Attestation Verification Failure

**Cause:** The server cannot verify the attestation certificate chain.

**Resolution:**
- If attestation is not required for your use case, set `attestation: "none"` and skip verification
- If attestation is required, ensure root certificates are up to date
- Download the latest FIDO MDS blob and update trust anchors
- Check certificate expiration dates in the attestation chain

## 6. Migration and Upgrade Issues

### 6.1 Cannot Change RP ID After Deployment

**Problem:** The RP ID was set incorrectly during initial deployment, and now all existing credentials are bound to the wrong domain.

**Impact:** All existing passkeys become permanently unusable if the RP ID is changed.

**Resolution:** There is no technical fix. The RP ID is cryptographically bound to the credential. Options:
1. Keep the incorrect RP ID and work around it (if possible)
2. Force all users to re-register new passkeys under the correct RP ID
3. Use WebAuthn Level 3 "related origins" if the incorrect RP ID is a related domain

### 6.2 Library Upgrade Breaking Changes

**Problem:** Upgrading the WebAuthn server library introduces breaking changes in the verification logic.

**Resolution:**
- Pin library versions in production
- Test upgrades thoroughly in staging with real authenticators
- Maintain backward compatibility for existing credential formats
- Never modify stored credential data during library upgrades
