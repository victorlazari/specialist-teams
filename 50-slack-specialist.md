# Slack Master Specialist

## 1. Role Definition and Expertise

The Slack Master Specialist possesses comprehensive knowledge of the Slack platform, covering message formatting with mrkdwn syntax, Block Kit UI framework, emoji and reactions systems, Web API methods, app development patterns, workflow automation, and enterprise administration. This specialist helps teams communicate effectively by crafting well-formatted messages, building interactive Block Kit layouts, developing Slack apps, troubleshooting integration issues, and optimizing workspace configurations for maximum productivity.

## 2. Slack Architecture Overview

Slack operates as a channel-based messaging platform built on a real-time event-driven architecture. The platform consists of workspaces (formerly teams), which contain channels (public and private), direct messages, and group messages. Each workspace has its own set of users, custom emoji, installed apps, and configuration settings.

### Core Concepts

| Concept | Description | ID Format |
|---------|-------------|-----------|
| Workspace | Top-level organizational unit | T followed by alphanumeric (T012AB3CD) |
| Channel | Conversation space (public or private) | C followed by alphanumeric (C012AB3CD) |
| User | Individual workspace member | U followed by alphanumeric (U012AB3CD) |
| Bot User | App-controlled user | B followed by alphanumeric (B012AB3CD) |
| Message | Individual post in a channel | Timestamp-based (1234567890.123456) |
| Thread | Reply chain under a parent message | Uses parent message ts |
| File | Uploaded or shared file | F followed by alphanumeric |
| App | Integration or bot application | A followed by alphanumeric |

### Message Addressing

Every message in Slack is uniquely identified by the combination of its channel ID and timestamp (ts). The timestamp serves as both a unique identifier and a chronological marker. Thread replies reference the parent message's ts as thread_ts.

### Enterprise Grid

Enterprise Grid connects multiple workspaces under a single organization. It provides centralized administration, shared channels across workspaces, organization-wide search, and unified compliance controls. Grid organizations have an org-level ID (E prefix) and manage multiple workspace-level teams.

## 3. Message Formatting with mrkdwn

Slack uses its own markup language called mrkdwn (not standard Markdown). Understanding the differences is critical for proper message formatting.

### Text Formatting

```
*bold text*          → bold text
_italic text_        → italic text
~strikethrough~      → strikethrough text
`inline code`        → inline code
```code block```     → code block (multi-line)
> blockquote         → indented quote (single line)
>>> blockquote all   → quotes everything after this marker
```

### Important mrkdwn vs Markdown Differences

| Feature | Markdown | Slack mrkdwn |
|---------|----------|--------------|
| Bold | `**text**` | `*text*` |
| Italic | `*text*` | `_text_` |
| Strikethrough | `~~text~~` | `~text~` |
| Headers | `# Header` | Not supported |
| Bullet lists | `- item` | Use bullet character or emoji |
| Numbered lists | `1. item` | Manual numbering |
| Tables | Pipe syntax | Not supported in mrkdwn |
| Images | `![alt](url)` | Not supported (use blocks) |
| Horizontal rule | `---` | Not supported |

### Links and References

```
<https://example.com|Display Text>              → Hyperlink with custom text
<mailto:user@example.com|Email Link>            → Email link
<#C012AB3CD>                                     → Channel reference (auto-resolves name)
<#C012AB3CD|general>                            → Channel with fallback text
<@U012AB3CD>                                     → User mention (triggers notification)
<!subteam^S012AB3CD|@team-name>                 → User group mention
<!here>                                          → Notify active channel members
<!channel>                                       → Notify all channel members
<!everyone>                                      → Notify everyone in #general
```

### Date Formatting

Slack provides locale-aware date formatting using a special syntax that renders dates in each user's local timezone:

```
<!date^1392734382^{date_num} at {time}|February 18th, 2014 at 6:39 AM PST>
```

Available date tokens:

| Token | Example Output |
|-------|---------------|
| `{date_num}` | 2014-02-18 |
| `{date}` | February 18th, 2014 |
| `{date_short}` | Feb 18, 2014 |
| `{date_long}` | Tuesday, February 18th, 2014 |
| `{date_pretty}` | yesterday, today, tomorrow (or falls back to {date}) |
| `{date_short_pretty}` | yesterday, today, tomorrow (or falls back to {date_short}) |
| `{date_long_pretty}` | yesterday, today, tomorrow (or falls back to {date_long}) |
| `{time}` | 6:39 AM |
| `{time_secs}` | 6:39:42 AM |

The syntax is: `<!date^UNIX_TIMESTAMP^TOKEN_STRING^OPTIONAL_LINK|FALLBACK_TEXT>`

### Special Characters and Escaping

Three characters must be escaped in mrkdwn:

```
& → &amp;
< → &lt;
> → &gt;
```

## 4. Block Kit Complete Reference

Block Kit is Slack's UI framework for building rich, interactive message layouts. It uses a JSON-based structure where blocks are stacked vertically to create complex layouts.

### Limits

- Messages: maximum 50 blocks
- Modals and Home tabs: maximum 100 blocks
- Text fields within blocks: maximum 3,000 characters (some fields allow less)

### 4.1 Section Block

The most versatile block type. Displays text with an optional accessory element and up to 10 fields.

```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*Project Update*\nThe deployment completed successfully."
  },
  "accessory": {
    "type": "button",
    "text": { "type": "plain_text", "text": "View Details" },
    "action_id": "view_details_btn",
    "url": "https://deploy.example.com/run/123"
  },
  "fields": [
    { "type": "mrkdwn", "text": "*Environment:*\nProduction" },
    { "type": "mrkdwn", "text": "*Duration:*\n3m 42s" }
  ]
}
```

### 4.2 Header Block

Displays large, bold text. Plain text only, max 150 characters.

```json
{
  "type": "header",
  "text": { "type": "plain_text", "text": "Weekly Status Report", "emoji": true }
}
```

### 4.3 Divider Block

A simple horizontal line separator.

```json
{ "type": "divider" }
```

### 4.4 Image Block

Displays a standalone image with optional title.

```json
{
  "type": "image",
  "image_url": "https://example.com/chart.png",
  "alt_text": "Monthly revenue chart",
  "title": { "type": "plain_text", "text": "Revenue Q1 2025" }
}
```

### 4.5 Actions Block

Holds up to 25 interactive elements.

```json
{
  "type": "actions",
  "elements": [
    {
      "type": "button",
      "text": { "type": "plain_text", "text": "Approve" },
      "style": "primary",
      "action_id": "approve_request",
      "value": "req_001"
    },
    {
      "type": "button",
      "text": { "type": "plain_text", "text": "Deny" },
      "style": "danger",
      "action_id": "deny_request",
      "value": "req_001"
    },
    {
      "type": "static_select",
      "placeholder": { "type": "plain_text", "text": "Assign to..." },
      "action_id": "assign_user",
      "options": [
        { "text": { "type": "plain_text", "text": "Alice" }, "value": "U001" },
        { "text": { "type": "plain_text", "text": "Bob" }, "value": "U002" }
      ]
    }
  ]
}
```

### 4.6 Context Block

Displays small contextual information (max 10 elements of images and text).

```json
{
  "type": "context",
  "elements": [
    { "type": "image", "image_url": "https://example.com/avatar.png", "alt_text": "avatar" },
    { "type": "mrkdwn", "text": "Submitted by <@U012AB3CD> on <!date^1716000000^{date_short}|May 18>" }
  ]
}
```

### 4.7 Input Block

Collects user input. Works in modals, Home tabs, and messages with dispatch_action.

```json
{
  "type": "input",
  "block_id": "title_input",
  "label": { "type": "plain_text", "text": "Issue Title" },
  "element": {
    "type": "plain_text_input",
    "action_id": "title_value",
    "placeholder": { "type": "plain_text", "text": "Enter a descriptive title" },
    "max_length": 150
  },
  "hint": { "type": "plain_text", "text": "Keep it concise but descriptive" },
  "optional": false
}
```

### 4.8 Rich Text Block

Provides structured rich text with formatting preserved.

```json
{
  "type": "rich_text",
  "elements": [
    {
      "type": "rich_text_section",
      "elements": [
        { "type": "text", "text": "Important: ", "style": { "bold": true } },
        { "type": "text", "text": "Review changes before merging." }
      ]
    },
    {
      "type": "rich_text_list",
      "style": "bullet",
      "elements": [
        { "type": "rich_text_section", "elements": [{ "type": "text", "text": "Updated schema" }] },
        { "type": "rich_text_section", "elements": [{ "type": "text", "text": "Added endpoints" }] }
      ]
    }
  ]
}
```

### 4.9 Video Block

```json
{
  "type": "video",
  "title": { "type": "plain_text", "text": "Product Demo" },
  "video_url": "https://www.youtube.com/embed/dQw4w9WgXcQ",
  "thumbnail_url": "https://example.com/thumb.png",
  "alt_text": "Product demonstration video"
}
```

### 4.10 Alert Block (Modals only)

```json
{
  "type": "alert",
  "text": { "type": "plain_text", "text": "This action cannot be undone." },
  "variant": "warning"
}
```

Variants: `info`, `warning`, `danger`, `success`.

### 4.11 Card Block

```json
{
  "type": "card",
  "title": { "type": "plain_text", "text": "Sprint Task" },
  "description": { "type": "mrkdwn", "text": "Implement user auth flow" },
  "thumbnail": { "image_url": "https://example.com/icon.png", "alt_text": "icon" }
}
```

### 4.12 Carousel Block

Horizontally-scrolling container of card blocks.

```json
{
  "type": "carousel",
  "elements": [
    { "type": "card", "title": { "type": "plain_text", "text": "Card 1" } },
    { "type": "card", "title": { "type": "plain_text", "text": "Card 2" } }
  ]
}
```

### 4.13 Data Table Block

```json
{
  "type": "data_table",
  "columns": [
    { "id": "name", "name": "Service", "type": "text" },
    { "id": "status", "name": "Status", "type": "text" }
  ],
  "rows": [
    { "cells": { "name": "API Gateway", "status": "Healthy" } },
    { "cells": { "name": "Payment API", "status": "Degraded" } }
  ]
}
```

### 4.14 Table Block

```json
{
  "type": "table",
  "rows": [
    { "cells": [{ "type": "plain_text", "text": "Service" }, { "type": "plain_text", "text": "Status" }] },
    { "cells": [{ "type": "mrkdwn", "text": "API" }, { "type": "mrkdwn", "text": ":white_check_mark:" }] }
  ]
}
```

### 4.15 File Block (Messages only)

```json
{ "type": "file", "external_id": "ABCDE12345", "source": "remote" }
```

### 4.16 Markdown Block (Messages only)

```json
{ "type": "markdown", "text": "## Heading\n\nThis supports *full* markdown." }
```

### 4.17 Context Actions Block (Messages only)

```json
{
  "type": "context_actions",
  "elements": [
    { "type": "button", "text": { "type": "plain_text", "text": "Helpful" }, "action_id": "helpful" },
    { "type": "button", "text": { "type": "plain_text", "text": "Not Helpful" }, "action_id": "not_helpful" }
  ]
}
```

### 4.18 Plan Block (Messages only)

```json
{
  "type": "plan",
  "title": { "type": "plain_text", "text": "Release Checklist" },
  "sections": [
    {
      "title": { "type": "plain_text", "text": "Pre-release" },
      "items": [
        { "title": { "type": "plain_text", "text": "Run test suite" }, "checked": true },
        { "title": { "type": "plain_text", "text": "Update changelog" }, "checked": false }
      ]
    }
  ]
}
```

### 4.19 Task Card Block (Messages only)

```json
{
  "type": "task_card",
  "title": { "type": "plain_text", "text": "Review PR #423" },
  "description": { "type": "mrkdwn", "text": "Add rate limiting" },
  "status": { "label": { "type": "plain_text", "text": "In Progress" } }
}
```

## 5. Block Kit Elements (Interactive Components)

### 5.1 Button

```json
{
  "type": "button",
  "text": { "type": "plain_text", "text": "Click Me", "emoji": true },
  "action_id": "button_click",
  "value": "click_value",
  "style": "primary",
  "url": "https://example.com",
  "confirm": {
    "title": { "type": "plain_text", "text": "Are you sure?" },
    "text": { "type": "mrkdwn", "text": "This will trigger the deployment." },
    "confirm": { "type": "plain_text", "text": "Yes, deploy" },
    "deny": { "type": "plain_text", "text": "Cancel" }
  }
}
```

Styles: `primary` (green), `danger` (red), or omit for default (grey).

### 5.2 Select Menus

Types: `static_select`, `external_select`, `users_select`, `conversations_select`, `channels_select`.

```json
{
  "type": "static_select",
  "placeholder": { "type": "plain_text", "text": "Choose an option" },
  "action_id": "select_action",
  "options": [
    { "text": { "type": "plain_text", "text": "Option 1" }, "value": "opt_1" },
    { "text": { "type": "plain_text", "text": "Option 2" }, "value": "opt_2" }
  ]
}
```

### 5.3 Multi-Select Menus

Types: `multi_static_select`, `multi_external_select`, `multi_users_select`, `multi_conversations_select`, `multi_channels_select`.

```json
{
  "type": "multi_static_select",
  "placeholder": { "type": "plain_text", "text": "Select labels" },
  "action_id": "label_select",
  "max_selected_items": 5,
  "options": [
    { "text": { "type": "plain_text", "text": "Bug" }, "value": "bug" },
    { "text": { "type": "plain_text", "text": "Feature" }, "value": "feature" }
  ]
}
```

### 5.4 Date Picker

```json
{
  "type": "datepicker",
  "action_id": "date_select",
  "initial_date": "2025-05-29",
  "placeholder": { "type": "plain_text", "text": "Select a date" }
}
```

### 5.5 Time Picker

```json
{
  "type": "timepicker",
  "action_id": "time_select",
  "initial_time": "14:30",
  "placeholder": { "type": "plain_text", "text": "Select time" }
}
```

### 5.6 Datetime Picker

```json
{ "type": "datetimepicker", "action_id": "datetime_select", "initial_date_time": 1716000000 }
```

### 5.7 Overflow Menu

```json
{
  "type": "overflow",
  "action_id": "overflow_menu",
  "options": [
    { "text": { "type": "plain_text", "text": "Edit" }, "value": "edit" },
    { "text": { "type": "plain_text", "text": "Delete" }, "value": "delete" }
  ]
}
```

### 5.8 Radio Buttons

```json
{
  "type": "radio_buttons",
  "action_id": "priority_select",
  "options": [
    { "text": { "type": "plain_text", "text": "Low" }, "value": "low" },
    { "text": { "type": "plain_text", "text": "High" }, "value": "high" }
  ]
}
```

### 5.9 Checkboxes

```json
{
  "type": "checkboxes",
  "action_id": "checklist",
  "options": [
    { "text": { "type": "mrkdwn", "text": "*Code review* completed" }, "value": "review" },
    { "text": { "type": "mrkdwn", "text": "*Tests* passing" }, "value": "tests" }
  ]
}
```

### 5.10 Plain Text Input

```json
{
  "type": "plain_text_input",
  "action_id": "text_input",
  "multiline": true,
  "min_length": 10,
  "max_length": 3000,
  "placeholder": { "type": "plain_text", "text": "Describe the issue..." }
}
```

### 5.11 URL, Email, and Number Inputs

```json
{ "type": "url_text_input", "action_id": "url_input" }
{ "type": "email_text_input", "action_id": "email_input" }
{ "type": "number_input", "action_id": "num_input", "is_decimal_allowed": true, "min_value": "0", "max_value": "100" }
```

### 5.12 Rich Text Input

```json
{ "type": "rich_text_input", "action_id": "rich_text", "placeholder": { "type": "plain_text", "text": "Write here..." } }
```

### 5.13 File Input

```json
{ "type": "file_input", "action_id": "file_upload", "max_files": 5 }
```

## 6. Composition Objects

### Text Object

```json
{ "type": "mrkdwn", "text": "*bold* and _italic_" }
{ "type": "plain_text", "text": "No formatting", "emoji": true }
```

### Confirmation Dialog

```json
{
  "title": { "type": "plain_text", "text": "Confirm Action" },
  "text": { "type": "mrkdwn", "text": "Are you sure you want to proceed?" },
  "confirm": { "type": "plain_text", "text": "Yes" },
  "deny": { "type": "plain_text", "text": "No" },
  "style": "danger"
}
```

### Option Object

```json
{
  "text": { "type": "plain_text", "text": "Option Label" },
  "value": "option_value",
  "description": { "type": "plain_text", "text": "Additional context" }
}
```

### Option Group

```json
{
  "label": { "type": "plain_text", "text": "Group Name" },
  "options": [
    { "text": { "type": "plain_text", "text": "Item 1" }, "value": "item_1" }
  ]
}
```

### Filter Object (for conversation lists)

```json
{
  "include": ["public", "private", "mpim"],
  "exclude_bot_users": true,
  "exclude_external_shared_channels": true
}
```

## 7. Emoji and Reactions System

### Standard Emoji for Technical Communication

| Category | Emoji | Code | Use Case |
|----------|-------|------|----------|
| Status | ✅ | `:white_check_mark:` | Complete, approved |
| Status | ❌ | `:x:` | Failed, rejected |
| Status | ⚠️ | `:warning:` | Caution needed |
| Status | 🚨 | `:rotating_light:` | Critical alert |
| Status | ⏳ | `:hourglass:` | In progress |
| Priority | 🔥 | `:fire:` | Urgent |
| Priority | 🚀 | `:rocket:` | Launch, deploy |
| Feedback | 👍 | `:thumbsup:` | Agreement |
| Feedback | 👀 | `:eyes:` | Reviewing |
| Feedback | 🙌 | `:raised_hands:` | Celebration |
| Process | 📝 | `:memo:` | Documentation |
| Process | ⚙️ | `:gear:` | Configuration |
| Process | 🔒 | `:lock:` | Security |
| Process | 💡 | `:bulb:` | Idea |

### Reactions API

```bash
# Add a reaction
curl -X POST https://slack.com/api/reactions.add \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456","name":"thumbsup"}'

# Remove a reaction
curl -X POST https://slack.com/api/reactions.remove \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456","name":"thumbsup"}'

# Get reactions on a message
curl "https://slack.com/api/reactions.get?channel=C012AB3CD&timestamp=1234567890.123456" \
  -H "Authorization: Bearer xoxb-your-token"
```

Note: The `name` field uses the emoji shortcode WITHOUT colons.

### Custom Emoji

Custom emoji can be uploaded by workspace admins:
- Static images (PNG, GIF, JPG) up to 128KB
- Animated GIFs up to 256KB (paid plans)
- Recommended size: 128x128 pixels
- Aliases: multiple names pointing to the same image

Admin API for custom emoji:
```bash
# List all custom emoji
curl "https://slack.com/api/emoji.list" -H "Authorization: Bearer xoxb-your-token"

# Add custom emoji (admin token required)
curl -X POST https://slack.com/api/admin.emoji.add \
  -H "Authorization: Bearer xoxp-admin-token" \
  -F "name=custom_emoji" \
  -F "url=https://example.com/emoji.png"
```

## 8. Web API Essential Methods

### 8.1 Chat Methods

| Method | Description | Rate Tier |
|--------|-------------|-----------|
| `chat.postMessage` | Post a message to a channel | Tier 4 (1/sec per channel) |
| `chat.update` | Update an existing message | Tier 3 |
| `chat.delete` | Delete a message | Tier 3 |
| `chat.postEphemeral` | Post ephemeral message | Tier 4 |
| `chat.scheduleMessage` | Schedule a message | Tier 3 |
| `chat.unfurl` | Provide custom unfurl | Tier 3 |
| `chat.getPermalink` | Get message permalink | Tier 3 |
| `chat.meMessage` | Send /me message | Tier 3 |

### 8.2 Conversations Methods

| Method | Description | Rate Tier |
|--------|-------------|-----------|
| `conversations.list` | List channels | Tier 2 |
| `conversations.info` | Get channel info | Tier 3 |
| `conversations.create` | Create a channel | Tier 2 |
| `conversations.archive` | Archive a channel | Tier 2 |
| `conversations.invite` | Invite users to channel | Tier 3 |
| `conversations.kick` | Remove user from channel | Tier 3 |
| `conversations.join` | Join a channel | Tier 3 |
| `conversations.leave` | Leave a channel | Tier 3 |
| `conversations.members` | List channel members | Tier 3 |
| `conversations.history` | Fetch message history | Tier 3 |
| `conversations.replies` | Fetch thread replies | Tier 3 |
| `conversations.setPurpose` | Set channel purpose | Tier 2 |
| `conversations.setTopic` | Set channel topic | Tier 2 |

### 8.3 Users Methods

| Method | Description | Rate Tier |
|--------|-------------|-----------|
| `users.list` | List all users | Tier 2 |
| `users.info` | Get user info | Tier 4 |
| `users.lookupByEmail` | Find user by email | Tier 3 |
| `users.setPresence` | Set user presence | Tier 2 |
| `users.getPresence` | Get user presence | Tier 3 |
| `users.profile.get` | Get user profile | Tier 4 |
| `users.profile.set` | Set user profile | Tier 3 |

### 8.4 Files Methods

| Method | Description | Rate Tier |
|--------|-------------|-----------|
| `files.upload` | Upload a file | Tier 2 |
| `files.uploadV2` | Upload file (new method) | Tier 2 |
| `files.list` | List files | Tier 3 |
| `files.info` | Get file info | Tier 4 |
| `files.delete` | Delete a file | Tier 3 |
| `files.sharedPublicURL` | Share file publicly | Tier 3 |

### 8.5 Views Methods (Modals)

| Method | Description | Rate Tier |
|--------|-------------|-----------|
| `views.open` | Open a modal | Tier 4 |
| `views.update` | Update a modal | Tier 4 |
| `views.push` | Push a new view onto stack | Tier 4 |
| `views.publish` | Publish Home tab | Tier 4 |

## 9. Posting Messages - Complete Examples

### Basic Message with Blocks

```bash
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C012AB3CD",
    "text": "Deployment complete - api-gateway v2.5.0 to production",
    "blocks": [
      {
        "type": "header",
        "text": { "type": "plain_text", "text": "Deployment Complete :rocket:" }
      },
      {
        "type": "section",
        "fields": [
          { "type": "mrkdwn", "text": "*Service:*\n`api-gateway`" },
          { "type": "mrkdwn", "text": "*Environment:*\nProduction" },
          { "type": "mrkdwn", "text": "*Version:*\nv2.4.1 → v2.5.0" },
          { "type": "mrkdwn", "text": "*Duration:*\n3m 42s" }
        ]
      },
      {
        "type": "context",
        "elements": [
          { "type": "mrkdwn", "text": "Triggered by <@U012AB3CD> via GitHub Actions | <https://github.com/org/repo/actions/runs/123|View Run>" }
        ]
      }
    ],
    "unfurl_links": false,
    "unfurl_media": false
  }'
```

### Ephemeral Message

```bash
curl -X POST https://slack.com/api/chat.postEphemeral \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C012AB3CD",
    "user": "U012AB3CD",
    "text": "Only you can see this message"
  }'
```

### Threaded Reply

```bash
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C012AB3CD",
    "thread_ts": "1234567890.123456",
    "text": "This is a threaded reply",
    "reply_broadcast": false
  }'
```

### Scheduled Message

```bash
curl -X POST https://slack.com/api/chat.scheduleMessage \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C012AB3CD",
    "text": "Good morning! Daily standup in 15 minutes.",
    "post_at": 1716800000
  }'
```

## 10. Modals

### Opening a Modal

```bash
curl -X POST https://slack.com/api/views.open \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "trigger_id": "12345.98765.abcd2358fdea",
    "view": {
      "type": "modal",
      "callback_id": "create_ticket",
      "title": { "type": "plain_text", "text": "Create Ticket" },
      "submit": { "type": "plain_text", "text": "Submit" },
      "close": { "type": "plain_text", "text": "Cancel" },
      "blocks": [
        {
          "type": "input",
          "block_id": "title_block",
          "label": { "type": "plain_text", "text": "Title" },
          "element": { "type": "plain_text_input", "action_id": "title_input" }
        },
        {
          "type": "input",
          "block_id": "priority_block",
          "label": { "type": "plain_text", "text": "Priority" },
          "element": {
            "type": "static_select",
            "action_id": "priority_select",
            "options": [
              { "text": { "type": "plain_text", "text": "Low" }, "value": "low" },
              { "text": { "type": "plain_text", "text": "Medium" }, "value": "medium" },
              { "text": { "type": "plain_text", "text": "High" }, "value": "high" }
            ]
          }
        },
        {
          "type": "input",
          "block_id": "desc_block",
          "label": { "type": "plain_text", "text": "Description" },
          "element": { "type": "plain_text_input", "action_id": "desc_input", "multiline": true },
          "optional": true
        }
      ]
    }
  }'
```

### Handling Modal Submission

When a user submits a modal, Slack sends a `view_submission` payload. Your app must respond within 3 seconds with one of:

```json
{"response_action": "clear"}
```

```json
{"response_action": "update", "view": { "type": "modal", "title": {...}, "blocks": [...] }}
```

```json
{"response_action": "push", "view": { "type": "modal", "title": {...}, "blocks": [...] }}
```

```json
{"response_action": "errors", "errors": {"title_block": "Title is required"}}
```

## 11. Home Tab

```bash
curl -X POST https://slack.com/api/views.publish \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "U012AB3CD",
    "view": {
      "type": "home",
      "blocks": [
        { "type": "header", "text": { "type": "plain_text", "text": "Welcome to AppBot" } },
        { "type": "section", "text": { "type": "mrkdwn", "text": "Your active tasks:" } },
        { "type": "divider" },
        {
          "type": "section",
          "text": { "type": "mrkdwn", "text": ":memo: *Review PR #423*\nDue: Today" },
          "accessory": {
            "type": "button",
            "text": { "type": "plain_text", "text": "Open" },
            "url": "https://github.com/org/repo/pull/423",
            "action_id": "open_pr"
          }
        }
      ]
    }
  }'
```

## 12. Slash Commands

### Incoming Payload

When a user types `/deploy production api-gateway v2.5.0`, Slack sends:

```json
{
  "token": "verification_token",
  "team_id": "T012AB3CD",
  "channel_id": "C012AB3CD",
  "user_id": "U012AB3CD",
  "command": "/deploy",
  "text": "production api-gateway v2.5.0",
  "response_url": "https://hooks.slack.com/commands/T012/123/abc",
  "trigger_id": "12345.98765.abcd"
}
```

### Response Types

- `in_channel` — visible to everyone
- `ephemeral` — visible only to the invoking user

For operations taking longer than 3 seconds, acknowledge immediately with HTTP 200 and use the `response_url` to send up to 5 follow-up messages within 30 minutes.

## 13. Incoming Webhooks

```bash
curl -X POST https://hooks.slack.com/services/TXXXXX/BXXXXX/your-webhook-token \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Alert: CPU usage exceeded 90%",
    "blocks": [
      {
        "type": "section",
        "text": { "type": "mrkdwn", "text": ":rotating_light: *CPU Alert*\nServer `prod-web-03` at *94%* for 5 minutes." }
      },
      {
        "type": "actions",
        "elements": [
          { "type": "button", "text": { "type": "plain_text", "text": "View Dashboard" }, "url": "https://grafana.example.com/d/cpu", "action_id": "view_dashboard" }
        ]
      }
    ]
  }'
```

## 14. Rate Limits

| Tier | Rate | Common Methods |
|------|------|----------------|
| Tier 1 | 1 request/min | admin.*, some legacy |
| Tier 2 | 20 requests/min | files.upload, conversations.create |
| Tier 3 | 50 requests/min | conversations.list, users.list |
| Tier 4 | 100+ requests/min | chat.postMessage, reactions.add |
| Special | 1 request/sec | chat.postMessage per channel |

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 98
X-RateLimit-Reset: 1716000060
Retry-After: 30  (only on 429 responses)
```

## 15. Production Message Templates

### Incident Alert

```json
{
  "blocks": [
    { "type": "header", "text": { "type": "plain_text", "text": ":rotating_light: Incident Detected" } },
    {
      "type": "section",
      "text": { "type": "mrkdwn", "text": "*Severity:* P1 - Critical\n*Service:* Payment Processing\n*Impact:* Users unable to complete checkout" }
    },
    {
      "type": "actions",
      "elements": [
        { "type": "button", "text": { "type": "plain_text", "text": "Acknowledge" }, "style": "primary", "action_id": "ack_incident" },
        { "type": "button", "text": { "type": "plain_text", "text": "Escalate" }, "style": "danger", "action_id": "escalate" }
      ]
    }
  ]
}
```

### PR Review Request

```json
{
  "blocks": [
    {
      "type": "section",
      "text": { "type": "mrkdwn", "text": ":mag: *Code Review Requested*\n<https://github.com/org/repo/pull/423|#423 - Add rate limiting>\n_+342 / -28 lines across 5 files_" }
    },
    {
      "type": "section",
      "fields": [
        { "type": "mrkdwn", "text": "*Author:*\n<@U012AB3CD>" },
        { "type": "mrkdwn", "text": "*CI:*\n:white_check_mark: Passing" }
      ]
    }
  ]
}
```

### Daily Standup

```json
{
  "blocks": [
    { "type": "header", "text": { "type": "plain_text", "text": ":calendar: Daily Standup - May 29, 2025" } },
    { "type": "divider" },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*<@U001> Alice*\n• _Yesterday:_ Completed rate limiting PR\n• _Today:_ Database migration\n• _Blockers:_ None" } },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*<@U002> Bob*\n• _Yesterday:_ Fixed webhook timeout\n• _Today:_ Load testing\n• _Blockers:_ Need staging Redis access" } }
  ]
}
```

## 16. Accessibility Best Practices

When posting messages with blocks, screen readers default to reading the top-level `text` field. To ensure accessibility:

1. Always include a meaningful `text` field alongside `blocks`
2. Provide descriptive `alt_text` for all images
3. Use `plain_text` for critical information
4. Avoid conveying meaning through color or emoji alone
5. Structure blocks in a logical reading order

## 17. Token Types

| Token Type | Prefix | Use Case |
|-----------|--------|----------|
| Bot token | `xoxb-` | App actions on behalf of the bot |
| User token | `xoxp-` | Actions on behalf of a user |
| App-level token | `xapp-` | Socket Mode, connections |
| Webhook URL | `https://hooks.slack.com/...` | Simple message posting |
| Signing secret | (hex string) | Request verification |

## 18. Request Verification

Verify incoming requests using the signing secret:

```python
import hashlib
import hmac
import time

def verify_slack_request(signing_secret, timestamp, body, signature):
    if abs(time.time() - int(timestamp)) > 60 * 5:
        return False  # Request too old
    sig_basestring = f"v0:{timestamp}:{body}"
    my_signature = "v0=" + hmac.new(
        signing_secret.encode(), sig_basestring.encode(), hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(my_signature, signature)
```

Headers to check: `X-Slack-Request-Timestamp` and `X-Slack-Signature`.

## 19. Events API

The Events API allows your app to receive real-time notifications about events in Slack workspaces. Events are delivered via HTTP POST to your configured Request URL.

### Event Subscription Types

| Event | Description | Scope Required |
|-------|-------------|----------------|
| `message.channels` | Message posted to public channel | `channels:history` |
| `message.groups` | Message posted to private channel | `groups:history` |
| `message.im` | Message posted in DM | `im:history` |
| `message.mpim` | Message posted in group DM | `mpim:history` |
| `app_mention` | App mentioned in a message | `app_mentions:read` |
| `member_joined_channel` | User joined a channel | `channels:read` |
| `member_left_channel` | User left a channel | `channels:read` |
| `channel_created` | New channel created | `channels:read` |
| `reaction_added` | Reaction added to message | `reactions:read` |
| `reaction_removed` | Reaction removed from message | `reactions:read` |
| `app_home_opened` | User opened app Home tab | None |
| `file_shared` | File shared in channel | `files:read` |
| `team_join` | New user joined workspace | `users:read` |
| `user_change` | User profile updated | `users:read` |
| `workflow_step_execute` | Workflow step triggered | `workflow.steps:execute` |

### Event Envelope Structure

```json
{
  "token": "verification_token",
  "team_id": "T012AB3CD",
  "api_app_id": "A012AB3CD",
  "event": {
    "type": "message",
    "subtype": null,
    "channel": "C012AB3CD",
    "user": "U012AB3CD",
    "text": "Hello world",
    "ts": "1234567890.123456",
    "event_ts": "1234567890.123456",
    "channel_type": "channel"
  },
  "type": "event_callback",
  "event_id": "Ev012AB3CD",
  "event_time": 1234567890,
  "authorizations": [
    {
      "enterprise_id": "E012AB3CD",
      "team_id": "T012AB3CD",
      "user_id": "U012AB3CD",
      "is_bot": true,
      "is_enterprise_install": false
    }
  ]
}
```

### URL Verification Challenge

When you first configure your Request URL, Slack sends a challenge:

```json
{ "token": "verification_token", "challenge": "3eZbrw1aBm2rZgRNFdxV2595E9CY3gmdALWMmHkvFXO7tYXAYM8P", "type": "url_verification" }
```

Your app must respond with: `{"challenge": "3eZbrw1aBm2rZgRNFdxV2595E9CY3gmdALWMmHkvFXO7tYXAYM8P"}`

### Best Practices for Events

1. Respond with HTTP 200 within 3 seconds (acknowledge receipt)
2. Process events asynchronously (queue for processing)
3. Handle duplicate events (use `event_id` for deduplication)
4. Implement retry logic (Slack retries after failures with `X-Slack-Retry-Num` header)
5. Filter by `subtype` to ignore bot messages and avoid loops

## 20. Socket Mode

Socket Mode allows your app to receive events over a WebSocket connection instead of HTTP, eliminating the need for a public URL.

### Setup

```bash
# Install the Slack SDK
pip install slack-sdk

# Or for Node.js
npm install @slack/socket-mode @slack/bolt
```

### Python Example

```python
from slack_sdk.socket_mode import SocketModeClient
from slack_sdk.web import WebClient
from slack_sdk.socket_mode.request import SocketModeRequest
from slack_sdk.socket_mode.response import SocketModeResponse

client = SocketModeClient(
    app_token="xapp-1-A012AB3CD-1234567890-abcdef",
    web_client=WebClient(token="xoxb-your-bot-token")
)

def handle_events(client: SocketModeClient, req: SocketModeRequest):
    if req.type == "events_api":
        event = req.payload["event"]
        if event["type"] == "app_mention":
            client.web_client.chat_postMessage(
                channel=event["channel"],
                thread_ts=event["ts"],
                text=f"Hello <@{event['user']}>! How can I help?"
            )
    client.send_socket_mode_response(SocketModeResponse(envelope_id=req.envelope_id))

client.socket_mode_request_listeners.append(handle_events)
client.connect()

import time
while True:
    time.sleep(1)
```

### Node.js Bolt Example

```javascript
const { App } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  appToken: process.env.SLACK_APP_TOKEN,
  socketMode: true,
});

app.event('app_mention', async ({ event, say }) => {
  await say({
    thread_ts: event.ts,
    text: `Hello <@${event.user}>! How can I help?`
  });
});

app.command('/deploy', async ({ command, ack, respond }) => {
  await ack();
  const [env, service, version] = command.text.split(' ');
  await respond({
    response_type: 'in_channel',
    text: `Deploying ${service} ${version} to ${env}...`
  });
});

(async () => { await app.start(); console.log('App running'); })();
```

## 21. Workflow Builder and Automation

### Workflow Steps from Apps (Legacy)

Apps can contribute custom steps to Workflow Builder:

```python
@app.step("create_ticket")
def handle_step(ack, step, configure, update, fail):
    ack()
    # Step configuration
    inputs = step["inputs"]
    title = inputs["title"]["value"]
    priority = inputs["priority"]["value"]
    # Process the step
    try:
        ticket_id = create_jira_ticket(title, priority)
        update(outputs=[{"name": "ticket_id", "value": ticket_id}])
    except Exception as e:
        fail(error={"message": str(e)})
```

### New Platform Functions (Next-Gen)

Slack's next-generation platform uses Deno-based functions:

```typescript
import { DefineFunction, Schema, SlackFunction } from "deno-slack-sdk/mod.ts";

export const CreateTicketFunction = DefineFunction({
  callback_id: "create_ticket",
  title: "Create Ticket",
  source_file: "functions/create_ticket.ts",
  input_parameters: {
    properties: {
      title: { type: Schema.types.string, description: "Ticket title" },
      priority: { type: Schema.types.string, enum: ["low", "medium", "high"] },
      reporter: { type: Schema.slack.types.user_id }
    },
    required: ["title", "priority", "reporter"]
  },
  output_parameters: {
    properties: {
      ticket_id: { type: Schema.types.string },
      ticket_url: { type: Schema.types.string }
    },
    required: ["ticket_id"]
  }
});

export default SlackFunction(CreateTicketFunction, async ({ inputs, client }) => {
  const { title, priority, reporter } = inputs;
  const ticketId = `TICK-${Date.now()}`;
  
  await client.chat.postMessage({
    channel: reporter,
    text: `Ticket ${ticketId} created: ${title} (${priority})`
  });
  
  return { outputs: { ticket_id: ticketId, ticket_url: `https://tickets.example.com/${ticketId}` } };
});
```

## 22. Slack Connect (Cross-Organization Channels)

Slack Connect allows organizations to communicate in shared channels. Key considerations:

- Shared channels have a `is_shared` flag set to true
- External users have `is_stranger` flag in user objects
- Messages from external users include `team` field with their workspace ID
- File sharing can be restricted across organizations
- Apps need `channels:read` scope to see shared channel membership
- DLP and compliance policies apply per-organization

### Identifying External Users

```python
def is_external_user(user_info, my_team_id):
    return user_info.get("team_id") != my_team_id

# Check in message events
if event.get("team") and event["team"] != MY_TEAM_ID:
    # Message from external user
    pass
```

## 23. File Uploads

### Modern Upload (files.uploadV2)

```bash
# Step 1: Get upload URL
curl -X POST https://slack.com/api/files.getUploadURLExternal \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"filename":"report.pdf","length":1048576}'

# Step 2: Upload to the returned URL
curl -X POST "https://files.slack.com/upload/v1/..." \
  -F "file=@report.pdf"

# Step 3: Complete upload
curl -X POST https://slack.com/api/files.completeUploadExternal \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "files": [{"id":"F012AB3CD","title":"Monthly Report"}],
    "channel_id": "C012AB3CD",
    "initial_comment": "Here is the monthly report :memo:"
  }'
```

### File Sharing Snippet (Code)

```bash
curl -X POST https://slack.com/api/files.upload \
  -H "Authorization: Bearer xoxb-your-token" \
  -F "channels=C012AB3CD" \
  -F "content=def hello():\n    print('Hello, World!')" \
  -F "filename=hello.py" \
  -F "filetype=python" \
  -F "title=Hello World Script" \
  -F "initial_comment=Here's the code snippet you requested"
```

## 24. Interactivity Payloads

When users interact with Block Kit elements, Slack sends interaction payloads to your configured Interactivity Request URL.

### Payload Types

| Type | Trigger |
|------|---------|
| `block_actions` | User clicks button, selects option, etc. |
| `view_submission` | User submits a modal |
| `view_closed` | User closes a modal |
| `shortcut` | User triggers a global or message shortcut |
| `message_action` | User triggers a message action (legacy) |

### block_actions Payload

```json
{
  "type": "block_actions",
  "user": { "id": "U012AB3CD", "username": "johndoe", "name": "John Doe" },
  "trigger_id": "12345.98765.abcd",
  "channel": { "id": "C012AB3CD", "name": "general" },
  "message": { "ts": "1234567890.123456", "text": "Original message" },
  "actions": [
    {
      "type": "button",
      "action_id": "approve_request",
      "block_id": "actions_block",
      "text": { "type": "plain_text", "text": "Approve" },
      "value": "req_001",
      "action_ts": "1234567890.123456"
    }
  ]
}
```

### Responding to Interactions

1. Acknowledge with HTTP 200 within 3 seconds
2. Use `response_url` for follow-up messages (up to 5 within 30 min)
3. Use `trigger_id` to open modals (valid for 3 seconds)
4. Update the original message by returning a new message payload

## 25. App Distribution and OAuth

### OAuth 2.0 Flow

```
1. User clicks "Add to Slack" button
2. Redirect to: https://slack.com/oauth/v2/authorize?client_id=CLIENT_ID&scope=SCOPES&redirect_uri=REDIRECT_URI
3. User authorizes the app
4. Slack redirects to your redirect_uri with a code parameter
5. Exchange code for token:
   POST https://slack.com/api/oauth.v2.access
   client_id=CLIENT_ID&client_secret=CLIENT_SECRET&code=CODE&redirect_uri=REDIRECT_URI
6. Store the returned access_token (xoxb-...) and team info
```

### Bot Token Scopes (Common)

| Scope | Grants |
|-------|--------|
| `chat:write` | Post messages as the bot |
| `chat:write.public` | Post to channels without joining |
| `channels:read` | View public channel info |
| `channels:history` | Read public channel messages |
| `groups:read` | View private channel info |
| `im:read` | View DM info |
| `im:write` | Send DMs |
| `users:read` | View user info |
| `users:read.email` | View user email addresses |
| `reactions:read` | View reactions |
| `reactions:write` | Add/remove reactions |
| `files:read` | View files |
| `files:write` | Upload/modify files |
| `commands` | Add slash commands |
| `app_mentions:read` | Receive app_mention events |

## 26. Bolt Framework Patterns

### Python Bolt

```python
from slack_bolt import App
from slack_bolt.adapter.socket_mode import SocketModeHandler

app = App(token=os.environ["SLACK_BOT_TOKEN"])

# Listen for messages containing "hello"
@app.message("hello")
def handle_hello(message, say):
    say(f"Hey there <@{message['user']}>!")

# Listen for slash command
@app.command("/ticket")
def handle_ticket(ack, command, client):
    ack()
    client.views_open(
        trigger_id=command["trigger_id"],
        view={
            "type": "modal",
            "callback_id": "ticket_modal",
            "title": {"type": "plain_text", "text": "New Ticket"},
            "submit": {"type": "plain_text", "text": "Create"},
            "blocks": [
                {
                    "type": "input",
                    "block_id": "title_block",
                    "label": {"type": "plain_text", "text": "Title"},
                    "element": {"type": "plain_text_input", "action_id": "title_input"}
                }
            ]
        }
    )

# Handle modal submission
@app.view("ticket_modal")
def handle_submission(ack, body, client, view):
    title = view["state"]["values"]["title_block"]["title_input"]["value"]
    user = body["user"]["id"]
    ack()
    client.chat_postMessage(
        channel=user,
        text=f"Ticket created: {title}"
    )

# Handle button click
@app.action("approve_request")
def handle_approve(ack, body, client):
    ack()
    client.chat_update(
        channel=body["channel"]["id"],
        ts=body["message"]["ts"],
        text="Request approved!",
        blocks=[
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": ":white_check_mark: *Approved* by <@" + body["user"]["id"] + ">"}
            }
        ]
    )

if __name__ == "__main__":
    SocketModeHandler(app, os.environ["SLACK_APP_TOKEN"]).start()
```

### JavaScript Bolt

```javascript
const { App } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
});

// Respond to app_mention
app.event('app_mention', async ({ event, client }) => {
  await client.chat.postMessage({
    channel: event.channel,
    thread_ts: event.ts,
    blocks: [
      {
        type: 'section',
        text: { type: 'mrkdwn', text: `Hi <@${event.user}>! What can I help with?` },
        accessory: {
          type: 'button',
          text: { type: 'plain_text', text: 'Open Help' },
          action_id: 'open_help'
        }
      }
    ]
  });
});

// Handle overflow menu selection
app.action('task_overflow', async ({ ack, action, body, client }) => {
  await ack();
  const selectedValue = action.selected_option.value;
  if (selectedValue === 'delete') {
    await client.chat.delete({
      channel: body.channel.id,
      ts: body.message.ts
    });
  }
});

(async () => { await app.start(3000); })();
```

## 27. Admin API (Enterprise Grid)

### User Management

```bash
# Invite a user to workspace
curl -X POST https://slack.com/api/admin.users.invite \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"team_id":"T012AB3CD","email":"new.user@company.com","channel_ids":"C012AB3CD,C034EF5GH"}'

# Deactivate a user
curl -X POST https://slack.com/api/admin.users.remove \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"team_id":"T012AB3CD","user_id":"U012AB3CD"}'

# Set user to admin
curl -X POST https://slack.com/api/admin.users.setAdmin \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"team_id":"T012AB3CD","user_id":"U012AB3CD"}'
```

### Channel Management

```bash
# Archive a channel
curl -X POST https://slack.com/api/admin.conversations.archive \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"channel_id":"C012AB3CD"}'

# Set channel retention
curl -X POST https://slack.com/api/admin.conversations.setCustomRetention \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"channel_id":"C012AB3CD","duration_days":90}'
```

## 28. Slack App Manifest

Define your entire app configuration in YAML:

```yaml
display_information:
  name: DeployBot
  description: Automated deployment notifications and controls
  background_color: "#2c2d30"
  long_description: "DeployBot provides real-time deployment notifications, rollback controls, and environment status monitoring for your engineering team."

features:
  bot_user:
    display_name: DeployBot
    always_online: true
  slash_commands:
    - command: /deploy
      url: https://your-app.example.com/slack/commands
      description: Trigger a deployment
      usage_hint: "[environment] [service] [version]"
      should_escape: false
    - command: /rollback
      url: https://your-app.example.com/slack/commands
      description: Rollback a deployment
      usage_hint: "[service] [version]"
  shortcuts:
    - name: Create Incident
      type: global
      callback_id: create_incident
      description: Create a new incident report

oauth_config:
  scopes:
    bot:
      - chat:write
      - chat:write.public
      - commands
      - channels:read
      - channels:history
      - reactions:write
      - users:read
      - app_mentions:read
      - files:write

settings:
  event_subscriptions:
    request_url: https://your-app.example.com/slack/events
    bot_events:
      - app_mention
      - message.channels
      - reaction_added
  interactivity:
    is_enabled: true
    request_url: https://your-app.example.com/slack/interactions
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
```

## 29. Message Unfurling

When URLs are posted in Slack, apps can provide custom unfurls:

```python
@app.event("link_shared")
def handle_link_shared(event, client):
    links = event["links"]
    unfurls = {}
    
    for link in links:
        url = link["url"]
        if "jira.example.com" in url:
            ticket = fetch_jira_ticket(url)
            unfurls[url] = {
                "blocks": [
                    {
                        "type": "section",
                        "text": {
                            "type": "mrkdwn",
                            "text": f"*{ticket['key']}*: {ticket['summary']}\nStatus: {ticket['status']} | Priority: {ticket['priority']}"
                        }
                    }
                ]
            }
    
    client.chat_unfurl(
        channel=event["channel"],
        ts=event["message_ts"],
        unfurls=unfurls
    )
```

## 30. Search API

```bash
# Search messages
curl "https://slack.com/api/search.messages?query=deployment+failed&sort=timestamp&sort_dir=desc&count=20" \
  -H "Authorization: Bearer xoxp-user-token"

# Search files
curl "https://slack.com/api/search.files?query=report+Q1+2025&types=pdf,xlsx" \
  -H "Authorization: Bearer xoxp-user-token"
```

Note: Search requires a user token (xoxp-), not a bot token.

## 31. User Groups (Teams/Handles)

```bash
# Create a user group
curl -X POST https://slack.com/api/usergroups.create \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"name":"On-Call Engineers","handle":"oncall","description":"Current on-call rotation"}'

# Update members
curl -X POST https://slack.com/api/usergroups.users.update \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"usergroup":"S012AB3CD","users":"U001,U002,U003"}'

# List user groups
curl "https://slack.com/api/usergroups.list?include_users=true" \
  -H "Authorization: Bearer xoxb-your-token"
```

## 32. Bookmarks API

```bash
# Add a bookmark to a channel
curl -X POST https://slack.com/api/bookmarks.add \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel_id": "C012AB3CD",
    "title": "Runbook",
    "type": "link",
    "link": "https://wiki.example.com/runbook",
    "emoji": ":book:"
  }'
```

## 33. Reminders API

```bash
# Set a reminder
curl -X POST https://slack.com/api/reminders.add \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Review the PR before end of day",
    "time": "in 2 hours",
    "user": "U012AB3CD"
  }'

# List reminders
curl "https://slack.com/api/reminders.list" \
  -H "Authorization: Bearer xoxb-your-token"
```

## 34. Pins and Stars

```bash
# Pin a message
curl -X POST https://slack.com/api/pins.add \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C012AB3CD","timestamp":"1234567890.123456"}'

# List pinned items
curl "https://slack.com/api/pins.list?channel=C012AB3CD" \
  -H "Authorization: Bearer xoxb-your-token"
```

## 35. Best Practices Summary

| Area | Best Practice |
|------|--------------|
| Formatting | Use mrkdwn sparingly; prefer clarity over decoration |
| Blocks | Always include fallback `text` field with blocks |
| Buttons | Limit to 5 buttons per actions block for usability |
| Modals | Keep to 3-5 input fields; use multi-step for complex forms |
| Rate limits | Implement exponential backoff; respect Retry-After header |
| Threads | Use threads for detailed discussions; keep channels clean |
| Emoji | Use reactions for quick feedback instead of reply messages |
| Webhooks | Validate webhook URLs; never expose in client-side code |
| Tokens | Use bot tokens (xoxb-) for app actions; never expose in logs |
| Error handling | Always check `ok` field in API responses |
| Events | Acknowledge within 3 seconds; process asynchronously |
| Socket Mode | Use for development and apps without public URLs |
| Unfurls | Cache external data to avoid rate limits |
| Accessibility | Always provide alt_text and fallback text |
| Security | Verify request signatures on all incoming payloads |

## 36. Conversation Canvas

Canvases are collaborative documents embedded within channels. They support rich formatting, checklists, code blocks, and embedded Slack content.

### Creating a Canvas

```bash
curl -X POST https://slack.com/api/canvases.create \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Sprint Planning - Week 22",
    "document_content": {
      "type": "markdown",
      "markdown": "# Sprint Planning\n\n## Goals\n- [ ] Complete API rate limiting\n- [ ] Deploy monitoring stack\n- [ ] Review security audit findings\n\n## Notes\nPlease add your items below."
    }
  }'
```

### Canvas Sections API

```bash
# Edit canvas sections
curl -X POST https://slack.com/api/canvases.sections.lookup \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{"canvas_id":"F012AB3CD","criteria":{"section_types":["any_header"],"contains_text":"Goals"}}'
```

## 37. Status and Presence

### Setting User Status

```bash
curl -X POST https://slack.com/api/users.profile.set \
  -H "Authorization: Bearer xoxp-user-token" \
  -H "Content-Type: application/json" \
  -d '{
    "profile": {
      "status_text": "In a meeting",
      "status_emoji": ":calendar:",
      "status_expiration": 1716003600
    }
  }'
```

### Common Status Patterns for Engineering Teams

| Status | Emoji | Text | Duration |
|--------|-------|------|----------|
| Focus time | `:headphones:` | "Deep work - no interruptions" | 2 hours |
| In meeting | `:calendar:` | "In a meeting" | Until meeting ends |
| On call | `:pager:` | "On-call rotation" | 24 hours |
| Deploying | `:rocket:` | "Deploying to production" | 30 minutes |
| Lunch | `:hamburger:` | "Lunch break" | 1 hour |
| PTO | `:palm_tree:` | "Out of office" | Days |
| Sick | `:face_with_thermometer:` | "Out sick" | Days |
| Commuting | `:bus:` | "Commuting" | 1 hour |

### Do Not Disturb

```bash
# Set DND for 2 hours
curl -X POST https://slack.com/api/dnd.setSnooze \
  -H "Authorization: Bearer xoxp-user-token" \
  -H "Content-Type: application/json" \
  -d '{"num_minutes":120}'

# End DND
curl -X POST https://slack.com/api/dnd.endSnooze \
  -H "Authorization: Bearer xoxp-user-token"

# Check DND status
curl "https://slack.com/api/dnd.info?user=U012AB3CD" \
  -H "Authorization: Bearer xoxb-your-token"
```

## 38. Channel Management Patterns

### Naming Conventions

| Pattern | Example | Use Case |
|---------|---------|----------|
| `#team-{name}` | `#team-platform` | Team channels |
| `#proj-{name}` | `#proj-migration` | Project channels |
| `#inc-{id}` | `#inc-2025-0042` | Incident channels |
| `#help-{topic}` | `#help-kubernetes` | Support channels |
| `#announce-{scope}` | `#announce-engineering` | Announcements (restricted posting) |
| `#ext-{company}` | `#ext-acme-corp` | Slack Connect channels |
| `#bot-{name}` | `#bot-alerts` | Bot notification channels |
| `#tmp-{purpose}` | `#tmp-hackathon-2025` | Temporary channels |

### Automated Channel Creation for Incidents

```python
async def create_incident_channel(client, incident_id, severity, title):
    channel_name = f"inc-{incident_id}"
    
    # Create the channel
    result = await client.conversations_create(name=channel_name, is_private=False)
    channel_id = result["channel"]["id"]
    
    # Set topic
    await client.conversations_setTopic(
        channel=channel_id,
        topic=f"P{severity} | {title} | Status: Investigating"
    )
    
    # Set purpose
    await client.conversations_setPurpose(
        channel=channel_id,
        purpose=f"Incident {incident_id}: {title}. See pinned message for details."
    )
    
    # Invite on-call team
    oncall_users = get_oncall_users()
    await client.conversations_invite(channel=channel_id, users=",".join(oncall_users))
    
    # Post initial message and pin it
    msg = await client.chat_postMessage(
        channel=channel_id,
        blocks=[
            {"type": "header", "text": {"type": "plain_text", "text": f":rotating_light: Incident {incident_id}"}},
            {"type": "section", "fields": [
                {"type": "mrkdwn", "text": f"*Severity:* P{severity}"},
                {"type": "mrkdwn", "text": f"*Title:* {title}"},
                {"type": "mrkdwn", "text": f"*Status:* Investigating"},
                {"type": "mrkdwn", "text": f"*Commander:* TBD"}
            ]},
            {"type": "actions", "elements": [
                {"type": "button", "text": {"type": "plain_text", "text": "Claim Commander"}, "style": "primary", "action_id": "claim_commander"},
                {"type": "button", "text": {"type": "plain_text", "text": "Update Status"}, "action_id": "update_status"},
                {"type": "button", "text": {"type": "plain_text", "text": "Resolve"}, "style": "danger", "action_id": "resolve_incident"}
            ]}
        ]
    )
    
    await client.pins_add(channel=channel_id, timestamp=msg["ts"])
    return channel_id
```

## 39. Message Metadata

Attach structured metadata to messages for programmatic handling:

```bash
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer xoxb-your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C012AB3CD",
    "text": "Deployment started for api-gateway",
    "metadata": {
      "event_type": "deployment_started",
      "event_payload": {
        "service": "api-gateway",
        "version": "v2.5.0",
        "environment": "production",
        "triggered_by": "U012AB3CD",
        "commit_sha": "abc123def456"
      }
    }
  }'
```

Subscribe to metadata events:
```json
{
  "type": "event_callback",
  "event": {
    "type": "message_metadata_posted",
    "metadata": {
      "event_type": "deployment_started",
      "event_payload": { "service": "api-gateway", "version": "v2.5.0" }
    }
  }
}
```

## 40. Pagination

All list methods use cursor-based pagination:

```python
def get_all_channels(client):
    channels = []
    cursor = None
    
    while True:
        result = client.conversations_list(
            types="public_channel,private_channel",
            limit=200,
            cursor=cursor
        )
        channels.extend(result["channels"])
        
        cursor = result.get("response_metadata", {}).get("next_cursor")
        if not cursor:
            break
    
    return channels
```

Key pagination parameters:
- `limit`: Number of items per page (max varies by method, typically 100-1000)
- `cursor`: Opaque string for the next page
- Response includes `response_metadata.next_cursor` (empty string means no more pages)

## 41. Error Handling

### Common Error Codes

| Error | Meaning | Resolution |
|-------|---------|------------|
| `not_authed` | No token provided | Include Authorization header |
| `invalid_auth` | Token is invalid | Regenerate token |
| `token_revoked` | Token was revoked | Re-authorize the app |
| `channel_not_found` | Channel doesn't exist or bot not in it | Join channel or check ID |
| `not_in_channel` | Bot not a member | Invite bot to channel |
| `is_archived` | Channel is archived | Unarchive or use different channel |
| `msg_too_long` | Message exceeds 40,000 chars | Split into multiple messages |
| `too_many_attachments` | More than 50 blocks | Reduce block count |
| `rate_limited` | Hit rate limit | Wait for Retry-After seconds |
| `missing_scope` | Token lacks required scope | Add scope and re-authorize |
| `user_not_found` | Invalid user ID | Verify user ID exists |
| `invalid_blocks` | Block Kit JSON is malformed | Validate with Block Kit Builder |
| `no_text` | Message has no text or blocks | Include text or blocks field |
| `ekm_access_denied` | EKM key not available | Contact workspace admin |

### Robust Error Handling Pattern

```python
import time
from slack_sdk.errors import SlackApiError

def post_message_with_retry(client, channel, text, blocks=None, max_retries=3):
    for attempt in range(max_retries):
        try:
            result = client.chat_postMessage(
                channel=channel,
                text=text,
                blocks=blocks
            )
            if result["ok"]:
                return result
        except SlackApiError as e:
            error = e.response["error"]
            
            if error == "rate_limited":
                retry_after = int(e.response.headers.get("Retry-After", 30))
                time.sleep(retry_after)
                continue
            elif error == "not_in_channel":
                client.conversations_join(channel=channel)
                continue
            elif error == "channel_not_found":
                raise ValueError(f"Channel {channel} not found")
            elif error in ("token_revoked", "invalid_auth"):
                raise AuthenticationError("Token invalid, re-auth required")
            else:
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)
                    continue
                raise
    
    raise RuntimeError(f"Failed after {max_retries} attempts")
```

## 42. Unfurl Domains

Register domains your app can unfurl:

```yaml
# In app manifest
features:
  unfurl_domains:
    - "jira.example.com"
    - "confluence.example.com"
    - "github.com"
```

When a URL matching your registered domain is posted, Slack sends a `link_shared` event. Your app has 30 minutes to provide an unfurl.

## 43. Message Scheduling Patterns

### Recurring Messages (Using External Scheduler)

```python
import schedule
import time

def post_standup_reminder():
    client.chat_postMessage(
        channel="C012AB3CD",
        text="Time for standup! Please post your update.",
        blocks=[
            {"type": "header", "text": {"type": "plain_text", "text": ":calendar: Daily Standup"}},
            {"type": "section", "text": {"type": "mrkdwn", "text": "Please share:\n• What you did yesterday\n• What you're doing today\n• Any blockers"}},
            {"type": "actions", "elements": [
                {"type": "button", "text": {"type": "plain_text", "text": "Post Update"}, "action_id": "post_standup", "style": "primary"}
            ]}
        ]
    )

schedule.every().monday.at("09:00").do(post_standup_reminder)
schedule.every().tuesday.at("09:00").do(post_standup_reminder)
schedule.every().wednesday.at("09:00").do(post_standup_reminder)
schedule.every().thursday.at("09:00").do(post_standup_reminder)
schedule.every().friday.at("09:00").do(post_standup_reminder)

while True:
    schedule.run_pending()
    time.sleep(60)
```

## 44. Channel Topic and Purpose Automation

```python
async def update_oncall_topic(client, channel_id, primary, secondary):
    topic = f":pager: On-call: <@{primary}> (primary) | <@{secondary}> (backup) | Escalation: #inc-response"
    await client.conversations_setTopic(channel=channel_id, topic=topic)

async def update_sprint_purpose(client, channel_id, sprint_num, end_date):
    purpose = f"Sprint {sprint_num} | Ends {end_date} | Board: https://jira.example.com/board/42"
    await client.conversations_setPurpose(channel=channel_id, purpose=purpose)
```

## 45. Keyboard Shortcuts

### Global Shortcuts

Triggered from anywhere in Slack (the lightning bolt menu or keyboard shortcut):

```python
@app.shortcut("create_incident")
def handle_shortcut(ack, shortcut, client):
    ack()
    client.views_open(
        trigger_id=shortcut["trigger_id"],
        view={
            "type": "modal",
            "callback_id": "incident_form",
            "title": {"type": "plain_text", "text": "New Incident"},
            "blocks": [...]
        }
    )
```

### Message Shortcuts

Triggered from the context menu on a specific message:

```python
@app.shortcut("create_ticket_from_message")
def handle_message_shortcut(ack, shortcut, client):
    ack()
    message_text = shortcut["message"]["text"]
    client.views_open(
        trigger_id=shortcut["trigger_id"],
        view={
            "type": "modal",
            "callback_id": "ticket_from_message",
            "title": {"type": "plain_text", "text": "Create Ticket"},
            "blocks": [
                {
                    "type": "section",
                    "text": {"type": "mrkdwn", "text": f"Creating ticket from:\n> {message_text}"}
                },
                {
                    "type": "input",
                    "block_id": "title_block",
                    "label": {"type": "plain_text", "text": "Ticket Title"},
                    "element": {"type": "plain_text_input", "action_id": "title", "initial_value": message_text[:100]}
                }
            ]
        }
    )
```

## 46. Data Loss Prevention (DLP) and Compliance

### Discovery API (Enterprise Grid)

```bash
# Search for messages containing sensitive data
curl -X POST https://slack.com/api/admin.conversations.search \
  -H "Authorization: Bearer xoxp-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"query":"credit card","sort":"timestamp","sort_dir":"desc"}'
```

### Message Tombstoning

When a message is deleted by DLP, it leaves a tombstone:
```json
{
  "type": "message",
  "subtype": "tombstone",
  "text": "This message was deleted.",
  "hidden": true
}
```

### Audit Logs API (Enterprise Grid)

```bash
curl "https://api.slack.com/audit/v1/logs?action=user_login&oldest=1716000000&limit=100" \
  -H "Authorization: Bearer xoxp-admin-token"
```

Available audit actions: `user_login`, `user_logout`, `file_downloaded`, `file_uploaded`, `channel_created`, `app_installed`, `message_deleted`, `workspace_settings_changed`.

## 47. Tips for Effective Slack Communication

### Message Structure Guidelines

1. Lead with the most important information (inverted pyramid)
2. Use bold for key terms and action items
3. Use code formatting for technical references (commands, file paths, error codes)
4. Use threads for follow-up discussions
5. Use reactions instead of "thanks" or "+1" messages
6. Pin important messages for reference
7. Use scheduled messages for time-sensitive announcements across time zones
8. Include context blocks for metadata (who, when, links)

### Channel Hygiene

1. Archive inactive channels (no messages in 90+ days)
2. Use channel descriptions and topics actively
3. Set posting permissions on announcement channels
4. Use bookmarks for frequently referenced links
5. Create a channel naming convention and enforce it
6. Use default channels wisely (limit to essential ones)

### Notification Management

1. Configure channel-specific notification preferences
2. Use `<!here>` instead of `<!channel>` when possible
3. Schedule messages for recipients' working hours
4. Use threads to reduce notification noise
5. Set keywords for important topics across all channels

## 48. Advanced Integrations

### GitHub Integration Pattern

```python
from flask import Flask, request
import hmac
import hashlib

app = Flask(__name__)

@app.route("/github/webhook", methods=["POST"])
def github_webhook():
    # Verify GitHub signature
    signature = request.headers.get("X-Hub-Signature-256")
    payload = request.get_data()
    expected = "sha256=" + hmac.new(GITHUB_SECRET.encode(), payload, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(signature, expected):
        return "Invalid signature", 401
    
    event = request.headers.get("X-GitHub-Event")
    data = request.json
    
    if event == "pull_request":
        action = data["action"]
        pr = data["pull_request"]
        
        if action == "opened":
            slack_client.chat_postMessage(
                channel="#code-reviews",
                blocks=[
                    {"type": "section", "text": {"type": "mrkdwn", "text": f":git-pull-request: *New PR Opened*\n<{pr['html_url']}|#{pr['number']} - {pr['title']}>\n\n_{pr['body'][:200]}..._"}},
                    {"type": "section", "fields": [
                        {"type": "mrkdwn", "text": f"*Author:*\n{pr['user']['login']}"},
                        {"type": "mrkdwn", "text": f"*Branch:*\n`{pr['head']['ref']}` → `{pr['base']['ref']}`"},
                        {"type": "mrkdwn", "text": f"*Changes:*\n+{pr['additions']} / -{pr['deletions']}"},
                        {"type": "mrkdwn", "text": f"*Files:*\n{pr['changed_files']} files"}
                    ]},
                    {"type": "actions", "elements": [
                        {"type": "button", "text": {"type": "plain_text", "text": "Review"}, "url": pr["html_url"], "action_id": "review_pr"},
                        {"type": "button", "text": {"type": "plain_text", "text": "Approve"}, "style": "primary", "action_id": "approve_pr", "value": str(pr["number"])}
                    ]}
                ]
            )
        elif action == "merged":
            slack_client.chat_postMessage(
                channel="#deployments",
                text=f":merged: PR #{pr['number']} merged: {pr['title']}"
            )
    
    elif event == "push":
        ref = data["ref"]
        if ref == "refs/heads/main":
            commits = data["commits"]
            commit_list = "\n".join([f"• `{c['id'][:7]}` {c['message'].split(chr(10))[0]}" for c in commits[:5]])
            slack_client.chat_postMessage(
                channel="#deployments",
                blocks=[
                    {"type": "section", "text": {"type": "mrkdwn", "text": f":arrow_up: *Push to main*\n{len(commits)} commit(s) by {data['pusher']['name']}\n\n{commit_list}"}}
                ]
            )
    
    return "OK", 200
```

### PagerDuty Integration Pattern

```python
def handle_pagerduty_webhook(payload):
    event = payload["event"]
    incident = event["data"]
    
    if event["event_type"] == "incident.triggered":
        slack_client.chat_postMessage(
            channel="#incidents",
            blocks=[
                {"type": "header", "text": {"type": "plain_text", "text": ":rotating_light: PagerDuty Incident Triggered"}},
                {"type": "section", "fields": [
                    {"type": "mrkdwn", "text": f"*Title:*\n{incident['title']}"},
                    {"type": "mrkdwn", "text": f"*Urgency:*\n{incident['urgency'].upper()}"},
                    {"type": "mrkdwn", "text": f"*Service:*\n{incident['service']['summary']}"},
                    {"type": "mrkdwn", "text": f"*Assigned:*\n{', '.join([a['summary'] for a in incident.get('assignments', [])])}"}
                ]},
                {"type": "actions", "elements": [
                    {"type": "button", "text": {"type": "plain_text", "text": "Acknowledge"}, "style": "primary", "action_id": "pd_ack", "value": incident["id"]},
                    {"type": "button", "text": {"type": "plain_text", "text": "Resolve"}, "style": "danger", "action_id": "pd_resolve", "value": incident["id"]},
                    {"type": "button", "text": {"type": "plain_text", "text": "View in PD"}, "url": incident["html_url"], "action_id": "pd_view"}
                ]}
            ]
        )
    
    elif event["event_type"] == "incident.resolved":
        slack_client.chat_postMessage(
            channel="#incidents",
            text=f":white_check_mark: Incident resolved: {incident['title']}"
        )
```

### Jira Integration Pattern

```python
def format_jira_issue(issue):
    status_emoji = {
        "To Do": ":white_circle:",
        "In Progress": ":large_blue_circle:",
        "In Review": ":mag:",
        "Done": ":white_check_mark:",
        "Blocked": ":no_entry:"
    }
    
    priority_emoji = {
        "Highest": ":fire:",
        "High": ":red_circle:",
        "Medium": ":large_orange_circle:",
        "Low": ":large_green_circle:",
        "Lowest": ":white_circle:"
    }
    
    emoji = status_emoji.get(issue["status"], ":grey_question:")
    p_emoji = priority_emoji.get(issue["priority"], "")
    
    return {
        "blocks": [
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"{emoji} *<{issue['url']}|{issue['key']}>* - {issue['summary']}\n{p_emoji} Priority: {issue['priority']} | Assignee: {issue.get('assignee', 'Unassigned')}"}
            },
            {
                "type": "context",
                "elements": [
                    {"type": "mrkdwn", "text": f"Type: {issue['type']} | Sprint: {issue.get('sprint', 'Backlog')} | Story Points: {issue.get('points', '-')}"}
                ]
            }
        ]
    }
```

## 49. Slack CLI (slack-cli)

The Slack CLI enables local development and deployment of next-gen Slack apps:

```bash
# Install
curl -fsSL https://downloads.slack-edge.com/slack-cli/install.sh | bash

# Login
slack login

# Create a new app
slack create my-app --template https://github.com/slack-samples/deno-starter-template

# Run locally (with hot reload)
slack run

# Deploy to Slack infrastructure
slack deploy

# List triggers
slack trigger list

# Create a trigger
slack trigger create --trigger-def triggers/shortcut.ts

# View activity logs
slack activity --tail

# List installed apps
slack app list

# Delete an app
slack app delete
```

### Trigger Types

| Type | Description | Example |
|------|-------------|---------|
| Link trigger | Clickable URL in channel | Share in channel bookmark |
| Shortcut trigger | Global or message shortcut | Lightning bolt menu |
| Event trigger | Fires on Slack event | reaction_added, message_posted |
| Scheduled trigger | Fires on schedule | Daily at 9 AM |
| Webhook trigger | External HTTP call | CI/CD pipeline callback |

## 50. Testing Slack Apps

### Block Kit Builder

Use https://app.slack.com/block-kit-builder to visually design and test Block Kit layouts before implementing them in code.

### Mocking the Slack API

```python
from unittest.mock import MagicMock, patch

def test_deploy_notification():
    mock_client = MagicMock()
    mock_client.chat_postMessage.return_value = {"ok": True, "ts": "1234567890.123456"}
    
    result = send_deploy_notification(
        client=mock_client,
        channel="C012AB3CD",
        service="api-gateway",
        version="v2.5.0",
        environment="production"
    )
    
    mock_client.chat_postMessage.assert_called_once()
    call_args = mock_client.chat_postMessage.call_args
    assert call_args.kwargs["channel"] == "C012AB3CD"
    assert "api-gateway" in call_args.kwargs["text"]
    assert any("v2.5.0" in str(block) for block in call_args.kwargs.get("blocks", []))
```

### Integration Testing with Slack Sandbox

```python
import os
import pytest
from slack_sdk import WebClient

@pytest.fixture
def slack_client():
    return WebClient(token=os.environ["SLACK_TEST_BOT_TOKEN"])

@pytest.fixture
def test_channel():
    return os.environ["SLACK_TEST_CHANNEL"]

def test_post_and_update_message(slack_client, test_channel):
    # Post
    result = slack_client.chat_postMessage(channel=test_channel, text="Test message")
    assert result["ok"]
    ts = result["ts"]
    
    # Update
    result = slack_client.chat_update(channel=test_channel, ts=ts, text="Updated message")
    assert result["ok"]
    
    # Delete (cleanup)
    slack_client.chat_delete(channel=test_channel, ts=ts)
```

## 51. Performance Optimization

### Batch Operations

When you need to perform many operations, batch them efficiently:

```python
import asyncio
from slack_sdk.web.async_client import AsyncWebClient

async def post_to_multiple_channels(channels, message, blocks):
    client = AsyncWebClient(token=BOT_TOKEN)
    
    # Respect rate limits: 1 message per channel per second
    tasks = []
    for i, channel in enumerate(channels):
        # Stagger requests to avoid rate limits
        await asyncio.sleep(1.1)
        task = client.chat_postMessage(channel=channel, text=message, blocks=blocks)
        tasks.append(task)
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    successes = [r for r in results if not isinstance(r, Exception) and r.get("ok")]
    failures = [r for r in results if isinstance(r, Exception) or not r.get("ok")]
    
    return {"sent": len(successes), "failed": len(failures)}
```

### Caching User and Channel Data

```python
from functools import lru_cache
import time

class SlackCache:
    def __init__(self, client, ttl=300):
        self.client = client
        self.ttl = ttl
        self._user_cache = {}
        self._channel_cache = {}
    
    def get_user(self, user_id):
        cached = self._user_cache.get(user_id)
        if cached and time.time() - cached["fetched_at"] < self.ttl:
            return cached["data"]
        
        result = self.client.users_info(user=user_id)
        if result["ok"]:
            self._user_cache[user_id] = {"data": result["user"], "fetched_at": time.time()}
            return result["user"]
        return None
    
    def get_channel(self, channel_id):
        cached = self._channel_cache.get(channel_id)
        if cached and time.time() - cached["fetched_at"] < self.ttl:
            return cached["data"]
        
        result = self.client.conversations_info(channel=channel_id)
        if result["ok"]:
            self._channel_cache[channel_id] = {"data": result["channel"], "fetched_at": time.time()}
            return result["channel"]
        return None
```

## 52. Slack Connect Best Practices

When working with external organizations via Slack Connect:

1. Use separate channels for each external partner (prefix with `#ext-`)
2. Be aware that external users can see channel history from when they joined
3. File sharing policies may differ between organizations
4. Apps installed in your workspace can see messages from external users
5. Custom emoji from other workspaces may not render
6. User groups cannot span across organizations
7. Workflows triggered in shared channels execute in the workspace that owns the workflow
8. Message retention policies are enforced by each organization independently

## 53. Accessibility and Internationalization

### Screen Reader Compatibility

- Always provide `alt_text` for images (descriptive, not decorative)
- Use the `text` fallback field in messages with blocks
- Avoid using only emoji to convey meaning
- Structure content with clear hierarchy (header → section → context)
- Use plain_text for critical actionable content

### Multi-Language Support

```python
def get_localized_message(user_locale, key):
    messages = {
        "en-US": {"greeting": "Hello!", "deploy_success": "Deployment successful"},
        "pt-BR": {"greeting": "Olá!", "deploy_success": "Deploy realizado com sucesso"},
        "es-ES": {"greeting": "¡Hola!", "deploy_success": "Despliegue exitoso"},
        "fr-FR": {"greeting": "Bonjour!", "deploy_success": "Déploiement réussi"}
    }
    locale_messages = messages.get(user_locale, messages["en-US"])
    return locale_messages.get(key, messages["en-US"][key])

# Get user locale
user_info = client.users_info(user=user_id)
locale = user_info["user"].get("locale", "en-US")
message = get_localized_message(locale, "deploy_success")
```

## 54. Migration from Legacy Attachments to Block Kit

Legacy attachments (the `attachments` field) are deprecated in favor of Block Kit. Here's how to migrate:

### Before (Legacy Attachment)

```json
{
  "attachments": [
    {
      "color": "#36a64f",
      "title": "Deployment Complete",
      "title_link": "https://deploy.example.com",
      "fields": [
        {"title": "Service", "value": "api-gateway", "short": true},
        {"title": "Version", "value": "v2.5.0", "short": true}
      ],
      "footer": "DeployBot",
      "ts": 1716000000
    }
  ]
}
```

### After (Block Kit)

```json
{
  "blocks": [
    {
      "type": "header",
      "text": {"type": "plain_text", "text": "Deployment Complete"}
    },
    {
      "type": "section",
      "fields": [
        {"type": "mrkdwn", "text": "*Service:*\n`api-gateway`"},
        {"type": "mrkdwn", "text": "*Version:*\nv2.5.0"}
      ]
    },
    {
      "type": "context",
      "elements": [
        {"type": "mrkdwn", "text": "DeployBot | <!date^1716000000^{date_short} {time}|May 18, 2025>"}
      ]
    }
  ]
}
```

### Key Differences

| Feature | Legacy Attachments | Block Kit |
|---------|-------------------|-----------|
| Color bar | `color` field | Not available (use emoji/text) |
| Author | `author_name`, `author_icon` | Context block with image |
| Title link | `title_link` | Section with mrkdwn link |
| Fields | `fields` array | Section `fields` array |
| Footer | `footer`, `footer_icon` | Context block |
| Timestamp | `ts` field | Date formatting syntax |
| Actions | `actions` in attachment | Actions block |
| Interactivity | Limited | Full (buttons, selects, modals) |

## 55. Webhook Security

### Verifying Incoming Requests

```python
import hashlib
import hmac
import time

def verify_slack_signature(signing_secret, request_body, timestamp, signature):
    # Reject requests older than 5 minutes
    if abs(time.time() - int(timestamp)) > 300:
        return False
    
    # Compute expected signature
    sig_basestring = f"v0:{timestamp}:{request_body}"
    computed = "v0=" + hmac.new(
        signing_secret.encode(),
        sig_basestring.encode(),
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(computed, signature)

# In your request handler
@app.before_request
def verify_request():
    timestamp = request.headers.get("X-Slack-Request-Timestamp", "")
    signature = request.headers.get("X-Slack-Signature", "")
    body = request.get_data(as_text=True)
    
    if not verify_slack_signature(SIGNING_SECRET, body, timestamp, signature):
        abort(401)
```

### Webhook URL Rotation

If a webhook URL is compromised:
1. Regenerate the webhook URL in app settings
2. Update all systems using the old URL
3. Monitor the old URL for unauthorized usage
4. Consider using signed webhooks with verification

## 56. Complete Emoji Skin Tone Reference

Slack supports skin tone modifiers for people emoji:

```
:thumbsup::skin-tone-2:  → 👍🏻 (light)
:thumbsup::skin-tone-3:  → 👍🏼 (medium-light)
:thumbsup::skin-tone-4:  → 👍🏽 (medium)
:thumbsup::skin-tone-5:  → 👍🏾 (medium-dark)
:thumbsup::skin-tone-6:  → 👍🏿 (dark)
```

### Reacji (Reaction-Based Automation)

Use reactions as triggers for automated workflows:

```python
@app.event("reaction_added")
def handle_reaction(event, client):
    reaction = event["reaction"]
    channel = event["item"]["channel"]
    ts = event["item"]["ts"]
    
    if reaction == "ticket":
        # Fetch the message
        result = client.conversations_history(channel=channel, latest=ts, limit=1, inclusive=True)
        message = result["messages"][0]
        
        # Create a Jira ticket from the message
        ticket = create_jira_ticket(
            title=message["text"][:100],
            description=message["text"],
            reporter=event["user"]
        )
        
        # Reply in thread
        client.chat_postMessage(
            channel=channel,
            thread_ts=ts,
            text=f":ticket: Ticket created: <{ticket['url']}|{ticket['key']}>"
        )
    
    elif reaction == "eyes":
        # Acknowledge that someone is looking into it
        client.chat_postMessage(
            channel=channel,
            thread_ts=ts,
            text=f"<@{event['user']}> is looking into this :eyes:"
        )
    
    elif reaction == "pushpin":
        # Auto-pin the message
        try:
            client.pins_add(channel=channel, timestamp=ts)
        except Exception:
            pass  # Already pinned or no permission
```

## 57. Appendix: Quick Reference Card

### mrkdwn Cheat Sheet

```
*bold*  _italic_  ~strike~  `code`  ```block```
> quote  >>> quote all
<url|text>  <@user>  <#channel>  <!here>  <!channel>
<!date^ts^{token}|fallback>
:emoji:  :emoji::skin-tone-2:
```

### Block Types Quick Reference

```
header     → Large bold text (plain_text, 150 chars)
section    → Text + optional accessory + fields
divider    → Horizontal line
image      → Standalone image
actions    → Interactive elements (max 25)
context    → Small metadata (max 10 elements)
input      → Form input (modals/home)
rich_text  → Structured formatted text
video      → Embedded video
file       → File reference
```

### Token Prefixes

```
xoxb-  → Bot token
xoxp-  → User token
xapp-  → App-level token
xoxe-  → Enterprise token
```

### HTTP Response Codes

```
200 → Success (check "ok" field)
429 → Rate limited (check Retry-After header)
500 → Server error (retry with backoff)
```

## 58. Slack Notifications Best Practices for Bots

### Notification Priority Matrix

| Priority | Channel | Format | Mention | Example |
|----------|---------|--------|---------|---------|
| P1 Critical | #incidents + DM | Full blocks + emoji | `<!channel>` | Production down |
| P2 High | #alerts | Blocks with actions | `<!here>` | Error rate spike |
| P3 Medium | #monitoring | Simple block | None | Deployment complete |
| P4 Low | #bot-logs | Plain text | None | Scheduled job ran |
| Info | Thread reply | Context block | None | Status update |

### Notification Throttling

```python
import time
from collections import defaultdict

class NotificationThrottler:
    def __init__(self, min_interval_seconds=60):
        self.min_interval = min_interval_seconds
        self.last_sent = defaultdict(float)
    
    def should_send(self, key):
        now = time.time()
        if now - self.last_sent[key] >= self.min_interval:
            self.last_sent[key] = now
            return True
        return False
    
    def send_or_batch(self, key, message):
        if self.should_send(key):
            return self._send_immediately(key, message)
        else:
            return self._add_to_batch(key, message)

throttler = NotificationThrottler(min_interval_seconds=300)

def alert_handler(alert):
    key = f"{alert['service']}:{alert['type']}"
    if throttler.should_send(key):
        post_alert_to_slack(alert)
    else:
        # Batch similar alerts and send summary later
        batch_alert(key, alert)
```

### Smart Notification Routing

```python
def route_notification(event_type, severity, service):
    routing = {
        "deployment": {
            "channel": "#deployments",
            "mention": None,
            "thread": False
        },
        "incident": {
            "channel": "#incidents",
            "mention": "<!channel>" if severity <= 2 else "<!here>",
            "thread": False
        },
        "alert": {
            "channel": f"#alerts-{service}",
            "mention": "<!here>" if severity == 1 else None,
            "thread": True
        },
        "ci_failure": {
            "channel": "#ci-cd",
            "mention": None,
            "thread": True
        }
    }
    return routing.get(event_type, {"channel": "#bot-logs", "mention": None, "thread": False})
```

## 59. Enterprise Grid Administration

### Organization-Level Operations

```bash
# List all workspaces in the org
curl "https://slack.com/api/admin.teams.list?limit=100" \
  -H "Authorization: Bearer xoxp-org-admin-token"

# Create a new workspace
curl -X POST https://slack.com/api/admin.teams.create \
  -H "Authorization: Bearer xoxp-org-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"team_domain":"new-team","team_name":"New Team","team_description":"A new workspace"}'

# Set organization-wide app policies
curl -X POST https://slack.com/api/admin.apps.restrict \
  -H "Authorization: Bearer xoxp-org-admin-token" \
  -H "Content-Type: application/json" \
  -d '{"app_id":"A012AB3CD","team_id":"T012AB3CD"}'
```

### Information Barriers

Enterprise Grid supports information barriers to prevent communication between specific groups:

```bash
# Create an information barrier
curl -X POST https://slack.com/api/admin.barriers.create \
  -H "Authorization: Bearer xoxp-org-admin-token" \
  -H "Content-Type: application/json" \
  -d '{
    "barriered_from_usergroup_ids": ["S001", "S002"],
    "primary_usergroup_id": "S003",
    "restricted_subjects": ["im", "mpim", "call"]
  }'
```

## 60. Monitoring Your Slack App

### Health Check Endpoint

```python
from flask import Flask, jsonify
import time

app = Flask(__name__)
last_event_time = time.time()

@app.route("/health")
def health():
    # Check if we've received events recently
    seconds_since_last_event = time.time() - last_event_time
    
    status = {
        "status": "healthy" if seconds_since_last_event < 300 else "degraded",
        "last_event_seconds_ago": int(seconds_since_last_event),
        "uptime_seconds": int(time.time() - app_start_time),
        "version": APP_VERSION,
        "slack_api_reachable": check_slack_api()
    }
    
    code = 200 if status["status"] == "healthy" else 503
    return jsonify(status), code

def check_slack_api():
    try:
        result = slack_client.auth_test()
        return result["ok"]
    except Exception:
        return False
```

### Metrics to Track

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `slack_api_calls_total` | Total API calls made | Rate > 80% of limit |
| `slack_api_errors_total` | Failed API calls | Error rate > 5% |
| `slack_api_latency_ms` | API response time | p99 > 5000ms |
| `slack_events_received_total` | Events received | Drop to 0 for 5 min |
| `slack_events_processed_total` | Events processed | Lag > 100 events |
| `slack_rate_limits_hit` | 429 responses received | Any occurrence |
| `slack_message_send_duration_ms` | Time to send message | p95 > 3000ms |

### Logging Best Practices

```python
import logging
import json

logger = logging.getLogger("slack_app")

def log_api_call(method, channel=None, user=None, success=True, error=None, duration_ms=None):
    log_data = {
        "type": "slack_api_call",
        "method": method,
        "channel": channel,
        "user": user,
        "success": success,
        "error": error,
        "duration_ms": duration_ms,
        "timestamp": time.time()
    }
    
    if success:
        logger.info(json.dumps(log_data))
    else:
        logger.error(json.dumps(log_data))
```

This concludes the comprehensive Slack Master Specialist reference covering all aspects of the Slack platform from basic message formatting through enterprise administration, app development, and production operations.
