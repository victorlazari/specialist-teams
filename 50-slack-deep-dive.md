# Slack Master Specialist — Deep Dive: Architecture and Internals

## 1. Slack Platform Architecture

### Message Flow

```
User types message
    → Slack Client (desktop/mobile/web)
    → Slack Gateway (WebSocket connection)
    → Message Service (validation, storage)
    → Channel Service (membership, permissions)
    → Search Index (Elasticsearch)
    → Event Dispatch (fan-out to subscribers)
    → App Event Delivery (HTTP POST or WebSocket)
    → Your App Server
```

### Real-Time Messaging (RTM) vs Events API vs Socket Mode

| Feature | RTM (deprecated) | Events API | Socket Mode |
|---------|-----------------|------------|-------------|
| Connection | WebSocket | HTTP POST | WebSocket |
| Public URL needed | No | Yes | No |
| Scalability | Limited (1 conn/workspace) | High | Medium |
| Event types | All | Subscribed only | Subscribed only |
| Recommended | No | Production | Development/Internal |

### Event Delivery Guarantees

- Events are delivered **at least once** (not exactly once)
- Slack retries up to 3 times with exponential backoff
- Retry intervals: ~1 min, ~5 min, ~30 min
- After 3 failures, the event is dropped and a warning appears in app dashboard
- Your app must handle duplicates (use `event_id` for deduplication)

## 2. Block Kit Rendering Engine

### How Blocks Are Rendered

```
Block Kit JSON
    → Server-side validation (schema check)
    → Client receives block array
    → React component tree (desktop/web)
    → Native component tree (mobile)
    → Layout engine (flexbox-like)
    → Rendered UI
```

### Block Limits

| Constraint | Limit |
|-----------|-------|
| Blocks per message | 50 |
| Blocks per modal | 100 |
| Blocks per App Home | 100 |
| Actions per actions block | 25 |
| Elements per context block | 10 |
| Fields per section block | 10 |
| Options per select menu | 100 |
| Option groups per select | 100 |
| Characters in mrkdwn text | 3,000 |
| Characters in plain_text | 3,000 |
| Characters in header | 150 |
| Characters in button text | 75 |
| Characters in option text | 75 |
| Characters in placeholder | 150 |
| Total message size | 40,000 chars |

### Text Object Types

| Type | Parsing | Use Case |
|------|---------|----------|
| `plain_text` | No formatting | Buttons, headers, labels, placeholders |
| `mrkdwn` | Slack markdown | Section text, context, fields |

mrkdwn supports: `*bold*`, `_italic_`, `~strike~`, `` `code` ``, ` ```code block``` `, `> quote`, `>>> block quote`, `<url|text>`, `<@user>`, `<#channel>`, `<!date^ts^format|fallback>`

## 3. OAuth 2.0 Implementation Details

### Token Types Deep Dive

**Bot Tokens (xoxb-)**
- Represent the app's bot user
- Persist across user sessions
- Limited to bot scopes
- One per workspace installation
- Cannot access user-specific data (search, profile write)

**User Tokens (xoxp-)**
- Represent a specific user
- Can access user-specific APIs (search, profile)
- Required for admin APIs
- One per user who authorizes
- Higher security risk

**App-Level Tokens (xapp-)**
- Used exclusively for Socket Mode connections
- Cannot call Web API methods
- One per app (not per workspace)
- Long-lived, rarely rotated

### Token Lifecycle

```
App Installation
    → OAuth flow initiated
    → User authorizes scopes
    → Slack issues access_token (+ refresh_token if rotation enabled)
    → Token stored by your app
    → Token used for API calls
    → [If rotation enabled] Token expires after 12 hours
    → Refresh token used to get new access_token
    → [If app uninstalled] Token revoked, tokens.revoked event sent
```

## 4. Interaction Payload Flow

### Complete Interaction Lifecycle

```
User clicks button/submits form
    → Slack sends interaction payload to Request URL
    → Your app receives payload (must respond in 3s)
    → Acknowledge with HTTP 200
    → [Optional] Return updated message/view in response body
    → [Optional] Use response_url for follow-up (within 30 min, max 5)
    → [Optional] Use trigger_id to open modal (within 3s)
    → [Optional] Call Web API for additional actions
```

### trigger_id Expiration

The `trigger_id` is valid for exactly 3 seconds from when the interaction occurred. This means:
- You must open modals immediately in your request handler
- You cannot fetch data from external APIs before opening a modal
- Pattern: Open modal with loading state, then update it with data

```python
@app.action("open_details")
def handle(ack, body, client):
    ack()
    
    # Open modal immediately with loading state
    result = client.views_open(
        trigger_id=body["trigger_id"],
        view={
            "type": "modal",
            "callback_id": "details_modal",
            "title": {"type": "plain_text", "text": "Loading..."},
            "blocks": [{"type": "section", "text": {"type": "mrkdwn", "text": ":hourglass: Loading details..."}}]
        }
    )
    
    # Now fetch data (can take as long as needed)
    data = fetch_details_from_api(body["actions"][0]["value"])
    
    # Update the modal with real data
    client.views_update(
        view_id=result["view"]["id"],
        view={
            "type": "modal",
            "callback_id": "details_modal",
            "title": {"type": "plain_text", "text": "Details"},
            "blocks": build_detail_blocks(data)
        }
    )
```

## 5. Message Storage and Retrieval

### Message Timestamps (ts)

The `ts` field is the unique identifier for messages within a channel:
- Format: `"1234567890.123456"` (Unix timestamp with microseconds)
- Used as message ID for updates, deletions, reactions, threading
- Thread parent `ts` becomes the `thread_ts` for replies
- Timestamps are unique within a channel but not globally

### Message History Pagination

```
conversations.history
    → Returns messages in reverse chronological order
    → Default limit: 100 messages
    → Max limit: 1000 messages
    → Use cursor-based pagination for more
    → oldest/latest parameters for time-based filtering
    → inclusive parameter to include boundary messages
```

### Message Subtypes

| Subtype | Meaning |
|---------|---------|
| `null` | Regular user message |
| `bot_message` | Posted by a bot (legacy) |
| `message_changed` | Message was edited |
| `message_deleted` | Message was deleted |
| `channel_join` | User joined channel |
| `channel_leave` | User left channel |
| `channel_topic` | Topic was changed |
| `channel_purpose` | Purpose was changed |
| `channel_name` | Channel was renamed |
| `file_share` | File was shared |
| `thread_broadcast` | Thread reply broadcast to channel |
| `tombstone` | DLP-deleted message placeholder |

## 6. File System

### File Upload Flow (V2)

```
1. Client requests upload URL
   POST files.getUploadURLExternal
   → Returns: upload_url, file_id

2. Client uploads file content
   POST {upload_url}
   → Binary file data uploaded directly

3. Client completes upload
   POST files.completeUploadExternal
   → File associated with channel/thread
   → Thumbnails generated
   → Search indexed
   → Virus scanned
```

### File Types and Processing

| Category | Types | Processing |
|----------|-------|-----------|
| Images | png, jpg, gif, svg, webp | Thumbnails, preview |
| Documents | pdf, doc, docx, ppt, pptx | Preview, text extraction |
| Code | py, js, ts, go, rb, etc. | Syntax highlighting |
| Archives | zip, tar, gz | File listing |
| Audio | mp3, wav, m4a | Playback, transcription |
| Video | mp4, webm, mov | Playback, thumbnails |

## 7. Search Architecture

### How Slack Search Works

- Uses Elasticsearch under the hood
- Indexes: messages, files, channels, users
- Real-time indexing (messages searchable within seconds)
- Supports: exact match, phrase, boolean operators
- Filters: from, in, during, has, before, after
- Ranking: relevance + recency + user affinity

### Search Modifiers

```
from:@username        → Messages from specific user
in:#channel           → Messages in specific channel
has:link              → Messages containing URLs
has:reaction          → Messages with reactions
has::emoji:           → Messages with specific reaction
before:2025-01-01    → Messages before date
after:2025-01-01     → Messages after date
during:january       → Messages during month
on:2025-05-29        → Messages on specific date
is:thread            → Messages that are thread replies
```

## 8. Workspace Data Model

### Entity Relationships

```
Enterprise (org)
  └── Workspaces (teams)
       ├── Users (members)
       │    ├── Profile
       │    ├── Status
       │    └── Preferences
       ├── Channels (conversations)
       │    ├── Public channels
       │    ├── Private channels
       │    ├── DMs (im)
       │    ├── Group DMs (mpim)
       │    └── Shared channels (Slack Connect)
       ├── User Groups
       ├── Apps (installed)
       │    ├── Bot User
       │    ├── Tokens
       │    └── Permissions
       └── Files
            ├── Uploads
            ├── Snippets
            └── Posts
```

### Channel Types

| Type | API Type | Properties |
|------|----------|-----------|
| Public | `public_channel` | Anyone can join, searchable |
| Private | `private_channel` | Invite only, not searchable by non-members |
| DM | `im` | 1:1 conversation |
| Group DM | `mpim` | Multi-party DM (up to 9 people) |
| Shared | `public_channel` + `is_shared` | Cross-organization |

## 9. Rate Limiting Internals

### How Rate Limits Are Calculated

Slack uses a token bucket algorithm per app per workspace:
- Each tier has a bucket size (burst capacity)
- Tokens refill at a steady rate
- When bucket is empty, requests get 429 response
- `Retry-After` header tells you when to retry

### Special Rate Limits

| Scenario | Limit | Notes |
|----------|-------|-------|
| `chat.postMessage` per channel | 1/sec | Prevents channel flooding |
| Incoming webhooks | 1/sec | Per webhook URL |
| Web API overall | Varies by tier | Per app per workspace |
| Events API delivery | No limit from Slack | But your server must handle |
| Socket Mode | 30,000 events/hour | Per app |

### Handling Burst Traffic

```python
import asyncio
from collections import defaultdict

class ChannelRateLimiter:
    """Ensures max 1 message per second per channel"""
    
    def __init__(self):
        self.last_post = defaultdict(float)
        self.lock = asyncio.Lock()
    
    async def wait_for_channel(self, channel_id):
        async with self.lock:
            now = asyncio.get_event_loop().time()
            elapsed = now - self.last_post[channel_id]
            if elapsed < 1.0:
                await asyncio.sleep(1.0 - elapsed)
            self.last_post[channel_id] = asyncio.get_event_loop().time()
```

## 10. Enterprise Grid Architecture

### Multi-Workspace Model

```
Organization (Enterprise Grid)
├── Org-level settings
│    ├── App management policies
│    ├── Information barriers
│    ├── DLP policies
│    └── Audit logs
├── Workspace A
│    ├── Local channels
│    ├── Local apps
│    └── Local users
├── Workspace B
│    ├── Local channels
│    ├── Local apps
│    └── Local users
└── Org-wide channels
     └── Available across all workspaces
```

### Org-Level vs Workspace-Level

| Feature | Org-Level | Workspace-Level |
|---------|-----------|-----------------|
| App approval | Org admin controls | Workspace admin installs |
| User management | Provisioned centrally | Workspace membership |
| Channels | Org-wide channels | Local channels |
| Policies | DLP, retention | Channel-specific |
| Audit logs | All workspaces | Single workspace |
| SSO/SAML | Configured once | Inherited |

## 11. Slack Connect Internals

### How Shared Channels Work

```
Workspace A                    Workspace B
    │                              │
    ├── Channel #shared-proj       │
    │       │                      │
    │       └── Shared via ────────┤
    │           Slack Connect      │
    │                              ├── Same channel appears
    │                              │   in Workspace B
    │                              │
    Messages are stored in both workspaces
    Each workspace applies its own:
    - Retention policies
    - DLP rules
    - Compliance exports
    - App access rules
```

### Security Boundaries

- Each org controls its own data retention
- Files can be restricted from external sharing
- Apps in one workspace cannot access the other's data
- User profiles show limited info to external users
- Admin controls for who can create shared channels

## 12. Performance Characteristics

### API Response Times (Typical)

| Method | p50 | p95 | p99 |
|--------|-----|-----|-----|
| `auth.test` | 50ms | 150ms | 500ms |
| `chat.postMessage` | 100ms | 300ms | 1000ms |
| `conversations.list` | 200ms | 500ms | 2000ms |
| `users.list` | 300ms | 800ms | 3000ms |
| `views.open` | 150ms | 400ms | 1500ms |
| `files.upload` | 500ms | 2000ms | 5000ms |

### Optimizing API Usage

1. **Cache aggressively**: User info, channel info rarely change
2. **Batch operations**: Use `conversations.list` instead of individual `info` calls
3. **Paginate efficiently**: Use max `limit` to reduce round trips
4. **Use Socket Mode**: Eliminates HTTP overhead for events
5. **Minimize blocks**: Fewer blocks = faster rendering
6. **Lazy load**: Don't fetch data until user requests it

## 13. Message Delivery Semantics

### Ordering Guarantees

- Messages within a channel are ordered by `ts`
- Events may arrive out of order (use `event_ts` for ordering)
- Thread replies are ordered within the thread
- Edited messages keep their original `ts` (position)

### Consistency Model

- Messages are eventually consistent across clients
- Real-time delivery is best-effort (WebSocket may drop)
- API reads are strongly consistent (always see latest)
- Search index has slight delay (seconds to minutes)

## 14. Slack's Technology Stack (Public Knowledge)

Based on publicly available information:
- **Backend**: PHP (legacy), Java, Go (newer services)
- **Frontend**: React (web), Electron (desktop)
- **Mobile**: React Native + native modules
- **Database**: MySQL (sharded), Vitess
- **Cache**: Memcached, Redis
- **Search**: Elasticsearch
- **Queue**: Kafka
- **Storage**: S3-compatible
- **CDN**: CloudFront
- **Real-time**: Custom WebSocket infrastructure

## 15. Future Platform Direction

### Next-Gen Platform (Deno-based)

- Functions run on Slack's infrastructure (no server needed)
- Deno runtime for TypeScript/JavaScript
- Built-in datastores (key-value)
- Triggers replace traditional event subscriptions
- Workflows compose functions visually
- Automatic scaling and zero-ops deployment

### Deprecation Timeline

| Feature | Status | Replacement |
|---------|--------|-------------|
| RTM API | Deprecated | Events API + Socket Mode |
| Legacy attachments | Deprecated | Block Kit |
| Classic apps | Deprecated | New Slack apps |
| Workflow Steps from Apps | Deprecated | Platform Functions |
| `files.upload` (v1) | Deprecated | `files.uploadV2` |

This deep dive provides the architectural understanding needed to build robust, performant Slack integrations that work with the platform rather than against it.
