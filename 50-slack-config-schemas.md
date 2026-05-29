# Slack Master Specialist — Configuration Schemas Reference

## 1. App Manifest (manifest.yaml)

The complete schema for defining a Slack app declaratively:

```yaml
_metadata:
  major_version: 2
  minor_version: 0

display_information:
  name: "App Name"                    # Required, max 35 chars
  description: "Short description"    # Max 140 chars
  long_description: "Detailed..."     # Max 4000 chars
  background_color: "#2c2d30"         # Hex color for app icon background
  app_home:
    home_tab_enabled: true
    messages_tab_enabled: true
    messages_tab_read_only_enabled: false

features:
  bot_user:
    display_name: "BotName"           # Required, max 80 chars
    always_online: true               # Show bot as always online
  
  slash_commands:
    - command: /deploy
      url: https://app.example.com/slack/commands
      description: "Trigger a deployment"    # Max 2000 chars
      usage_hint: "[env] [service] [version]"  # Max 1000 chars
      should_escape: false
    - command: /status
      url: https://app.example.com/slack/commands
      description: "Check system status"
      usage_hint: "[service]"
      should_escape: false
  
  shortcuts:
    - name: "Create Incident"
      type: global                    # global or message
      callback_id: create_incident
      description: "Create a new incident report"
    - name: "Create Ticket"
      type: message
      callback_id: create_ticket_from_msg
      description: "Create ticket from this message"
  
  unfurl_domains:
    - "jira.example.com"
    - "confluence.example.com"
    - "github.com"
  
  workflow_steps:
    - name: "Create Ticket"
      callback_id: create_ticket_step

oauth_config:
  redirect_urls:
    - https://app.example.com/slack/oauth/callback
  scopes:
    user:
      - search:read
      - users.profile:write
    bot:
      - app_mentions:read
      - channels:history
      - channels:read
      - chat:write
      - chat:write.public
      - commands
      - files:read
      - files:write
      - groups:history
      - groups:read
      - im:history
      - im:read
      - im:write
      - mpim:history
      - reactions:read
      - reactions:write
      - users:read
      - users:read.email

settings:
  event_subscriptions:
    request_url: https://app.example.com/slack/events
    bot_events:
      - app_home_opened
      - app_mention
      - message.channels
      - message.groups
      - message.im
      - member_joined_channel
      - reaction_added
      - link_shared
    user_events:
      - message.channels
  
  interactivity:
    is_enabled: true
    request_url: https://app.example.com/slack/interactions
    message_menu_options_url: https://app.example.com/slack/options
  
  org_deploy_enabled: false
  socket_mode_enabled: false
  token_rotation_enabled: false
  
  allowed_ip_address_ranges:
    - "203.0.113.0/24"
    - "198.51.100.0/24"
```

## 2. Block Kit Schemas

### All Block Types

```json
// Header Block
{"type": "header", "text": {"type": "plain_text", "text": "Title", "emoji": true}, "block_id": "optional_id"}

// Section Block
{
  "type": "section",
  "text": {"type": "mrkdwn", "text": "*Bold* and _italic_"},
  "block_id": "section_1",
  "fields": [
    {"type": "mrkdwn", "text": "*Field 1*\nValue"},
    {"type": "mrkdwn", "text": "*Field 2*\nValue"}
  ],
  "accessory": {/* element */}
}

// Divider Block
{"type": "divider", "block_id": "divider_1"}

// Image Block
{
  "type": "image",
  "image_url": "https://example.com/image.png",
  "alt_text": "Description of image",
  "title": {"type": "plain_text", "text": "Image Title"},
  "block_id": "image_1"
}

// Actions Block
{
  "type": "actions",
  "block_id": "actions_1",
  "elements": [/* up to 25 interactive elements */]
}

// Context Block
{
  "type": "context",
  "block_id": "context_1",
  "elements": [/* up to 10 image or text elements */]
}

// Input Block (modals and App Home only)
{
  "type": "input",
  "block_id": "input_1",
  "label": {"type": "plain_text", "text": "Label"},
  "element": {/* input element */},
  "hint": {"type": "plain_text", "text": "Helper text"},
  "optional": false,
  "dispatch_action": false
}

// Video Block
{
  "type": "video",
  "title": {"type": "plain_text", "text": "Video Title"},
  "video_url": "https://www.youtube.com/embed/VIDEO_ID",
  "thumbnail_url": "https://example.com/thumb.png",
  "alt_text": "Video description",
  "author_name": "Author",
  "provider_name": "YouTube",
  "provider_icon_url": "https://example.com/icon.png"
}

// Rich Text Block
{
  "type": "rich_text",
  "elements": [
    {"type": "rich_text_section", "elements": [{"type": "text", "text": "Hello", "style": {"bold": true}}]},
    {"type": "rich_text_list", "style": "bullet", "elements": [
      {"type": "rich_text_section", "elements": [{"type": "text", "text": "Item 1"}]}
    ]},
    {"type": "rich_text_preformatted", "elements": [{"type": "text", "text": "code here"}]},
    {"type": "rich_text_quote", "elements": [{"type": "text", "text": "quoted text"}]}
  ]
}
```

### Interactive Elements

```json
// Button
{
  "type": "button",
  "text": {"type": "plain_text", "text": "Click Me"},
  "action_id": "button_click",
  "value": "button_value",
  "style": "primary",  // "primary" (green), "danger" (red), or omit (default)
  "url": "https://example.com",  // Opens URL instead of sending action
  "confirm": {
    "title": {"type": "plain_text", "text": "Are you sure?"},
    "text": {"type": "mrkdwn", "text": "This action cannot be undone."},
    "confirm": {"type": "plain_text", "text": "Yes"},
    "deny": {"type": "plain_text", "text": "Cancel"},
    "style": "danger"
  }
}

// Static Select
{
  "type": "static_select",
  "action_id": "select_action",
  "placeholder": {"type": "plain_text", "text": "Choose an option"},
  "options": [
    {"text": {"type": "plain_text", "text": "Option 1"}, "value": "opt1"},
    {"text": {"type": "plain_text", "text": "Option 2"}, "value": "opt2"}
  ],
  "option_groups": [
    {"label": {"type": "plain_text", "text": "Group 1"}, "options": [...]}
  ],
  "initial_option": {"text": {"type": "plain_text", "text": "Option 1"}, "value": "opt1"}
}

// External Select (dynamic options)
{
  "type": "external_select",
  "action_id": "external_select",
  "placeholder": {"type": "plain_text", "text": "Search..."},
  "min_query_length": 3
}

// Multi-Static Select
{
  "type": "multi_static_select",
  "action_id": "multi_select",
  "placeholder": {"type": "plain_text", "text": "Select options"},
  "options": [...],
  "max_selected_items": 5
}

// Users Select
{"type": "users_select", "action_id": "user_pick", "placeholder": {"type": "plain_text", "text": "Pick a user"}}

// Conversations Select
{"type": "conversations_select", "action_id": "channel_pick", "placeholder": {"type": "plain_text", "text": "Pick a channel"}, "filter": {"include": ["public", "private"], "exclude_bot_users": true}}

// Date Picker
{
  "type": "datepicker",
  "action_id": "date_pick",
  "placeholder": {"type": "plain_text", "text": "Select a date"},
  "initial_date": "2025-06-01"
}

// Time Picker
{
  "type": "timepicker",
  "action_id": "time_pick",
  "placeholder": {"type": "plain_text", "text": "Select time"},
  "initial_time": "14:30"
}

// Date-Time Picker
{
  "type": "datetimepicker",
  "action_id": "datetime_pick",
  "initial_date_time": 1716100000
}

// Overflow Menu
{
  "type": "overflow",
  "action_id": "overflow_menu",
  "options": [
    {"text": {"type": "plain_text", "text": "Edit"}, "value": "edit"},
    {"text": {"type": "plain_text", "text": "Delete"}, "value": "delete"}
  ]
}

// Radio Buttons
{
  "type": "radio_buttons",
  "action_id": "radio_pick",
  "options": [
    {"text": {"type": "mrkdwn", "text": "*Option A*\nDescription"}, "value": "a"},
    {"text": {"type": "mrkdwn", "text": "*Option B*\nDescription"}, "value": "b"}
  ]
}

// Checkboxes
{
  "type": "checkboxes",
  "action_id": "checkbox_pick",
  "options": [
    {"text": {"type": "mrkdwn", "text": "Option 1"}, "value": "1"},
    {"text": {"type": "mrkdwn", "text": "Option 2"}, "value": "2"}
  ],
  "initial_options": [{"text": {"type": "mrkdwn", "text": "Option 1"}, "value": "1"}]
}

// Plain Text Input
{
  "type": "plain_text_input",
  "action_id": "text_input",
  "placeholder": {"type": "plain_text", "text": "Enter text"},
  "initial_value": "Default",
  "multiline": false,
  "min_length": 1,
  "max_length": 500,
  "dispatch_action_config": {"trigger_actions_on": ["on_enter_pressed"]}
}

// URL Input
{"type": "url_text_input", "action_id": "url_input", "placeholder": {"type": "plain_text", "text": "https://..."}}

// Email Input
{"type": "email_text_input", "action_id": "email_input", "placeholder": {"type": "plain_text", "text": "user@example.com"}}

// Number Input
{"type": "number_input", "action_id": "number_input", "is_decimal_allowed": true, "min_value": "0", "max_value": "100"}
```

## 3. Modal View Schema

```json
{
  "type": "modal",
  "callback_id": "modal_callback",
  "title": {"type": "plain_text", "text": "Modal Title", "emoji": true},
  "submit": {"type": "plain_text", "text": "Submit"},
  "close": {"type": "plain_text", "text": "Cancel"},
  "private_metadata": "{\"key\":\"value\"}",
  "clear_on_close": false,
  "notify_on_close": false,
  "external_id": "unique_external_id",
  "blocks": [
    // Input blocks for forms
    // Section blocks for display
    // Divider blocks for separation
  ]
}
```

## 4. App Home View Schema

```json
{
  "type": "home",
  "blocks": [
    // Any block type except input
    // Header, section, divider, image, actions, context, rich_text, video
  ],
  "private_metadata": "",
  "external_id": ""
}
```

## 5. Message Payload Schema

```json
{
  "channel": "C012AB3CD",
  "text": "Fallback text (required with blocks)",
  "blocks": [...],
  "thread_ts": "1234567890.123456",
  "reply_broadcast": false,
  "unfurl_links": true,
  "unfurl_media": true,
  "mrkdwn": true,
  "metadata": {
    "event_type": "custom_event",
    "event_payload": {"key": "value"}
  }
}
```

## 6. Bolt Configuration (Python)

```python
from slack_bolt import App

app = App(
    token=os.environ["SLACK_BOT_TOKEN"],
    signing_secret=os.environ["SLACK_SIGNING_SECRET"],
    # OR for Socket Mode:
    # token=os.environ["SLACK_BOT_TOKEN"],
    # For OAuth:
    # client_id=os.environ["SLACK_CLIENT_ID"],
    # client_secret=os.environ["SLACK_CLIENT_SECRET"],
    # scopes=["chat:write", "commands"],
    # installation_store=FileInstallationStore(),
    # oauth_state_store=FileOAuthStateStore(expiration_seconds=600),
)

# Middleware configuration
app.use(lambda next, event, logger: (logger.info(f"Event: {event}"), next()))

# Error handler
@app.error
def handle_error(error, body, logger):
    logger.exception(f"Error: {error}")
    logger.info(f"Request body: {body}")
```

## 7. Bolt Configuration (JavaScript)

```javascript
const { App, LogLevel } = require('@slack/bolt');

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
  logLevel: LogLevel.DEBUG,
  
  // Custom receiver (for Express integration)
  // receiver: new ExpressReceiver({ signingSecret, endpoints: '/slack/events' }),
  
  // OAuth configuration
  // clientId: process.env.SLACK_CLIENT_ID,
  // clientSecret: process.env.SLACK_CLIENT_SECRET,
  // stateSecret: 'my-state-secret',
  // scopes: ['chat:write', 'commands'],
  // installationStore: new FileInstallationStore(),
});

// Global middleware
app.use(async ({ next, body, logger }) => {
  logger.info(`Incoming: ${body.type}`);
  await next();
});

// Error handler
app.error(async (error) => {
  console.error(`Error: ${error.message}`, error.original || error);
});
```

## 8. Next-Gen Platform Configuration (Deno)

### manifest.ts

```typescript
import { Manifest } from "deno-slack-sdk/mod.ts";
import { CreateTicketWorkflow } from "./workflows/create_ticket.ts";

export default Manifest({
  name: "DeployBot",
  description: "Deployment automation",
  icon: "assets/icon.png",
  workflows: [CreateTicketWorkflow],
  outgoingDomains: ["api.example.com"],
  datastores: [],
  botScopes: ["commands", "chat:write", "channels:read"],
});
```

### slack.json

```json
{
  "hooks": {
    "get-hooks": "deno run -q --allow-read --allow-net https://deno.land/x/deno_slack_hooks/mod.ts"
  }
}
```

## 9. Webhook Payload Schemas

### Incoming Webhook

```json
{
  "text": "Fallback text",
  "blocks": [...],
  "response_type": "in_channel",
  "replace_original": false,
  "delete_original": false,
  "unfurl_links": false,
  "unfurl_media": false
}
```

### Interaction Payload (block_actions)

```json
{
  "type": "block_actions",
  "user": {"id": "U012AB3CD", "username": "user", "name": "User Name", "team_id": "T012AB3CD"},
  "api_app_id": "A012AB3CD",
  "token": "verification_token",
  "trigger_id": "12345.98765.abcdef",
  "team": {"id": "T012AB3CD", "domain": "team"},
  "enterprise": null,
  "is_enterprise_install": false,
  "channel": {"id": "C012AB3CD", "name": "general"},
  "message": {"type": "message", "ts": "1234567890.123456", "text": "Original"},
  "response_url": "https://hooks.slack.com/actions/T012/B012/xxxx",
  "actions": [
    {
      "type": "button",
      "action_id": "action_id",
      "block_id": "block_id",
      "text": {"type": "plain_text", "text": "Button Text"},
      "value": "button_value",
      "action_ts": "1234567890.123456"
    }
  ]
}
```

### View Submission Payload

```json
{
  "type": "view_submission",
  "team": {"id": "T012AB3CD"},
  "user": {"id": "U012AB3CD"},
  "view": {
    "id": "V012AB3CD",
    "type": "modal",
    "callback_id": "modal_id",
    "title": {"type": "plain_text", "text": "Title"},
    "state": {
      "values": {
        "block_id": {
          "action_id": {
            "type": "plain_text_input",
            "value": "user entered text"
          }
        }
      }
    },
    "private_metadata": "{\"key\":\"value\"}"
  }
}
```

## 10. Date Formatting Tokens

```
{date_num}         → 2025-05-29
{date}             → May 29, 2025
{date_short}       → May 29, 2025
{date_long}        → Thursday, May 29, 2025
{date_pretty}      → May 29, 2025 (or "yesterday", "today", "tomorrow")
{date_short_pretty} → May 29 (or "yesterday")
{date_long_pretty} → Thursday, May 29, 2025 (or "yesterday")
{time}             → 2:30 PM
{time_secs}        → 2:30:45 PM

// Usage in mrkdwn:
<!date^1716988200^{date_short} at {time}|May 29, 2025 at 2:30 PM>
```

This configuration schemas reference provides the complete structure for all Slack app configurations, Block Kit elements, and payload formats needed for building any Slack integration.
