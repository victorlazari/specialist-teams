# Slack Master Specialist — Advanced Patterns

## 1. Advanced Block Kit Composition

### Dynamic Block Generation

Building blocks programmatically based on data allows for complex, data-driven UIs:

```python
def build_deployment_dashboard(deployments):
    blocks = [
        {"type": "header", "text": {"type": "plain_text", "text": ":rocket: Deployment Dashboard"}},
        {"type": "divider"}
    ]
    
    for deploy in deployments:
        status_emoji = {
            "running": ":large_blue_circle:",
            "success": ":white_check_mark:",
            "failed": ":x:",
            "pending": ":hourglass:"
        }.get(deploy["status"], ":grey_question:")
        
        blocks.append({
            "type": "section",
            "text": {"type": "mrkdwn", "text": f"{status_emoji} *{deploy['service']}* `{deploy['version']}`\nEnvironment: `{deploy['env']}` | Duration: {deploy['duration']}s"},
            "accessory": {
                "type": "overflow",
                "action_id": f"deploy_actions_{deploy['id']}",
                "options": [
                    {"text": {"type": "plain_text", "text": ":arrow_right: View Logs"}, "value": f"logs_{deploy['id']}"},
                    {"text": {"type": "plain_text", "text": ":rewind: Rollback"}, "value": f"rollback_{deploy['id']}"},
                    {"text": {"type": "plain_text", "text": ":mag: Details"}, "value": f"details_{deploy['id']}"}
                ]
            }
        })
    
    # Add summary context
    total = len(deployments)
    success = sum(1 for d in deployments if d["status"] == "success")
    failed = sum(1 for d in deployments if d["status"] == "failed")
    
    blocks.append({"type": "divider"})
    blocks.append({
        "type": "context",
        "elements": [{"type": "mrkdwn", "text": f"Total: {total} | :white_check_mark: {success} | :x: {failed} | Last updated: <!date^{int(time.time())}^{{date_short}} {{time}}|now>"}]
    })
    
    return blocks
```

### Multi-Step Modal Workflows

```python
# Step 1: Initial form
@app.command("/onboard")
def start_onboard(ack, command, client):
    ack()
    client.views_open(
        trigger_id=command["trigger_id"],
        view={
            "type": "modal",
            "callback_id": "onboard_step1",
            "title": {"type": "plain_text", "text": "New Team Member"},
            "submit": {"type": "plain_text", "text": "Next"},
            "blocks": [
                {"type": "input", "block_id": "name", "label": {"type": "plain_text", "text": "Full Name"}, "element": {"type": "plain_text_input", "action_id": "name_input"}},
                {"type": "input", "block_id": "role", "label": {"type": "plain_text", "text": "Role"}, "element": {"type": "static_select", "action_id": "role_select", "options": [
                    {"text": {"type": "plain_text", "text": "Engineer"}, "value": "engineer"},
                    {"text": {"type": "plain_text", "text": "Designer"}, "value": "designer"},
                    {"text": {"type": "plain_text", "text": "Product Manager"}, "value": "pm"}
                ]}},
                {"type": "input", "block_id": "team", "label": {"type": "plain_text", "text": "Team"}, "element": {"type": "external_select", "action_id": "team_select", "min_query_length": 0}}
            ]
        }
    )

# Step 2: Additional details based on role
@app.view("onboard_step1")
def handle_step1(ack, body, client, view):
    values = view["state"]["values"]
    name = values["name"]["name_input"]["value"]
    role = values["role"]["role_select"]["selected_option"]["value"]
    
    # Build step 2 based on role
    role_blocks = []
    if role == "engineer":
        role_blocks = [
            {"type": "input", "block_id": "github", "label": {"type": "plain_text", "text": "GitHub Username"}, "element": {"type": "plain_text_input", "action_id": "github_input"}},
            {"type": "input", "block_id": "languages", "label": {"type": "plain_text", "text": "Languages"}, "element": {"type": "multi_static_select", "action_id": "lang_select", "options": [
                {"text": {"type": "plain_text", "text": "Go"}, "value": "go"},
                {"text": {"type": "plain_text", "text": "Python"}, "value": "python"},
                {"text": {"type": "plain_text", "text": "TypeScript"}, "value": "typescript"},
                {"text": {"type": "plain_text", "text": "Rust"}, "value": "rust"}
            ]}}
        ]
    
    ack(response_action="update", view={
        "type": "modal",
        "callback_id": "onboard_step2",
        "private_metadata": json.dumps({"name": name, "role": role}),
        "title": {"type": "plain_text", "text": "Details"},
        "submit": {"type": "plain_text", "text": "Complete"},
        "blocks": [
            {"type": "section", "text": {"type": "mrkdwn", "text": f"Setting up *{name}* as *{role}*"}},
            {"type": "divider"}
        ] + role_blocks
    })
```

### Conditional Block Rendering

```python
def build_alert_message(alert):
    blocks = [
        {"type": "section", "text": {"type": "mrkdwn", "text": f":warning: *Alert: {alert['title']}*\n{alert['description']}"}}
    ]
    
    # Add graph image if available
    if alert.get("graph_url"):
        blocks.append({"type": "image", "image_url": alert["graph_url"], "alt_text": f"Graph for {alert['title']}"})
    
    # Add runbook link if exists
    if alert.get("runbook_url"):
        blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": f":book: <{alert['runbook_url']}|View Runbook>"}})
    
    # Add action buttons based on alert type
    actions = []
    if alert["type"] == "infrastructure":
        actions.extend([
            {"type": "button", "text": {"type": "plain_text", "text": "Scale Up"}, "action_id": "scale_up", "value": alert["resource_id"]},
            {"type": "button", "text": {"type": "plain_text", "text": "Restart"}, "action_id": "restart", "style": "danger", "value": alert["resource_id"], "confirm": {
                "title": {"type": "plain_text", "text": "Confirm Restart"},
                "text": {"type": "mrkdwn", "text": f"Are you sure you want to restart `{alert['resource_id']}`?"},
                "confirm": {"type": "plain_text", "text": "Restart"},
                "deny": {"type": "plain_text", "text": "Cancel"}
            }}
        ])
    
    if actions:
        blocks.append({"type": "actions", "elements": actions})
    
    return blocks
```

## 2. App Home Tab

The App Home provides a personalized dashboard for each user:

```python
@app.event("app_home_opened")
def update_home_tab(client, event):
    user_id = event["user"]
    
    # Fetch user-specific data
    user_tasks = get_user_tasks(user_id)
    user_prs = get_user_open_prs(user_id)
    oncall_status = get_oncall_status(user_id)
    
    blocks = [
        {"type": "header", "text": {"type": "plain_text", "text": ":house: Your Dashboard"}},
        {"type": "section", "text": {"type": "mrkdwn", "text": f"Welcome back, <@{user_id}>! Here's your overview."}}
    ]
    
    # On-call status
    if oncall_status["is_oncall"]:
        blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": f":pager: *You are currently on-call*\nShift ends: <!date^{oncall_status['end_ts']}^{{date_long}} {{time}}|soon>"}})
    
    # Open PRs
    if user_prs:
        blocks.append({"type": "divider"})
        blocks.append({"type": "header", "text": {"type": "plain_text", "text": ":git-pull-request: Your Open PRs"}})
        for pr in user_prs[:5]:
            blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": f"<{pr['url']}|#{pr['number']} {pr['title']}> — {pr['reviews']} reviews, {pr['age']} days old"}})
    
    # Tasks
    if user_tasks:
        blocks.append({"type": "divider"})
        blocks.append({"type": "header", "text": {"type": "plain_text", "text": ":clipboard: Your Tasks"}})
        for task in user_tasks[:5]:
            checkbox = ":white_check_mark:" if task["done"] else ":white_large_square:"
            blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": f"{checkbox} {task['title']}"}})
    
    # Quick actions
    blocks.append({"type": "divider"})
    blocks.append({"type": "actions", "elements": [
        {"type": "button", "text": {"type": "plain_text", "text": ":heavy_plus_sign: New Task"}, "action_id": "new_task"},
        {"type": "button", "text": {"type": "plain_text", "text": ":calendar: Schedule Meeting"}, "action_id": "schedule_meeting"},
        {"type": "button", "text": {"type": "plain_text", "text": ":mag: Search Docs"}, "action_id": "search_docs"}
    ]})
    
    client.views_publish(user_id=user_id, view={"type": "home", "blocks": blocks})
```

## 3. Advanced Incoming Webhooks

### Webhook with Full Block Kit

```python
import requests

WEBHOOK_URL = "https://hooks.slack.com/services/YOUR_T/YOUR_B/YOUR_TOKEN"

def send_build_notification(build):
    status_map = {
        "success": (":white_check_mark:", "#36a64f"),
        "failure": (":x:", "#dc3545"),
        "running": (":arrows_counterclockwise:", "#ffc107")
    }
    emoji, color = status_map.get(build["status"], (":grey_question:", "#6c757d"))
    
    payload = {
        "blocks": [
            {"type": "header", "text": {"type": "plain_text", "text": f"{emoji} Build #{build['number']} — {build['status'].title()}"}},
            {"type": "section", "fields": [
                {"type": "mrkdwn", "text": f"*Repository:*\n`{build['repo']}`"},
                {"type": "mrkdwn", "text": f"*Branch:*\n`{build['branch']}`"},
                {"type": "mrkdwn", "text": f"*Commit:*\n`{build['commit'][:7]}`"},
                {"type": "mrkdwn", "text": f"*Duration:*\n{build['duration']}s"},
                {"type": "mrkdwn", "text": f"*Triggered by:*\n{build['author']}"},
                {"type": "mrkdwn", "text": f"*Tests:*\n{build['tests_passed']}/{build['tests_total']} passed"}
            ]},
            {"type": "context", "elements": [
                {"type": "mrkdwn", "text": f"Pipeline: {build['pipeline']} | Stage: {build['stage']}"}
            ]}
        ]
    }
    
    if build["status"] == "failure" and build.get("error_log"):
        payload["blocks"].append({
            "type": "section",
            "text": {"type": "mrkdwn", "text": f"*Error:*\n```{build['error_log'][:2000]}```"}
        })
    
    response = requests.post(WEBHOOK_URL, json=payload)
    return response.status_code == 200
```

## 4. Rate Limiting Strategy

### Tier-Based Rate Limits

| Tier | Rate | Methods |
|------|------|---------|
| Tier 1 | 1 req/min | `admin.*`, `migration.*` |
| Tier 2 | 20 req/min | `channels.list`, `users.list` |
| Tier 3 | 50 req/min | `chat.postMessage`, `reactions.add` |
| Tier 4 | 100 req/min | `auth.test`, `conversations.info` |
| Special | 1 req/sec/channel | `chat.postMessage` per channel |

### Token Bucket Implementation

```python
import time
import threading

class SlackRateLimiter:
    def __init__(self):
        self.locks = {}
        self.tokens = {}
        self.last_refill = {}
        self.tier_limits = {
            1: (1, 60),    # 1 token, refill every 60s
            2: (20, 60),   # 20 tokens, refill every 60s
            3: (50, 60),   # 50 tokens, refill every 60s
            4: (100, 60),  # 100 tokens, refill every 60s
        }
    
    def _get_tier(self, method):
        tier_map = {
            "chat.postMessage": 3,
            "chat.update": 3,
            "reactions.add": 3,
            "conversations.list": 2,
            "users.list": 2,
            "conversations.info": 4,
            "auth.test": 4,
        }
        return tier_map.get(method, 3)
    
    def acquire(self, method):
        tier = self._get_tier(method)
        max_tokens, refill_interval = self.tier_limits[tier]
        
        if method not in self.locks:
            self.locks[method] = threading.Lock()
            self.tokens[method] = max_tokens
            self.last_refill[method] = time.time()
        
        with self.locks[method]:
            # Refill tokens
            now = time.time()
            elapsed = now - self.last_refill[method]
            refill_amount = int(elapsed / refill_interval * max_tokens)
            if refill_amount > 0:
                self.tokens[method] = min(max_tokens, self.tokens[method] + refill_amount)
                self.last_refill[method] = now
            
            if self.tokens[method] > 0:
                self.tokens[method] -= 1
                return True
            else:
                # Calculate wait time
                wait = refill_interval / max_tokens
                time.sleep(wait)
                return True

rate_limiter = SlackRateLimiter()
```

## 5. Slash Command Response Patterns

### Ephemeral vs In-Channel Responses

```python
@app.command("/status")
def handle_status(ack, command, respond):
    ack()
    
    # Ephemeral (only visible to the user who triggered it)
    respond(
        response_type="ephemeral",
        text="Checking status...",
        replace_original=False
    )
    
    # Fetch actual status
    status = get_system_status()
    
    # In-channel (visible to everyone)
    respond(
        response_type="in_channel",
        replace_original=True,
        blocks=[
            {"type": "section", "text": {"type": "mrkdwn", "text": f":green_circle: All systems operational\nLast check: <!date^{int(time.time())}^{{time}}|now>"}}
        ]
    )
```

### Delayed Responses (response_url)

The `response_url` is valid for 30 minutes and allows up to 5 responses:

```python
import requests

@app.command("/deploy")
def handle_deploy(ack, command, respond):
    ack()
    response_url = command["response_url"]
    
    # Immediate acknowledgment
    respond(response_type="ephemeral", text=":hourglass: Starting deployment...")
    
    # Start async deployment
    threading.Thread(target=run_deployment, args=(command, response_url)).start()

def run_deployment(command, response_url):
    # ... long-running deployment ...
    
    # Send delayed response
    requests.post(response_url, json={
        "response_type": "in_channel",
        "replace_original": True,
        "blocks": [
            {"type": "section", "text": {"type": "mrkdwn", "text": ":white_check_mark: Deployment complete!"}}
        ]
    })
```

## 6. External Data Sources for Select Menus

```python
@app.options("team_select")
def handle_team_options(ack, body):
    query = body.get("value", "")
    
    teams = fetch_teams_from_api(query)
    
    options = [
        {"text": {"type": "plain_text", "text": team["name"]}, "value": team["id"]}
        for team in teams
        if query.lower() in team["name"].lower()
    ][:100]  # Max 100 options
    
    ack(options=options)
```

## 7. Message Update Patterns

### Progressive Updates (Long-Running Operations)

```python
async def deploy_with_progress(client, channel, service, version):
    # Initial message
    msg = await client.chat_postMessage(
        channel=channel,
        blocks=[
            {"type": "section", "text": {"type": "mrkdwn", "text": f":hourglass: *Deploying {service} {version}*\n\n:white_circle: Building image\n:white_circle: Pushing to registry\n:white_circle: Updating pods\n:white_circle: Health check"}}
        ]
    )
    ts = msg["ts"]
    
    steps = ["Building image", "Pushing to registry", "Updating pods", "Health check"]
    
    for i, step in enumerate(steps):
        await asyncio.sleep(5)  # Simulate work
        
        progress_lines = []
        for j, s in enumerate(steps):
            if j < i:
                progress_lines.append(f":white_check_mark: ~~{s}~~")
            elif j == i:
                progress_lines.append(f":arrows_counterclockwise: *{s}*")
            else:
                progress_lines.append(f":white_circle: {s}")
        
        await client.chat_update(
            channel=channel,
            ts=ts,
            blocks=[
                {"type": "section", "text": {"type": "mrkdwn", "text": f":rocket: *Deploying {service} {version}*\n\n" + "\n".join(progress_lines)}}
            ]
        )
    
    # Final success
    await client.chat_update(
        channel=channel,
        ts=ts,
        blocks=[
            {"type": "section", "text": {"type": "mrkdwn", "text": f":white_check_mark: *{service} {version} deployed successfully!*\n\n" + "\n".join([f":white_check_mark: ~~{s}~~" for s in steps])}},
            {"type": "context", "elements": [{"type": "mrkdwn", "text": f"Completed in 20s | <!date^{int(time.time())}^{{date_short}} {{time}}|now>"}]}
        ]
    )
```

## 8. Conversation Store Pattern

For apps that need to maintain state across interactions:

```python
import sqlite3
import json

class ConversationStore:
    def __init__(self, db_path="conversations.db"):
        self.conn = sqlite3.connect(db_path, check_same_thread=False)
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS conversations (
                key TEXT PRIMARY KEY,
                data TEXT,
                expires_at REAL
            )
        """)
    
    def set(self, key, data, ttl=3600):
        expires_at = time.time() + ttl
        self.conn.execute(
            "INSERT OR REPLACE INTO conversations (key, data, expires_at) VALUES (?, ?, ?)",
            (key, json.dumps(data), expires_at)
        )
        self.conn.commit()
    
    def get(self, key):
        row = self.conn.execute(
            "SELECT data, expires_at FROM conversations WHERE key = ?", (key,)
        ).fetchone()
        if row and row[1] > time.time():
            return json.loads(row[0])
        return None
    
    def delete(self, key):
        self.conn.execute("DELETE FROM conversations WHERE key = ?", (key,))
        self.conn.commit()

store = ConversationStore()

# Usage: Track multi-step workflows
@app.action("start_deploy")
def handle_start(ack, body, client):
    ack()
    user_id = body["user"]["id"]
    store.set(f"deploy:{user_id}", {"step": 1, "service": None, "env": None}, ttl=600)
    # Open next step...
```

## 9. Scheduled Messages

```python
import datetime

# Schedule a message for a specific time
def schedule_reminder(client, channel, text, when):
    result = client.chat_scheduleMessage(
        channel=channel,
        text=text,
        post_at=int(when.timestamp())
    )
    return result["scheduled_message_id"]

# Schedule for tomorrow at 9 AM
tomorrow_9am = datetime.datetime.now().replace(hour=9, minute=0, second=0) + datetime.timedelta(days=1)
msg_id = schedule_reminder(client, "C012AB3CD", "Don't forget: sprint review today at 2 PM!", tomorrow_9am)

# List scheduled messages
scheduled = client.chat_scheduledMessages_list(channel="C012AB3CD")

# Delete a scheduled message
client.chat_deleteScheduledMessage(channel="C012AB3CD", scheduled_message_id=msg_id)
```

## 10. Rich Text Blocks

Rich text blocks provide structured formatting without mrkdwn parsing:

```json
{
  "type": "rich_text",
  "elements": [
    {
      "type": "rich_text_section",
      "elements": [
        {"type": "text", "text": "Important: ", "style": {"bold": true}},
        {"type": "text", "text": "Please review the following changes before merging."}
      ]
    },
    {
      "type": "rich_text_list",
      "style": "bullet",
      "elements": [
        {"type": "rich_text_section", "elements": [{"type": "text", "text": "Database migration added"}]},
        {"type": "rich_text_section", "elements": [{"type": "text", "text": "API endpoint deprecated"}]},
        {"type": "rich_text_section", "elements": [{"type": "text", "text": "New environment variable required: "}, {"type": "text", "text": "REDIS_URL", "style": {"code": true}}]}
      ]
    },
    {
      "type": "rich_text_preformatted",
      "elements": [
        {"type": "text", "text": "ALTER TABLE users ADD COLUMN preferences JSONB DEFAULT '{}';\nCREATE INDEX idx_users_preferences ON users USING GIN (preferences);"}
      ]
    }
  ]
}
```

This advanced guide covers the most sophisticated patterns for building production-grade Slack applications with proper state management, progressive UIs, and robust error handling.
