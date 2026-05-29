# Slack Master Specialist — Security Audit Guide

## 1. Token Security

### Token Types and Risk Levels

| Token | Prefix | Risk | Exposure Impact |
|-------|--------|------|-----------------|
| Bot Token | `xoxb-` | High | Full bot access to workspace |
| User Token | `xoxp-` | Critical | Full user access, can impersonate |
| App Token | `xapp-` | Medium | Socket Mode connection only |
| Webhook URL | `https://hooks...` | Medium | Can post to specific channel |
| Signing Secret | 32-char hex | High | Can forge requests to your app |

### Token Storage Best Practices

```python
# NEVER do this
BOT_TOKEN = "xoxb-123-456-abc"  # Hardcoded in source

# CORRECT: Use environment variables
import os
BOT_TOKEN = os.environ["SLACK_BOT_TOKEN"]

# CORRECT: Use secrets manager
from aws_secretsmanager import get_secret
BOT_TOKEN = get_secret("slack/bot-token")
```

### Token Rotation

```python
# Check if token rotation is enabled
# When enabled, tokens expire and must be refreshed

def refresh_token(client_id, client_secret, refresh_token):
    response = requests.post("https://slack.com/api/oauth.v2.access", data={
        "client_id": client_id,
        "client_secret": client_secret,
        "grant_type": "refresh_token",
        "refresh_token": refresh_token
    })
    data = response.json()
    if data["ok"]:
        return {
            "access_token": data["access_token"],
            "refresh_token": data["refresh_token"],
            "expires_in": data["expires_in"]
        }
    raise Exception(f"Token refresh failed: {data['error']}")
```

## 2. Request Verification

### Signing Secret Verification (Required)

Every incoming request from Slack must be verified:

```python
import hashlib
import hmac
import time

def verify_slack_request(signing_secret, body, timestamp, signature):
    # Prevent replay attacks (reject requests older than 5 minutes)
    if abs(time.time() - int(timestamp)) > 300:
        return False
    
    # Compute expected signature
    sig_basestring = f"v0:{timestamp}:{body}"
    expected_signature = "v0=" + hmac.new(
        signing_secret.encode("utf-8"),
        sig_basestring.encode("utf-8"),
        hashlib.sha256
    ).hexdigest()
    
    # Constant-time comparison to prevent timing attacks
    return hmac.compare_digest(expected_signature, signature)
```

### Middleware Implementation

```python
from flask import Flask, request, abort
from functools import wraps

def require_slack_verification(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        timestamp = request.headers.get("X-Slack-Request-Timestamp", "")
        signature = request.headers.get("X-Slack-Signature", "")
        body = request.get_data(as_text=True)
        
        if not verify_slack_request(SIGNING_SECRET, body, timestamp, signature):
            abort(401)
        
        return f(*args, **kwargs)
    return decorated
```

## 3. OAuth Security

### Secure OAuth Flow

```python
import secrets

# Generate state parameter to prevent CSRF
def generate_oauth_state():
    state = secrets.token_urlsafe(32)
    # Store in session/database with expiry
    store_state(state, expires_in=600)
    return state

# Verify state on callback
def handle_oauth_callback(request):
    state = request.args.get("state")
    if not verify_state(state):
        abort(403, "Invalid state parameter - possible CSRF attack")
    
    code = request.args.get("code")
    # Exchange code for token...
```

### Scope Minimization

Only request the scopes your app actually needs:

```yaml
# BAD: Over-permissioned
oauth_config:
  scopes:
    bot:
      - admin
      - channels:manage
      - chat:write
      - users:read
      - files:read
      - files:write

# GOOD: Minimal permissions
oauth_config:
  scopes:
    bot:
      - chat:write          # Only if posting messages
      - channels:read       # Only if reading channel info
      - app_mentions:read   # Only if responding to mentions
```

## 4. Data Protection

### Sensitive Data in Messages

```python
# NEVER include sensitive data in messages
# BAD
client.chat_postMessage(channel=channel, text=f"API Key: {api_key}")

# GOOD: Use ephemeral messages for sensitive info
client.chat_postEphemeral(
    channel=channel,
    user=user_id,
    text="Your temporary access code has been sent to your email."
)
```

### Message Retention and Deletion

```python
# Delete sensitive messages after a timeout
import threading

def post_and_auto_delete(client, channel, text, delete_after_seconds=300):
    result = client.chat_postMessage(channel=channel, text=text)
    ts = result["ts"]
    
    def delete_later():
        time.sleep(delete_after_seconds)
        try:
            client.chat_delete(channel=channel, ts=ts)
        except Exception:
            pass
    
    threading.Thread(target=delete_later, daemon=True).start()
    return result
```

### Input Sanitization

```python
def sanitize_user_input(text, max_length=3000):
    # Remove potential mrkdwn injection
    sanitized = text.replace("<", "&lt;").replace(">", "&gt;").replace("&", "&amp;")
    
    # Prevent @channel/@here abuse
    sanitized = sanitized.replace("<!channel>", "[channel]")
    sanitized = sanitized.replace("<!here>", "[here]")
    sanitized = sanitized.replace("<!everyone>", "[everyone]")
    
    # Truncate to prevent oversized messages
    if len(sanitized) > max_length:
        sanitized = sanitized[:max_length - 3] + "..."
    
    return sanitized
```

## 5. App Installation Security

### Restricting App Installation

For Enterprise Grid, restrict which workspaces can install your app:

```python
def handle_oauth_callback(request):
    code = request.args.get("code")
    result = client.oauth_v2_access(
        client_id=CLIENT_ID,
        client_secret=CLIENT_SECRET,
        code=code
    )
    
    team_id = result["team"]["id"]
    enterprise_id = result.get("enterprise", {}).get("id")
    
    # Verify the installing workspace is allowed
    allowed_teams = get_allowed_teams()
    if team_id not in allowed_teams:
        return "Installation not permitted for this workspace", 403
    
    # Store token securely
    store_token_encrypted(team_id, result["access_token"])
```

## 6. Webhook Security

### Webhook URL Protection

- Never expose webhook URLs in client-side code
- Rotate webhook URLs periodically
- Use IP allowlisting if possible
- Monitor webhook usage for anomalies

### Outgoing Webhook Verification

```python
def verify_outgoing_webhook(token, expected_token):
    """Verify the token in outgoing webhook payloads"""
    return hmac.compare_digest(token, expected_token)
```

## 7. Network Security

### HTTPS Requirements

- All Slack API communication must use HTTPS
- Request URLs must have valid SSL certificates
- Self-signed certificates are not accepted
- TLS 1.2+ is required

### IP Allowlisting

Slack does not publish a fixed IP range for outgoing requests. Instead:
- Use request signature verification (signing secret)
- Implement proper authentication on all endpoints
- Use Socket Mode to avoid exposing public endpoints

## 8. Audit Logging

### App Activity Logging

```python
import logging
import json
from datetime import datetime

audit_logger = logging.getLogger("slack_audit")
audit_logger.setLevel(logging.INFO)
handler = logging.FileHandler("slack_audit.log")
audit_logger.addHandler(handler)

def log_action(action, user_id, channel=None, details=None):
    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "action": action,
        "user_id": user_id,
        "channel": channel,
        "details": details
    }
    audit_logger.info(json.dumps(entry))

# Usage
log_action("message_posted", user_id="U012AB3CD", channel="C012AB3CD", details={"text_length": 150})
log_action("modal_submitted", user_id="U012AB3CD", details={"callback_id": "create_ticket"})
log_action("file_uploaded", user_id="U012AB3CD", channel="C012AB3CD", details={"file_type": "pdf", "size_bytes": 1048576})
```

### Enterprise Grid Audit Logs

```bash
# Query audit logs
curl "https://api.slack.com/audit/v1/logs?action=user_login&oldest=1716000000" \
  -H "Authorization: Bearer xoxp-org-admin-token"

# Monitor app installations
curl "https://api.slack.com/audit/v1/logs?action=app_installed" \
  -H "Authorization: Bearer xoxp-org-admin-token"

# Track file downloads
curl "https://api.slack.com/audit/v1/logs?action=file_downloaded&actor=U012AB3CD" \
  -H "Authorization: Bearer xoxp-org-admin-token"
```

## 9. Security Checklist

### Development Phase

- [ ] Signing secret stored in environment variable, not code
- [ ] All tokens stored in secrets manager
- [ ] Request signature verification on all endpoints
- [ ] OAuth state parameter implemented (CSRF protection)
- [ ] Minimal scopes requested
- [ ] Input sanitization on all user-provided data
- [ ] No sensitive data logged or stored in messages
- [ ] Error messages don't leak internal details

### Deployment Phase

- [ ] HTTPS with valid certificate on all endpoints
- [ ] Token rotation enabled (if supported)
- [ ] Rate limiting implemented
- [ ] Audit logging enabled
- [ ] Monitoring for unusual API usage patterns
- [ ] Webhook URLs not exposed in public repositories
- [ ] Environment variables not in Docker images
- [ ] Secrets not in CI/CD logs

### Operations Phase

- [ ] Regular token rotation schedule
- [ ] Unused app permissions removed
- [ ] Inactive apps uninstalled
- [ ] Audit logs reviewed weekly
- [ ] Incident response plan for token compromise
- [ ] Backup authentication method available
- [ ] DLP policies configured (Enterprise)
- [ ] Information barriers set (Enterprise Grid)

## 10. Incident Response for Token Compromise

### Immediate Actions

1. Revoke the compromised token immediately
2. Check audit logs for unauthorized activity
3. Regenerate all related secrets (signing secret, client secret)
4. Re-install the app to generate new tokens
5. Notify affected workspace admins
6. Review and rotate any other secrets that may have been exposed

### Prevention

```python
# Implement token usage monitoring
def monitor_token_usage(method, response):
    # Alert on unusual patterns
    if method in ["admin.users.remove", "conversations.delete"]:
        alert_security_team(f"Sensitive API call: {method}")
    
    # Track usage patterns
    record_api_usage(method, response.get("ok"), time.time())
```

This security audit guide ensures your Slack applications follow enterprise-grade security practices and are resilient against common attack vectors.
