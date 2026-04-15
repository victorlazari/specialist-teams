# OpenClaw Specialist

> **Role:** OpenClaw Platform Expert — Architecture, Deployment, Multi-Agent Routing, and Plugin Development
> **Official Sources:** [OpenClaw Documentation](https://docs.openclaw.ai/) | [GitHub Repository](https://github.com/openclaw/openclaw)

---

## 1. Introduction and Overview

OpenClaw is an open-source personal AI assistant gateway licensed under the MIT License. With over 100,000 stars on GitHub, it has become one of the most widely adopted self-hosted AI gateway platforms in the ecosystem. OpenClaw operates as a bridge between everyday chat applications and powerful AI coding agents, enabling users to interact with sophisticated AI models through familiar messaging interfaces such as WhatsApp, Telegram, Slack, Discord, and many more.

The platform is designed around the principle of **self-sovereignty** — users retain complete control over their data, AI interactions, and infrastructure. Unlike cloud-dependent solutions, OpenClaw runs locally or on private servers, providing unparalleled customization and privacy guarantees. The gateway architecture supports multi-agent routing with isolated sessions, tool streaming, and a rich plugin ecosystem that allows specialists to extend functionality to meet virtually any use case.

OpenClaw requires **Node.js 24** (recommended) or **Node.js 22 LTS** (version 22.14 or newer) as its runtime environment. The platform is installed globally via npm and managed through a comprehensive CLI that handles onboarding, daemon management, and dashboard access.

---

## 2. Architecture and Core Components

The OpenClaw architecture follows a modular, event-driven design built on Node.js's asynchronous I/O capabilities. Understanding the architectural layers is essential for effective deployment and troubleshooting.

### 2.1 High-Level Architecture

| Component | Description | Responsibility |
|---|---|---|
| **Gateway Core** | Central message broker and routing engine | Receives messages from channels, routes to agents, returns responses |
| **Channel Adapters** | Protocol-specific connectors | Translate between platform-native formats and OpenClaw's internal message format |
| **Agent Runtime** | Embedded execution environment | Manages agent lifecycles, tool streaming, and session isolation |
| **Model Provider Layer** | LLM integration abstraction | Connects to 35+ AI providers (Anthropic, OpenAI, Google, etc.) |
| **Tooling Layer** | External capability interface | Browser automation, exec, sandboxing, web search engines |
| **Session Manager** | State persistence engine | Maintains conversational context across turns and agents |
| **Skills/Plugins Engine** | Extension framework | Loads and executes custom skills and third-party plugins |
| **Web Control UI** | Administration dashboard | Configuration, monitoring, and management interface |
| **Daemon Manager** | Process lifecycle controller | Ensures persistent background operation |

### 2.2 Message Flow

When a user sends a message through any connected chat platform, the flow proceeds as follows. The Channel Adapter receives the raw message and normalizes it into OpenClaw's internal format. The Message Parser analyzes the content to determine intent, target agent, and any skill invocations. The Router dispatches the message to the appropriate Agent Instance within the Agent Runtime. The agent processes the message, potentially invoking tools (browser, exec, search) or querying an LLM through the Model Provider Layer. The response travels back through the same chain, being translated back into the platform-native format before delivery.

This architecture ensures that adding a new chat platform requires only implementing a new Channel Adapter, while the rest of the system remains unchanged. Similarly, new AI providers can be integrated by adding a Model Provider module without affecting channel or agent logic.

### 2.3 Configuration System

OpenClaw centralizes its configuration in `~/.openclaw/openclaw.json`. This file contains all channel credentials, agent definitions, model provider API keys, tool configurations, and system preferences. The configuration can be managed through the Web Control UI, the CLI, or by directly editing the JSON file.

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "phoneNumberId": "YOUR_PHONE_NUMBER_ID",
      "accessToken": "YOUR_ACCESS_TOKEN",
      "webhookVerifyToken": "YOUR_VERIFY_TOKEN"
    },
    "telegram": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN"
    }
  },
  "agents": {
    "default": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514",
      "systemPrompt": "You are a helpful assistant.",
      "tools": ["browser", "exec", "search"]
    }
  },
  "providers": {
    "anthropic": {
      "apiKey": "YOUR_ANTHROPIC_KEY"
    },
    "openai": {
      "apiKey": "YOUR_OPENAI_KEY"
    }
  }
}
```

---

## 3. Supported Channels

OpenClaw supports an extensive list of chat platforms, making it one of the most versatile AI gateways available. Each channel adapter handles the specific authentication, webhook, and message format requirements of its platform.

| Channel | Protocol | Auth Method | Media Support | Notes |
|---|---|---|---|---|
| **WhatsApp** | Cloud API / Baileys | Phone Number ID + Access Token | Images, audio, video, documents | Business API or unofficial Baileys |
| **Telegram** | Bot API | Bot Token | Full media support | Supports groups, channels, inline mode |
| **Slack** | Events API + Web API | Bot Token + App Token | Files, images, threads | Socket Mode or HTTP webhooks |
| **Discord** | Gateway + REST | Bot Token | Attachments, embeds | Supports slash commands |
| **Signal** | Signal CLI / signald | Phone number registration | Images, attachments | Requires Signal account |
| **iMessage** | AppleScript / BlueBubbles | macOS host required | Images, attachments | macOS-only integration |
| **Google Chat** | Chat API | Service Account | Cards, attachments | Google Workspace integration |
| **IRC** | IRC protocol | Server + nick | Text only | Classic protocol support |
| **Matrix** | Matrix Client-Server API | Access token | Full media | Federated protocol |
| **WebChat** | WebSocket | Configurable auth | Full media | Embeddable widget |
| **Zalo** | Zalo OA API | OA credentials | Images, files | Vietnamese platform |
| **Nostr** | NIP protocol | nsec key | Text, media | Decentralized protocol |
| **Twitch** | IRC + EventSub | OAuth token | Text | Streaming platform integration |

### 3.1 Channel Configuration Best Practices

When configuring channels, specialists should follow these guidelines. First, enable only the channels that are actively needed to reduce the attack surface and resource consumption. Second, use environment variables or secrets management for API keys rather than hardcoding them in the configuration file. Third, configure allowlists to restrict which users or groups can interact with the AI agents. Fourth, set up mention rules to control when the bot responds (e.g., only when mentioned by name, or on all messages in DMs). Fifth, test each channel independently before enabling multi-channel operation to isolate configuration issues.

---

## 4. Agent Configuration and Multi-Agent Routing

The agent system is the heart of OpenClaw's AI capabilities. Agents are configured with specific model providers, system prompts, tool access, and behavioral parameters.

### 4.1 Agent Definition

Each agent is defined with the following properties:

```json
{
  "agents": {
    "coder": {
      "provider": "anthropic",
      "model": "claude-sonnet-4-20250514",
      "systemPrompt": "You are an expert software engineer...",
      "tools": ["browser", "exec", "search", "sandbox"],
      "maxTokens": 8192,
      "temperature": 0.3
    },
    "researcher": {
      "provider": "openai",
      "model": "gpt-4.1",
      "systemPrompt": "You are a research specialist...",
      "tools": ["search", "browser"],
      "maxTokens": 4096,
      "temperature": 0.7
    }
  }
}
```

### 4.2 Multi-Agent Routing

OpenClaw supports sophisticated multi-agent routing with isolated sessions. The routing system determines which agent handles each incoming message based on configurable rules. Routing can be based on channel (e.g., Slack messages go to the coder agent), user identity, message content patterns, or explicit agent selection via commands.

The session isolation ensures that each agent maintains its own conversational context, preventing cross-contamination between different AI personas. This is particularly important when running agents with different security levels or access permissions.

> **Key Concept:** Multi-agent routing in OpenClaw uses a priority-based rule system. Rules are evaluated in order, and the first matching rule determines the target agent. A default agent handles messages that do not match any specific rule.

### 4.3 Session Management

Sessions in OpenClaw track the state of each conversation. A session is identified by a combination of channel, user, and agent. The Session Manager persists context across turns, enabling multi-turn conversations with memory. Sessions can be configured with TTL (time-to-live) values to automatically expire stale conversations, and specialists can implement custom session storage backends for scalability.

---

## 5. Tools and Capabilities

OpenClaw provides agents with a rich set of built-in tools that extend their capabilities beyond pure text generation.

### 5.1 Built-in Tools

| Tool | Description | Use Cases |
|---|---|---|
| **Browser** | Headless browser automation | Web scraping, form filling, screenshot capture |
| **Exec** | Arbitrary command execution | Running scripts, system administration, file operations |
| **Sandbox** | Isolated execution environment | Safe code execution, testing untrusted code |
| **Web Search** | Multi-engine search integration | Information retrieval, fact-checking, research |
| **File System** | File read/write operations | Document processing, data management |

### 5.2 Web Search Providers

OpenClaw integrates with multiple search engines, allowing agents to retrieve up-to-date information. Supported providers include Brave Search, DuckDuckGo, Exa, Firecrawl, Google (via SerpAPI or Custom Search), and Bing. Each provider can be configured with API keys and search parameters.

### 5.3 Media Handling

The platform supports comprehensive media processing across all connected channels. Agents can receive and process images, audio files, video files, and documents. They can also generate images (via DALL-E, Stable Diffusion, or other providers), generate video, transcribe voice notes to text, and convert text to speech (TTS). Media handling is channel-aware, automatically adapting file formats and sizes to meet each platform's requirements.

---

## 6. Skills, Plugins, and Workflow Pipelines

### 6.1 Skills

Skills are modular capabilities that extend an agent's functionality. A skill is represented as a directory containing instructions, metadata, and optional resources (scripts, templates). Skills can be shared across agents and are loaded dynamically at runtime.

### 6.2 Plugins

Plugins provide deeper integration points than skills, allowing developers to modify OpenClaw's core behavior. Plugins can intercept messages, add new channel adapters, implement custom authentication, or extend the tooling layer. The plugin API provides hooks into the message lifecycle (pre-processing, post-processing, error handling) and the agent runtime.

### 6.3 Lobster Workflow Pipelines

Lobster is OpenClaw's workflow orchestration engine. It enables the creation of complex, multi-step automated processes that chain together agent actions, tool invocations, and conditional logic. Workflows can be triggered by cron schedules, incoming messages, or external webhooks.

```yaml
name: daily-report
trigger:
  cron: "0 9 * * 1-5"
steps:
  - agent: researcher
    action: search
    query: "latest industry news"
    output: news_results
  - agent: coder
    action: generate
    prompt: "Create a summary report from: {{news_results}}"
    output: report
  - channel: slack
    action: send
    target: "#daily-reports"
    message: "{{report}}"
```

---

## 7. Installation, CLI Reference, and Deployment

### 7.1 Installation

```bash
# Install OpenClaw globally
npm install -g openclaw@latest

# Run the onboarding wizard with daemon installation
openclaw onboard --install-daemon

# Open the Web Control UI dashboard
openclaw dashboard
```

### 7.2 CLI Reference

| Command | Description |
|---|---|
| `openclaw onboard` | Interactive setup wizard |
| `openclaw onboard --install-daemon` | Setup with daemon auto-start |
| `openclaw dashboard` | Open Web Control UI |
| `openclaw start` | Start the gateway |
| `openclaw stop` | Stop the gateway |
| `openclaw restart` | Restart the gateway |
| `openclaw status` | Check gateway status |
| `openclaw config` | View/edit configuration |
| `openclaw logs` | View gateway logs |
| `openclaw update` | Update to latest version |
| `openclaw plugin install <name>` | Install a plugin |
| `openclaw skill add <path>` | Add a skill directory |

### 7.3 Deployment Patterns

For production deployments, specialists should consider the following patterns. **Single-server deployment** is suitable for personal use or small teams, running OpenClaw as a daemon on a single machine. **Docker deployment** provides containerization for consistent environments and easier scaling. **Reverse proxy deployment** places OpenClaw behind Nginx or Caddy for SSL termination, rate limiting, and load balancing. **Remote access** can be achieved through SSH tunneling, Tailscale, or Cloudflare Tunnel for secure access without exposing ports directly.

---

## 8. Security Hardening

Security is a critical concern when operating an AI gateway that connects to multiple external services and processes user messages.

### 8.1 Access Control

OpenClaw supports allowlists at multiple levels. Channel-level allowlists restrict which users or groups can send messages. Agent-level allowlists control which users can access specific agents. Tool-level restrictions limit which tools each agent can invoke.

### 8.2 Token and Secret Management

All API keys, bot tokens, and credentials should be managed through environment variables or a secrets manager rather than being stored in plain text in the configuration file. OpenClaw supports environment variable interpolation in its configuration.

### 8.3 DM Safety and Mention Rules

Configure mention rules to prevent the bot from responding to every message in group chats. In DM (direct message) contexts, implement rate limiting and content filtering to prevent abuse. The safety configuration allows specialists to define content policies that agents must follow.

### 8.4 Network Security

When deploying OpenClaw, ensure that webhook endpoints are protected with verification tokens, the Web Control UI is accessible only through authenticated connections, and outbound connections to AI providers use TLS encryption.

---

## 9. Troubleshooting

### 9.1 Common Issues

| Issue | Cause | Resolution |
|---|---|---|
| Gateway fails to start | Node.js version mismatch | Verify Node.js 22.14+ or 24 is installed |
| Channel not connecting | Invalid credentials | Check API keys and tokens in configuration |
| Agent not responding | Model provider error | Verify provider API key and model availability |
| High memory usage | Too many concurrent sessions | Configure session TTL and garbage collection |
| Webhook failures | Incorrect URL or firewall | Verify webhook URL accessibility and verification token |
| Plugin load error | Incompatible plugin version | Update plugin or check compatibility matrix |

### 9.2 Diagnostic Commands

```bash
# Check OpenClaw status and health
openclaw status

# View real-time logs
openclaw logs --follow

# Test channel connectivity
openclaw test channel <channel-name>

# Verify configuration syntax
openclaw config validate
```

---

## 10. Advanced Topics

For advanced deployment patterns, custom plugin development, Lobster workflow pipelines, performance tuning, and scaling strategies, refer to the companion document **[01-openclaw-advanced.md](./01-openclaw-advanced.md)**.

---

## References

1. OpenClaw Official Documentation — https://docs.openclaw.ai/
2. OpenClaw GitHub Repository — https://github.com/openclaw/openclaw
3. OpenClaw Features — https://docs.openclaw.ai/concepts/features
4. OpenClaw Showcase — https://docs.openclaw.ai/start/showcase
5. Node.js Official Documentation — https://nodejs.org/docs/
