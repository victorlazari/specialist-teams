# Slack Master Specialist — Troubleshooting Guide

## 1. Message Delivery Issues

### Messages Not Appearing

| Symptom | Cause | Fix |
|---------|-------|-----|
| Message not posted | Bot not in channel | `conversations.join` or invite bot |
| Message not visible | Posted as ephemeral | Use `chat.postMessage` instead of `chat.postEphemeral` |
| Message delayed | Rate limited | Check for 429 response, implement backoff |
| Message in wrong channel | Channel ID mismatch | Verify channel ID with `conversations.info` |
| Blocks not rendering | Invalid block JSON | Validate in Block Kit Builder |
| Formatting broken | Wrong text type | Use `mrkdwn` type for formatting, `plain_text` for no parsing |

### Debugging Message Failures

```python
try:
    result = client.chat_postMessage(channel=channel, text=text, blocks=blocks)
    if not result["ok"]:
        print(f"API returned ok=false: {result.get('error')}")
except SlackApiError as e:
    print(f"Error: {e.response['error']}")
    print(f"Response: {e.response.data}")
    
    # Common fixes
    if e.response["error"] == "not_in_channel":
        client.conversations_join(channel=channel)
        # Retry
    elif e.response["error"] == "channel_not_found":
        # Channel may be private or deleted
        pass
    elif e.response["error"] == "invalid_blocks":
        # Validate blocks at https://app.slack.com/block-kit-builder
        print(f"Invalid blocks: {json.dumps(blocks, indent=2)}")
```

## 2. Rate Limiting Issues

### Identifying Rate Limits

```python
import time

def handle_rate_limit(response):
    if response.status_code == 429:
        retry_after = int(response.headers.get("Retry-After", 30))
        print(f"Rate limited. Retry after {retry_after} seconds")
        time.sleep(retry_after)
        return True
    return False
```

### Rate Limit Tiers

| Tier | Limit | Methods | Strategy |
|------|-------|---------|----------|
| Tier 1 | 1/min | `admin.*` | Queue with 60s delay |
| Tier 2 | 20/min | `conversations.list`, `users.list` | Cache results, paginate efficiently |
| Tier 3 | 50/min | `chat.postMessage`, `reactions.add` | Batch operations, use queues |
| Tier 4 | 100/min | `auth.test`, `conversations.info` | Generally safe |
| Special | 1/sec/channel | `chat.postMessage` per channel | Distribute across channels |

### Burst Protection Pattern

```python
from collections import deque
import time

class BurstProtector:
    def __init__(self, max_per_minute=50):
        self.max_per_minute = max_per_minute
        self.timestamps = deque()
    
    def wait_if_needed(self):
        now = time.time()
        # Remove timestamps older than 60 seconds
        while self.timestamps and self.timestamps[0] < now - 60:
            self.timestamps.popleft()
        
        if len(self.timestamps) >= self.max_per_minute:
            wait_time = 60 - (now - self.timestamps[0])
            if wait_time > 0:
                time.sleep(wait_time)
        
        self.timestamps.append(time.time())
```

## 3. Authentication Issues

### Token Problems

| Error | Cause | Fix |
|-------|-------|-----|
| `not_authed` | Missing token | Add `Authorization: Bearer xoxb-...` header |
| `invalid_auth` | Malformed token | Check token format (xoxb- prefix) |
| `token_revoked` | App uninstalled or token rotated | Re-authorize the app |
| `missing_scope` | Token lacks permission | Add scope in app settings, re-install |
| `account_inactive` | User deactivated | Use a different user's token |
| `org_login_required` | SSO required | User must re-authenticate via SSO |
| `ekm_access_denied` | Enterprise key management | Contact workspace admin |

### Verifying Your Token

```bash
# Quick token check
curl "https://slack.com/api/auth.test" -H "Authorization: Bearer xoxb-YOUR-TOKEN"

# Expected response
# {"ok":true,"url":"https://team.slack.com/","team":"Team Name","user":"bot_name","team_id":"T012AB3CD","user_id":"U012AB3CD","bot_id":"B012AB3CD"}
```

### Scope Debugging

```bash
# Check what scopes your token has
curl "https://slack.com/api/auth.test" -H "Authorization: Bearer xoxb-TOKEN" -v 2>&1 | grep "x-oauth-scopes"
```

## 4. Block Kit Issues

### Common Block Kit Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `invalid_blocks` | Malformed JSON | Validate at Block Kit Builder |
| `too_many_blocks` | More than 50 blocks | Split into multiple messages |
| `invalid_blocks_format` | Wrong block structure | Check required fields |
| Text too long | Section text > 3000 chars | Truncate or split |
| Action ID collision | Duplicate action_id | Use unique action_ids |

### Block Validation Checklist

```python
def validate_blocks(blocks):
    errors = []
    
    if len(blocks) > 50:
        errors.append(f"Too many blocks: {len(blocks)} (max 50)")
    
    for i, block in enumerate(blocks):
        if "type" not in block:
            errors.append(f"Block {i}: missing 'type' field")
        
        if block.get("type") == "section":
            text = block.get("text", {})
            if text.get("type") == "mrkdwn" and len(text.get("text", "")) > 3000:
                errors.append(f"Block {i}: mrkdwn text exceeds 3000 chars")
            if text.get("type") == "plain_text" and len(text.get("text", "")) > 3000:
                errors.append(f"Block {i}: plain_text exceeds 3000 chars")
            
            fields = block.get("fields", [])
            if len(fields) > 10:
                errors.append(f"Block {i}: too many fields ({len(fields)}, max 10)")
        
        if block.get("type") == "actions":
            elements = block.get("elements", [])
            if len(elements) > 25:
                errors.append(f"Block {i}: too many action elements ({len(elements)}, max 25)")
        
        if block.get("type") == "context":
            elements = block.get("elements", [])
            if len(elements) > 10:
                errors.append(f"Block {i}: too many context elements ({len(elements)}, max 10)")
        
        if block.get("type") == "header":
            text = block.get("text", {}).get("text", "")
            if len(text) > 150:
                errors.append(f"Block {i}: header text exceeds 150 chars")
    
    return errors
```

## 5. Events API Issues

### Events Not Being Received

| Symptom | Cause | Fix |
|---------|-------|-----|
| No events at all | URL not verified | Complete URL verification challenge |
| Events stop arriving | Server returning non-200 | Fix server to respond 200 within 3s |
| Duplicate events | Not deduplicating | Track `event_id` to skip duplicates |
| Missing specific events | Not subscribed | Add event type in app settings |
| Events from wrong workspace | Multi-workspace app | Check `team_id` in payload |

### Event Debugging

```python
import logging

logging.basicConfig(level=logging.DEBUG)

@app.event("message")
def handle_all_messages(event, logger):
    logger.debug(f"Received event: {json.dumps(event, indent=2)}")
    
    # Check for subtypes that should be ignored
    if event.get("subtype") in ["bot_message", "message_changed", "message_deleted"]:
        logger.debug(f"Ignoring subtype: {event['subtype']}")
        return
    
    # Check for bot messages to avoid loops
    if event.get("bot_id"):
        logger.debug("Ignoring bot message")
        return
```

### Retry Headers

When Slack retries an event delivery, it includes:

```
X-Slack-Retry-Num: 1        # Retry attempt number (1, 2, or 3)
X-Slack-Retry-Reason: http_timeout  # Why it's retrying
```

Handle retries:
```python
@app.before_request
def handle_retries():
    retry_num = request.headers.get("X-Slack-Retry-Num")
    if retry_num:
        # Already processed this event, just acknowledge
        return make_response("", 200)
```

## 6. Socket Mode Issues

### Connection Problems

| Symptom | Cause | Fix |
|---------|-------|-----|
| Connection refused | Wrong app token | Use `xapp-` token, not `xoxb-` |
| Immediate disconnect | Socket Mode not enabled | Enable in app settings |
| Intermittent drops | Network instability | Implement reconnection logic |
| Events not received | Wrong event subscriptions | Check app event subscriptions |

### Socket Mode Reconnection

```python
from slack_sdk.socket_mode import SocketModeClient
from slack_sdk.socket_mode.listeners import SocketModeRequestListener
import time

def create_resilient_client():
    client = SocketModeClient(
        app_token="xapp-TOKEN",
        web_client=WebClient(token="xoxb-TOKEN"),
        auto_reconnect_enabled=True,
        trace_enabled=True
    )
    
    # Monitor connection health
    def on_disconnect():
        print("Disconnected! Auto-reconnect will handle this.")
    
    def on_connect():
        print("Connected to Slack!")
    
    return client
```

## 7. Modal and Interactivity Issues

### Modal Not Opening

| Symptom | Cause | Fix |
|---------|-------|-----|
| `expired_trigger_id` | trigger_id expired (3s) | Open modal immediately in ack handler |
| `invalid_trigger_id` | Wrong trigger_id format | Use trigger_id from the interaction payload |
| Modal opens blank | Empty blocks array | Add at least one block |
| Submit not working | Missing callback_id | Add `callback_id` to view |

### View Submission Errors

```python
@app.view("my_form")
def handle_submission(ack, body, view):
    values = view["state"]["values"]
    errors = {}
    
    # Validate inputs
    email = values["email_block"]["email_input"]["value"]
    if not email or "@" not in email:
        errors["email_block"] = "Please enter a valid email"
    
    name = values["name_block"]["name_input"]["value"]
    if not name or len(name) < 2:
        errors["name_block"] = "Name must be at least 2 characters"
    
    if errors:
        # Return validation errors (modal stays open)
        ack(response_action="errors", errors=errors)
    else:
        # Success - close modal
        ack()
        # Process the submission...
```

## 8. File Upload Issues

### Common Upload Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `file_not_found` | Invalid file path | Verify file exists locally |
| `too_large` | File exceeds limit | Compress or split file (max varies by plan) |
| `invalid_channel` | Bot not in channel | Join channel first |
| `not_allowed_token_type` | Using wrong token type | Use bot token with `files:write` scope |

### File Size Limits

| Plan | Max File Size |
|------|--------------|
| Free | 5 GB total workspace storage |
| Pro | 10 GB per member |
| Business+ | 20 GB per member |
| Enterprise Grid | 1 TB per member |

Individual file upload limit: 1 GB regardless of plan.

## 9. Webhook Issues

### Incoming Webhook Failures

| Symptom | Cause | Fix |
|---------|-------|-----|
| 404 response | Webhook URL invalid | Regenerate webhook in app settings |
| 410 response | Webhook revoked | App was uninstalled, re-install |
| 500 response | Slack server error | Retry with exponential backoff |
| No message appears | Invalid JSON payload | Validate JSON structure |
| Formatting not working | Wrong content type | Set `Content-Type: application/json` |

### Webhook Debugging

```bash
# Test webhook with verbose output
curl -v -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"text":"test"}'

# Check if webhook URL is reachable
curl -s -o /dev/null -w "%{http_code}" -X POST "$WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"text":"ping"}'
```

## 10. Slash Command Issues

### Command Not Responding

| Symptom | Cause | Fix |
|---------|-------|-----|
| "This command didn't work" | No response within 3s | Acknowledge immediately with `ack()` |
| "dispatch_failed" | Request URL unreachable | Check server is running and URL is correct |
| Command not found | Not installed in workspace | Re-install app or check command config |
| Wrong response format | Invalid response_type | Use "ephemeral" or "in_channel" |

### Proper Command Handling

```python
# WRONG - processing before ack causes timeout
@app.command("/deploy")
def bad_handler(ack, command):
    result = run_long_deployment()  # Takes 30 seconds
    ack(text=f"Done: {result}")  # TOO LATE - already timed out

# CORRECT - ack immediately, process async
@app.command("/deploy")
def good_handler(ack, command, respond):
    ack()  # Acknowledge within 3 seconds
    
    # Process asynchronously
    result = run_long_deployment()
    
    # Use respond() to send follow-up (uses response_url)
    respond(text=f"Done: {result}", response_type="in_channel")
```

## 11. Formatting Issues

### mrkdwn Not Rendering

| Issue | Cause | Fix |
|-------|-------|-----|
| `*bold*` shows literally | Using `plain_text` type | Change to `"type": "mrkdwn"` |
| Links not clickable | Wrong link format | Use `<url|text>` format |
| Newlines ignored | Using `\n` in plain_text | Use actual newlines or switch to mrkdwn |
| Code block broken | Unescaped backticks | Escape with backslash or use rich_text |
| Mentions not working | Wrong format | Use `<@U012AB3CD>` not `@username` |
| Channel links broken | Using # prefix | Use `<#C012AB3CD>` format |

### Special Characters That Need Escaping

In mrkdwn text, these characters have special meaning:
- `&` → `&amp;`
- `<` → `&lt;`
- `>` → `&gt;`

```python
def escape_mrkdwn(text):
    return text.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;")

# Use when including user-generated content in mrkdwn
user_input = "Check if x < 5 && y > 3"
safe_text = f"User said: {escape_mrkdwn(user_input)}"
```

## 12. Performance Issues

### Slow API Responses

```python
import time

def measure_api_call(method, **kwargs):
    start = time.time()
    try:
        result = getattr(client, method)(**kwargs)
        duration = (time.time() - start) * 1000
        print(f"{method} took {duration:.0f}ms")
        return result
    except SlackApiError as e:
        duration = (time.time() - start) * 1000
        print(f"{method} FAILED after {duration:.0f}ms: {e.response['error']}")
        raise
```

### Memory Issues with Large Workspaces

```python
# BAD: Loading all users into memory
all_users = client.users_list()["members"]  # Could be 10,000+ users

# GOOD: Stream with pagination
def process_users_streaming(client, callback):
    cursor = None
    while True:
        result = client.users_list(limit=200, cursor=cursor)
        for user in result["members"]:
            callback(user)  # Process one at a time
        cursor = result.get("response_metadata", {}).get("next_cursor")
        if not cursor:
            break
```

## 13. Common Integration Pitfalls

### Bot Message Loops

```python
# DANGER: This creates an infinite loop
@app.event("message")
def echo_message(event, say):
    say(event["text"])  # Bot responds to its own messages!

# SAFE: Filter out bot messages
@app.event("message")
def echo_message(event, say):
    if event.get("bot_id") or event.get("subtype"):
        return  # Ignore bot messages and subtypes
    say(f"You said: {event['text']}")
```

### Thread vs Channel Confusion

```python
# Post to thread (reply)
client.chat_postMessage(
    channel="C012AB3CD",
    thread_ts="1234567890.123456",  # Parent message timestamp
    text="This is a thread reply"
)

# Post to thread AND broadcast to channel
client.chat_postMessage(
    channel="C012AB3CD",
    thread_ts="1234567890.123456",
    reply_broadcast=True,  # Also shows in channel
    text="Important thread reply"
)
```

## 14. Diagnostic Commands

```bash
# Check API status
curl -s "https://status.slack.com/api/v2.0.0/current" | jq .

# Verify bot permissions
curl -s "https://slack.com/api/auth.test" -H "Authorization: Bearer $SLACK_BOT_TOKEN" | jq .

# Check if bot is in a channel
curl -s "https://slack.com/api/conversations.info?channel=C012AB3CD" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" | jq '.channel.is_member'

# List bot's channels
curl -s "https://slack.com/api/users.conversations?types=public_channel,private_channel" \
  -H "Authorization: Bearer $SLACK_BOT_TOKEN" | jq '.channels[].name'

# Check rate limit headers in response
curl -v "https://slack.com/api/auth.test" -H "Authorization: Bearer $SLACK_BOT_TOKEN" 2>&1 | grep -i "retry\|x-ratelimit"
```

This troubleshooting guide covers all common issues encountered when developing and operating Slack applications in production environments.
