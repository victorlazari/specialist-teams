# OpenClaw Specialist Guide

> **Role:** OpenClaw Platform Specialist
> **Domain:** Self-hosted AI gateway, multi-channel messaging, agent runtime, tool orchestration
> **Official Documentation:** [docs.openclaw.ai](https://docs.openclaw.ai/)
> **License:** MIT

---

## 1. Executive Summary and Core Philosophy

OpenClaw is an open-source, self-hosted gateway that connects chat applications to AI coding agents. At its core, OpenClaw operates as a single long-lived Gateway process that bridges messaging platforms (Discord, Slack, Telegram, WhatsApp, and many others) with AI assistants, enabling always-on, multi-channel AI interactions that run entirely on the operator's own hardware. The project is MIT-licensed, community-driven, and designed around the principle that users should retain full control over their data, their infrastructure, and their AI interactions [1].

The fundamental philosophy of OpenClaw centers on three pillars. First, **sovereignty**: the Gateway runs on your hardware under your rules, with no external telemetry or cloud dependency required. Second, **universality**: a single Gateway instance can serve dozens of messaging channels simultaneously, routing conversations to the appropriate agent with isolated sessions. Third, **extensibility**: the platform exposes a rich plugin architecture that allows developers to add new channels, tools, model providers, and skills without modifying the core codebase [1].

OpenClaw requires Node.js 24 (recommended) or Node.js 22 LTS (22.14+) and an API key from a chosen model provider. The entire setup process can be completed in approximately five minutes using the guided onboarding CLI. The default configuration ships with a bundled Pi binary running in RPC mode with per-sender sessions, and the configuration file resides at `~/.openclaw/openclaw.json` [1].

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
openclaw dashboard
```

---

## 2. Architecture and Gateway Runbook

### 2.1 Gateway Architecture

The OpenClaw architecture revolves around a single long-lived Gateway process that owns all messaging surfaces and serves as the central hub for all communication. The Gateway binds to a single multiplexed port (default `127.0.0.1:18789`) that simultaneously handles WebSocket RPC connections, HTTP API endpoints, the Control UI dashboard, and webhook callbacks. This design eliminates the need for multiple processes or complex service meshes — one process handles everything [2].

The architecture comprises four primary component types, each playing a distinct role in the system:

| Component | Connection Type | Role |
|-----------|----------------|------|
| **Gateway (daemon)** | Long-lived process | Maintains provider connections, exposes typed WebSocket API, validates JSON Schema, emits events (agent, chat, presence, health, heartbeat, cron) |
| **Clients** (macOS app, CLI, web admin) | One WebSocket connection per client | Send requests (health, status, send, agent, system-presence), subscribe to events |
| **Nodes** (macOS, iOS, Android, headless) | WebSocket with `role: node` | Device identity, expose commands (canvas.*, camera.*, screen.record, location.get) |
| **WebChat** | Static UI using Gateway WebSocket API | Browser-based chat interface |

A critical architectural constraint is that only **one Gateway per host** should run, as it is the sole process that opens and maintains channel sessions (such as the WhatsApp session via Baileys). The Canvas host is served at `/__openclaw__/canvas/` and the A2UI at `/__openclaw__/a2ui/` [2].

### 2.2 Wire Protocol

All communication between the Gateway and its clients uses WebSocket with text frames containing JSON payloads. The protocol follows a strict handshake sequence where the first frame must be a "connect" message. After connection establishment, the protocol supports two message patterns [2]:

**Request-Response Pattern:** Clients send requests in the format `{type:"req", id, method, params}` and receive responses as `{type:"res", id, ok, payload|error}`. Idempotency keys are required for all side-effecting methods such as `send` and `agent` to prevent duplicate operations.

**Event Subscription Pattern:** The Gateway pushes events to subscribed clients in the format `{type:"event", event, payload, seq?, stateVersion?}`. Events cover agent lifecycle, chat messages, presence updates, health checks, heartbeats, and cron triggers.

Authentication is enforced by default and supports multiple mechanisms: shared-secret token or password, Tailscale Serve for zero-config remote access, and trusted-proxy headers for reverse proxy deployments [2].

### 2.3 Pairing and Local Trust

All WebSocket clients must include a device identity on connect. New device IDs require explicit pairing approval from the operator, ensuring that only authorized devices can interact with the Gateway. Direct local loopback connections can be configured for auto-approval. The signature payload (version 3) binds the platform and device family, and the Gateway pins paired metadata on reconnect to prevent identity spoofing [2].

### 2.4 Hot Reload Configuration

The Gateway supports four hot reload modes that control how configuration changes are applied at runtime:

| Mode | Behavior |
|------|----------|
| `off` | No configuration reload; requires manual restart |
| `hot` | Apply only hot-safe changes without restart |
| `restart` | Full restart on any reload-required change |
| `hybrid` (default) | Hot-apply when safe, restart when required |

### 2.5 Operator Commands

The Gateway provides a comprehensive set of CLI commands for operational management:

```bash
openclaw gateway status              # Basic status check
openclaw gateway status --deep       # Deep health check
openclaw gateway status --json       # Machine-readable output
openclaw gateway install             # Install as system daemon
openclaw gateway restart             # Restart the gateway
openclaw gateway stop                # Stop the gateway
openclaw secrets reload              # Reload secrets without restart
openclaw logs --follow               # Stream live logs
openclaw doctor                      # Diagnose common issues
```

### 2.6 OpenAI-Compatible API Endpoints

The Gateway exposes OpenAI-compatible HTTP endpoints, making it possible to use OpenClaw as a drop-in replacement for OpenAI API calls in existing applications:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/models` | GET | List available models |
| `/v1/models/{id}` | GET | Get specific model details |
| `/v1/embeddings` | POST | Generate embeddings |
| `/v1/chat/completions` | POST | Chat completions |
| `/v1/responses` | POST | Response generation |

The models endpoint returns `openclaw`, `openclaw/default`, and `openclaw/<agentId>` as available model identifiers, enabling agent-first routing [2].

---

## 3. The Agent Loop and Runtime Model

### 3.1 Agent Loop Definition

The agent loop is the fundamental execution unit in OpenClaw. It represents the complete run cycle: intake of a user message, context assembly from session history and workspace files, model inference via the configured provider, tool execution if the model requests it, streaming of replies back to the user, and persistence of the conversation state. Each agent loop is a single, serialized run per session that emits lifecycle and stream events throughout its execution [3].

### 3.2 Execution Flow

The agent loop follows a precise sequence of operations:

1. The `agent` RPC call validates parameters, resolves the target session, persists metadata, and returns `{runId, acceptedAt}` immediately to the caller.
2. The `agentCommand` function resolves the model configuration, loads applicable skills, and invokes `runEmbeddedPiAgent`.
3. The `runEmbeddedPiAgent` function serializes execution via per-session and global queues, resolves model and authentication, subscribes to Pi events, and enforces the configured timeout.
4. The `subscribeEmbeddedPiSession` function bridges internal events to the WebSocket protocol: tool events become "tool" messages, assistant deltas become "assistant" messages, and lifecycle transitions become "lifecycle" messages.
5. The `agent.wait` method allows clients to wait for the lifecycle end or error event for a specific `runId`.

### 3.3 Queueing and Concurrency

OpenClaw implements a sophisticated concurrency model to prevent race conditions during agent execution. Runs are serialized per session key (the "session lane") with an optional global lane that can further limit concurrency across all sessions. This prevents tool and session races that could corrupt state. The queue supports three modes: `collect` (batch incoming messages), `steer` (redirect to active run), and `followup` (queue as next run). A session write lock provides process-aware, file-based, non-reentrant locking by default [3].

### 3.4 Timeouts

| Timeout | Default Value | Purpose |
|---------|---------------|---------|
| `agent.wait` | 30 seconds | Maximum wait for RPC response |
| Agent runtime | 172,800 seconds (48 hours) | Maximum total agent run duration |
| Model idle | 120 seconds | Capped for non-configured providers |

### 3.5 Plugin Hooks

The agent loop exposes a comprehensive set of plugin hooks that allow developers to intercept and modify behavior at every stage of execution:

| Hook | Phase | Purpose |
|------|-------|---------|
| `before_model_resolve` | Pre-session | Override provider or model selection |
| `before_prompt_build` | After session load | Inject context or system prompt modifications |
| `before_agent_start` | Legacy | Backward compatibility |
| `before_agent_reply` | After inline actions | Claim turn or generate synthetic reply |
| `agent_end` | After completion | Inspect final messages |
| `before_compaction` / `after_compaction` | Compaction | Observe or annotate compaction |
| `before_tool_call` / `after_tool_call` | Tool lifecycle | Intercept parameters or results |
| `before_install` | Install | Block skill or plugin installations |
| `tool_result_persist` | Post-tool | Transform results before persistence |
| `message_received` / `message_sending` / `message_sent` | Message pipeline | Inbound and outbound hooks |
| `session_start` / `session_end` | Session lifecycle | Session boundary events |
| `gateway_start` / `gateway_stop` | Gateway lifecycle | Gateway lifecycle events |

---

## 4. Channels and Integration Patterns

### 4.1 Channel Taxonomy

OpenClaw supports an extensive array of messaging channels organized into four categories. This breadth of channel support is one of OpenClaw's most distinctive features, enabling a single Gateway instance to serve users across virtually any messaging platform [1] [4].

**Built-in Channels** are natively integrated into the Gateway core and require no additional plugins:

| Channel | Protocol/API | Key Notes |
|---------|-------------|-----------|
| Discord | Bot API + Gateway | Servers, channels, DMs |
| Google Chat | HTTP webhook | Google Chat API app |
| iMessage (legacy) | imsg CLI | Deprecated; use BlueBubbles |
| IRC | IRC protocol | Classic channels + DMs with pairing/allowlist |
| Signal | signal-cli | Privacy-focused messaging |
| Slack | Bolt SDK | Workspace apps |
| Telegram | Bot API (grammY) | Groups supported, fastest setup |
| WebChat | WebSocket | Gateway WebChat UI |
| WhatsApp | Baileys | QR pairing, most popular channel |

**Bundled Plugin Channels** ship with OpenClaw but run as plugins:

| Channel | Protocol/API | Key Notes |
|---------|-------------|-----------|
| BlueBubbles | REST API | Recommended for iMessage; edit, unsend, effects, reactions, group management |
| Feishu | WebSocket | Feishu/Lark bot |
| LINE | LINE Messaging API | Bot integration |
| Matrix | Matrix protocol | Full protocol support |
| Mattermost | Bot API + WebSocket | Channels, groups, DMs |
| Microsoft Teams | Bot Framework | Enterprise support |
| Nextcloud Talk | Nextcloud Talk API | Self-hosted chat |
| Nostr | NIP-04 | Decentralized DMs |
| QQ Bot | QQ Bot API | Private chat, group chat, rich media |
| Synology Chat | Webhooks | Outgoing + incoming webhooks |
| Tlon | Urbit-based | Messenger |
| Twitch | IRC connection | Chat integration |
| Zalo | Zalo Bot API | Vietnam's popular messenger |
| Zalo Personal | QR login | Personal account access |

**Optional Separate Plugins** require separate installation:

| Channel | Protocol | Notes |
|---------|----------|-------|
| Voice Call | Plivo/Twilio | Telephony integration |
| WeChat | Tencent iLink Bot | QR login, private chats only |

### 4.2 Channel Delivery Patterns

Each channel has specific delivery behaviors that the specialist must understand. Telegram converts markdown image syntax to media replies on outbound messages. Slack routes multi-person DMs as group chats. WhatsApp uses install-on-demand architecture where the runtime is loaded only when the channel is active, conserving resources when WhatsApp is not in use [4].

### 4.3 Group Chat and DM Safety

Group chat activation is mention-based, meaning the agent only responds when explicitly mentioned (e.g., `@agent`). Direct message safety is enforced through allowlists and pairing mechanisms, preventing unauthorized users from interacting with the agent. Sessions in direct chats collapse into a shared main session, while group chat sessions are isolated per group [4].

---

## 5. Tools and Plugin Ecosystem

### 5.1 Three-Layer Architecture

OpenClaw's extensibility is built on three distinct layers that work together to provide a comprehensive tool ecosystem [5]:

**Tools** are typed functions that the agent can call during execution. Each tool has a defined schema, input parameters, and output format. Tools handle concrete actions like executing shell commands, browsing the web, or sending messages.

**Skills** are markdown files (named `SKILL.md`) that are injected into the system prompt to provide the agent with domain-specific guidance. Skills do not execute code but instead shape the agent's behavior through natural language instructions.

**Plugins** are packages that can register any combination of channels, model providers, tools, skills, and speech capabilities. Plugins are the primary extension mechanism for adding new functionality to OpenClaw.

### 5.2 Built-in Tools Reference

| Tool | Purpose |
|------|---------|
| `exec` / `process` | Shell commands, background processes |
| `code_execution` | Sandboxed remote Python analysis |
| `browser` | Chromium browser control (navigate, click, screenshot) |
| `web_search` / `x_search` / `web_fetch` | Web search, X posts search, page fetch |
| `read` / `write` / `edit` | File I/O in workspace |
| `apply_patch` | Multi-hunk file patches |
| `message` | Send messages across all channels |
| `canvas` | Drive node Canvas (present, eval, snapshot) |
| `nodes` | Discover and target paired devices |
| `cron` / `gateway` | Scheduled jobs; inspect/patch/restart gateway |
| `image` / `image_generate` | Analyze or generate images |
| `music_generate` | Generate music tracks |
| `video_generate` | Generate videos |
| `tts` | Text-to-speech conversion |
| `sessions_*` / `subagents` / `agents_list` | Session management, sub-agent orchestration |
| `session_status` | Lightweight status readback and session model override |

### 5.3 Tool Profiles and Groups

Tool profiles control which tools are available to the agent in different contexts:

| Profile | Tools Included |
|---------|---------------|
| `full` | No restriction — all tools available |
| `coding` | group:fs, group:runtime, group:web, group:sessions, group:memory, cron, image, image_generate, music_generate, video_generate |
| `messaging` | group:messaging, sessions_list, sessions_history, sessions_send, session_status |
| `minimal` | session_status only |

Tool groups provide logical groupings for access control:

| Group | Tools |
|-------|-------|
| `group:runtime` | exec, process, code_execution |
| `group:fs` | read, write, edit, apply_patch |
| `group:web` | web_search, x_search, web_fetch |
| `group:ui` | browser, canvas |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:nodes` | nodes |
| `group:agents` | agents_list |
| `group:media` | image, image_generate, music_generate, video_generate, tts |
| `group:openclaw` | All built-in OpenClaw tools |

### 5.4 Tool Access Control

Tool access is controlled through allow/deny lists configured via `tools.allow` and `tools.deny`. The deny list always takes precedence over the allow list. If the resolved allowlist contains no callable tools, the system fails closed (the agent cannot call any tools). Provider-specific restrictions can be configured via `tools.byProvider` to limit which tools are available when using specific model providers [5].

### 5.5 Web Search Providers

OpenClaw supports a wide range of web search providers: Brave, DuckDuckGo, Exa, Firecrawl, Gemini, Grok, Kimi, MiniMax Search, Ollama Web Search, Perplexity, SearXNG, and Tavily. This diversity allows operators to choose the search provider that best fits their privacy requirements and use case [4].

---

## 6. Models and Provider Management

### 6.1 Provider Directory

OpenClaw supports over 50 model providers, making it one of the most provider-agnostic AI gateways available. The complete list includes: Alibaba Model Studio, Amazon Bedrock, Amazon Bedrock Mantle, Anthropic (API + Claude CLI), Arcee AI, Azure Speech, BytePlus, Cerebras, Chutes, Cloudflare AI Gateway, ComfyUI, DeepSeek, ElevenLabs, fal, Fireworks, GitHub Copilot, GLM, Google (Gemini), Gradium, Groq, Hugging Face, inferrs, Kilocode, LiteLLM, LM Studio, MiniMax, Mistral, Moonshot AI, NVIDIA, Ollama, OpenAI (API + Codex), OpenCode, OpenCode Go, OpenRouter, Perplexity, Qianfan, Qwen Cloud, Runway, SenseAudio, SGLang, StepFun, Synthetic, Tencent Cloud, Together AI, Venice, Vercel AI Gateway, vLLM, Volcengine, Vydra, xAI, Xiaomi, and Z.AI [6].

### 6.2 Provider Categories

| Category | Providers |
|----------|-----------|
| **Major Cloud** | Anthropic, OpenAI, Google (Gemini), Amazon Bedrock, Azure |
| **Open Source / Self-Hosted** | Ollama, vLLM, SGLang, LM Studio, LiteLLM |
| **Specialized** | ElevenLabs (TTS), Runway (video), fal (media), ComfyUI (images) |
| **Regional** | Alibaba, BytePlus, DeepSeek, GLM, MiniMax, Moonshot AI, Qianfan, Qwen Cloud, Tencent Cloud, Volcengine, Xiaomi, Z.AI |
| **Aggregators** | OpenRouter, Cloudflare AI Gateway, Vercel AI Gateway, Fireworks, Together AI, Groq, Cerebras |

### 6.3 Transcription and Media Generation

For speech-to-text capabilities, OpenClaw supports: Deepgram, ElevenLabs, Mistral, OpenAI, SenseAudio, and xAI. Media generation tools work across providers with automatic failover [6]:

| Capability | Tool | Description |
|-----------|------|-------------|
| Image Generation | `image_generate` | Provider selection with failover |
| Music Generation | `music_generate` | Generate music tracks |
| Video Generation | `video_generate` | Generate videos |

Provider authentication is handled through the `openclaw onboard` CLI command, which guides the operator through credential setup. The default model is configured via `agents.defaults.model.primary` in the configuration file. For example: `agents.defaults.model.primary: "anthropic/claude-opus-4-6"` [6].

---

## 7. Operational Readiness and Diagnostics

### 7.1 VoiceClaw Real-Time Brain

OpenClaw includes VoiceClaw, a real-time voice interaction system accessible via a WebSocket endpoint at `/voiceclaw/realtime`. VoiceClaw uses Gemini Live for real-time audio processing, enabling voice-based interactions with the AI agent. Tool calls in VoiceClaw return an immediate working result to maintain conversational flow, followed by asynchronous execution of the actual tool operation [2].

### 7.2 Health and Diagnostics

The `openclaw doctor` command provides a comprehensive diagnostic check of the entire system, identifying common issues with configuration, connectivity, and dependencies. The `openclaw gateway status --deep` command performs an in-depth health check that validates all channel connections, provider availability, and system resources [2].

### 7.3 Session Management

Sessions in OpenClaw follow specific routing rules. Direct chat sessions collapse into a shared main session per user, while group chat sessions are isolated per group. The system supports session pruning to manage memory usage, and session tools allow programmatic access to session history and state. Multi-agent routing enables isolated sessions per agent, workspace, or sender [3].

### 7.4 Memory and Compaction

OpenClaw implements a compaction system for managing conversation memory. As sessions grow, the compaction engine summarizes older messages to reduce context window usage while preserving essential information. The `before_compaction` and `after_compaction` hooks allow plugins to observe and annotate the compaction process [3].

### 7.5 Security Considerations

The Gateway exposes a comprehensive security surface that operators must understand. Authentication is required by default, with support for shared-secret tokens, passwords, and trusted-proxy configurations. The pairing system ensures that only approved devices can connect. Network binding defaults to loopback (127.0.0.1) to prevent external access unless explicitly configured. For remote access, OpenClaw integrates with Tailscale for zero-config VPN connectivity [2].

### 7.6 Apps and Interfaces

OpenClaw provides multiple client interfaces for different platforms and use cases:

| Interface | Platform | Key Features |
|-----------|----------|--------------|
| WebChat | Browser | Gateway WebChat UI, embeddable |
| Control UI | Browser | Configuration, monitoring, management dashboard |
| macOS App | macOS | Menu bar companion app |
| iOS Node | iOS | Pairing, Canvas, camera, screen recording, location, voice |
| Android Node | Android | Pairing, chat, voice, Canvas, camera, device commands |

### 7.7 Cron Jobs and Automation

OpenClaw supports cron-based scheduling through the `cron` tool, enabling agents to perform automated tasks on a schedule. The heartbeat system provides regular health checks and can trigger automated responses when issues are detected. The Lobster workflow pipeline system enables complex multi-step automation chains [4].

---

## References

[1]: [OpenClaw Documentation - Main Page](https://docs.openclaw.ai/)
[2]: [OpenClaw Gateway Architecture](https://docs.openclaw.ai/concepts/architecture)
[3]: [OpenClaw Agent Loop](https://docs.openclaw.ai/concepts/agent-loop)
[4]: [OpenClaw Features](https://docs.openclaw.ai/concepts/features)
[5]: [OpenClaw Tools and Plugins](https://docs.openclaw.ai/tools)
[6]: [OpenClaw Providers](https://docs.openclaw.ai/providers)
