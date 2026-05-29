# Slack Master Specialist — CLI and API Reference

## 1. Web API Methods (Complete Reference)

### chat.* Methods

```bash
# Post a message
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","text":"Hello","blocks":[...]}'

# Update a message
curl -X POST https://slack.com/api/chat.update \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","ts":"1234567890.123456","text":"Updated"}'

# Delete a message
curl -X POST https://slack.com/api/chat.delete \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","ts":"1234567890.123456"}'

# Schedule a message
curl -X POST https://slack.com/api/chat.scheduleMessage \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","text":"Reminder!","post_at":1716100000}'

# Post ephemeral message (visible only to one user)
curl -X POST https://slack.com/api/chat.postEphemeral \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","user":"U012AB3CD","text":"Only you can see this"}'

# Get message permalink
curl "https://slack.com/api/chat.getPermalink?channel=C012AB3CD&message_ts=1234567890.123456" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Unfurl links in a message
curl -X POST https://slack.com/api/chat.unfurl \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","ts":"1234567890.123456","unfurls":{"https://example.com":{"blocks":[...]}}}'
```

### conversations.* Methods

```bash
# List channels
curl "https://slack.com/api/conversations.list?types=public_channel,private_channel&limit=200" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Get channel info
curl "https://slack.com/api/conversations.info?channel=C012AB3CD&include_num_members=true" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Create a channel
curl -X POST https://slack.com/api/conversations.create \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"proj-new-feature","is_private":false}'

# Join a channel
curl -X POST https://slack.com/api/conversations.join \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD"}'

# Invite users to a channel
curl -X POST https://slack.com/api/conversations.invite \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","users":"U001,U002,U003"}'

# Kick a user from a channel
curl -X POST https://slack.com/api/conversations.kick \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","user":"U012AB3CD"}'

# Get channel history
curl "https://slack.com/api/conversations.history?channel=C012AB3CD&limit=100&oldest=1716000000" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Get thread replies
curl "https://slack.com/api/conversations.replies?channel=C012AB3CD&ts=1234567890.123456" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Get channel members
curl "https://slack.com/api/conversations.members?channel=C012AB3CD&limit=200" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Set channel topic
curl -X POST https://slack.com/api/conversations.setTopic \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","topic":"Sprint 42 | Ends June 1 | Board: https://jira.example.com"}'

# Set channel purpose
curl -X POST https://slack.com/api/conversations.setPurpose \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","purpose":"Team discussions for the platform team"}'

# Archive a channel
curl -X POST https://slack.com/api/conversations.archive \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD"}'

# Unarchive a channel
curl -X POST https://slack.com/api/conversations.unarchive \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD"}'

# Rename a channel
curl -X POST https://slack.com/api/conversations.rename \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","name":"new-channel-name"}'

# Open a DM
curl -X POST https://slack.com/api/conversations.open \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"users":"U001,U002"}'
```

### users.* Methods

```bash
# Get user info
curl "https://slack.com/api/users.info?user=U012AB3CD" \
  -H "Authorization: Bearer xoxb-TOKEN"

# List all users
curl "https://slack.com/api/users.list?limit=200" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Lookup user by email
curl "https://slack.com/api/users.lookupByEmail?email=user@example.com" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Get user presence
curl "https://slack.com/api/users.getPresence?user=U012AB3CD" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Set user profile (requires user token)
curl -X POST https://slack.com/api/users.profile.set \
  -H "Authorization: Bearer xoxp-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"profile":{"status_text":"In a meeting","status_emoji":":calendar:","status_expiration":1716100000}}'
```

### reactions.* Methods

```bash
# Add a reaction
curl -X POST https://slack.com/api/reactions.add \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456","name":"thumbsup"}'

# Remove a reaction
curl -X POST https://slack.com/api/reactions.remove \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456","name":"thumbsup"}'

# Get reactions on a message
curl "https://slack.com/api/reactions.get?channel=C012AB3CD&timestamp=1234567890.123456" \
  -H "Authorization: Bearer xoxb-TOKEN"

# List items a user reacted to
curl "https://slack.com/api/reactions.list?user=U012AB3CD&limit=100" \
  -H "Authorization: Bearer xoxb-TOKEN"
```

### views.* Methods

```bash
# Open a modal
curl -X POST https://slack.com/api/views.open \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"trigger_id":"12345.98765","view":{"type":"modal","callback_id":"my_modal","title":{"type":"plain_text","text":"My Modal"},"blocks":[...]}}'

# Update a modal
curl -X POST https://slack.com/api/views.update \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"view_id":"V012AB3CD","view":{"type":"modal","callback_id":"my_modal","title":{"type":"plain_text","text":"Updated"},"blocks":[...]}}'

# Push a new view onto the modal stack
curl -X POST https://slack.com/api/views.push \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"trigger_id":"12345.98765","view":{"type":"modal","callback_id":"step2","title":{"type":"plain_text","text":"Step 2"},"blocks":[...]}}'

# Publish App Home tab
curl -X POST https://slack.com/api/views.publish \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"user_id":"U012AB3CD","view":{"type":"home","blocks":[...]}}'
```

### files.* Methods

```bash
# Upload a file (v2 - modern)
# Step 1: Get upload URL
curl -X POST https://slack.com/api/files.getUploadURLExternal \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"filename":"report.pdf","length":1048576}'

# Step 2: Upload to returned URL
curl -X POST "$UPLOAD_URL" -F "file=@report.pdf"

# Step 3: Complete upload
curl -X POST https://slack.com/api/files.completeUploadExternal \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"files":[{"id":"F012AB3CD","title":"Report"}],"channel_id":"C012AB3CD"}'

# List files
curl "https://slack.com/api/files.list?channel=C012AB3CD&types=pdfs,images&count=50" \
  -H "Authorization: Bearer xoxb-TOKEN"

# Delete a file
curl -X POST https://slack.com/api/files.delete \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file":"F012AB3CD"}'

# Get file info
curl "https://slack.com/api/files.info?file=F012AB3CD" \
  -H "Authorization: Bearer xoxb-TOKEN"
```

### pins.* Methods

```bash
# Pin a message
curl -X POST https://slack.com/api/pins.add \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456"}'

# Unpin a message
curl -X POST https://slack.com/api/pins.remove \
  -H "Authorization: Bearer xoxb-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456"}'

# List pinned items
curl "https://slack.com/api/pins.list?channel=C012AB3CD" \
  -H "Authorization: Bearer xoxb-TOKEN"
```

## 2. Slack CLI (slack-cli) Commands

```bash
# Authentication
slack login                    # Login to Slack
slack logout                   # Logout
slack auth list                # List authenticated workspaces

# App Management
slack create my-app            # Create new app from template
slack create my-app --template https://github.com/slack-samples/deno-starter-template
slack run                      # Run app locally with hot reload
slack deploy                   # Deploy to Slack infrastructure
slack delete                   # Delete deployed app

# Triggers
slack trigger create --trigger-def triggers/shortcut.ts
slack trigger list             # List all triggers
slack trigger info --trigger-id Ft012AB3CD
slack trigger delete --trigger-id Ft012AB3CD
slack trigger update --trigger-id Ft012AB3CD --trigger-def triggers/updated.ts

# Activity & Debugging
slack activity                 # View app activity
slack activity --tail          # Stream activity in real-time
slack activity --source functions  # Filter by source

# Datastore (for next-gen apps)
slack datastore put '{"datastore":"tasks","item":{"id":"1","title":"Test"}}'
slack datastore get '{"datastore":"tasks","id":"1"}'
slack datastore query '{"datastore":"tasks","expression":"#status = :s","expression_attributes":{"#status":"status"},"expression_values":{":s":"open"}}'
slack datastore delete '{"datastore":"tasks","id":"1"}'

# Environment Variables
slack env add MY_SECRET --value "secret123"
slack env list
slack env remove MY_SECRET

# Manifest
slack manifest info            # View current manifest
slack manifest validate        # Validate manifest
```

## 3. Python SDK Quick Reference

```python
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError

client = WebClient(token="xoxb-TOKEN")

# Post message
client.chat_postMessage(channel="C012AB3CD", text="Hello", blocks=[...])

# Post to thread
client.chat_postMessage(channel="C012AB3CD", thread_ts="1234567890.123456", text="Reply")

# Update message
client.chat_update(channel="C012AB3CD", ts="1234567890.123456", text="Updated")

# Delete message
client.chat_delete(channel="C012AB3CD", ts="1234567890.123456")

# Open modal
client.views_open(trigger_id="12345.98765", view={...})

# Upload file
client.files_upload_v2(channel="C012AB3CD", file="./report.pdf", title="Report")

# Add reaction
client.reactions_add(channel="C012AB3CD", timestamp="1234567890.123456", name="thumbsup")

# Get user info
client.users_info(user="U012AB3CD")

# List channels with pagination
channels = []
cursor = None
while True:
    result = client.conversations_list(types="public_channel", limit=200, cursor=cursor)
    channels.extend(result["channels"])
    cursor = result.get("response_metadata", {}).get("next_cursor")
    if not cursor:
        break
```

## 4. Node.js SDK Quick Reference

```javascript
const { WebClient } = require('@slack/web-api');
const client = new WebClient(process.env.SLACK_BOT_TOKEN);

// Post message
await client.chat.postMessage({ channel: 'C012AB3CD', text: 'Hello', blocks: [...] });

// Post to thread
await client.chat.postMessage({ channel: 'C012AB3CD', thread_ts: '1234567890.123456', text: 'Reply' });

// Update message
await client.chat.update({ channel: 'C012AB3CD', ts: '1234567890.123456', text: 'Updated' });

// Open modal
await client.views.open({ trigger_id: '12345.98765', view: {...} });

// Upload file
await client.filesUploadV2({ channel_id: 'C012AB3CD', file: './report.pdf', filename: 'report.pdf' });

// Paginate through channels
let cursor;
const channels = [];
do {
  const result = await client.conversations.list({ types: 'public_channel', limit: 200, cursor });
  channels.push(...result.channels);
  cursor = result.response_metadata?.next_cursor;
} while (cursor);
```

## 5. Webhook One-Liners

```bash
# Simple text message
curl -X POST "$WEBHOOK_URL" -H "Content-Type: application/json" -d '{"text":"Hello from CLI"}'

# Message with emoji
curl -X POST "$WEBHOOK_URL" -H "Content-Type: application/json" -d '{"text":":rocket: Deployment started"}'

# Message with mention
curl -X POST "$WEBHOOK_URL" -H "Content-Type: application/json" -d '{"text":"<!here> Server alert!"}'

# Message with blocks
curl -X POST "$WEBHOOK_URL" -H "Content-Type: application/json" -d '{
  "blocks":[{"type":"section","text":{"type":"mrkdwn","text":"*Build #42* passed :white_check_mark:"}}]
}'

# From a shell script (CI/CD)
SLACK_MSG=":white_check_mark: Build \`${BUILD_NUMBER}\` passed for \`${REPO_NAME}\` on branch \`${BRANCH}\`"
curl -s -X POST "$SLACK_WEBHOOK" -H "Content-Type: application/json" -d "{\"text\":\"$SLACK_MSG\"}"
```

## 6. Common jq Patterns for Slack API Responses

```bash
# Extract channel IDs and names
curl -s "https://slack.com/api/conversations.list?types=public_channel" \
  -H "Authorization: Bearer xoxb-TOKEN" | jq -r '.channels[] | "\(.id)\t\(.name)"'

# Find channels with no messages in 30 days
curl -s "https://slack.com/api/conversations.list?types=public_channel" \
  -H "Authorization: Bearer xoxb-TOKEN" | jq -r '.channels[] | select(.updated < (now - 2592000)) | .name'

# Get user emails
curl -s "https://slack.com/api/users.list" \
  -H "Authorization: Bearer xoxb-TOKEN" | jq -r '.members[] | select(.deleted == false) | "\(.real_name)\t\(.profile.email)"'

# Count members per channel
curl -s "https://slack.com/api/conversations.list?types=public_channel" \
  -H "Authorization: Bearer xoxb-TOKEN" | jq -r '.channels[] | "\(.name)\t\(.num_members)"' | sort -t$'\t' -k2 -rn

# Extract message text from history
curl -s "https://slack.com/api/conversations.history?channel=C012AB3CD&limit=50" \
  -H "Authorization: Bearer xoxb-TOKEN" | jq -r '.messages[] | "\(.user // "bot"): \(.text)"'
```

## 7. Environment Variables Reference

| Variable | Description | Example |
|----------|-------------|---------|
| `SLACK_BOT_TOKEN` | Bot user OAuth token | `xoxb-123-456-abc` |
| `SLACK_APP_TOKEN` | App-level token (Socket Mode) | `xapp-1-A01-123-abc` |
| `SLACK_SIGNING_SECRET` | Request verification secret | `abc123def456` |
| `SLACK_CLIENT_ID` | OAuth client ID | `123456.789012` |
| `SLACK_CLIENT_SECRET` | OAuth client secret | `abcdef123456` |
| `SLACK_WEBHOOK_URL` | Incoming webhook URL | `https://hooks.slack.com/services/YOUR_T/YOUR_B/TOKEN` |
| `SLACK_LOG_LEVEL` | SDK log level | `DEBUG`, `INFO`, `WARN`, `ERROR` |

This reference covers all major Slack API methods, CLI commands, SDK patterns, and one-liners needed for production Slack app development and operations.
