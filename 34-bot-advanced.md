# Bot Specialist: Advanced Bot Engineering with OpenClaw, NemoClaw, and OpenShell

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Advanced Plugin Development](#advanced-plugin-development)  
   1. [Plugin Types: Native vs Bundle](#plugin-types-native-vs-bundle)  
   2. [Plugin Lifecycle Hooks](#plugin-lifecycle-hooks)  
   3. [Distribution and Management via ClawHub](#distribution-and-management-via-clawhub)  
   4. [Example: Creating a Native Plugin](#example-creating-a-native-plugin)  
3. [Advanced Memory System](#advanced-memory-system)  
   1. [Memory Engines: QMD, Honcho, LanceDB](#memory-engines-qmd-honcho-lancedb)  
   2. [Dreaming and Grounded Backfill](#dreaming-and-grounded-backfill)  
   3. [Memory Wiki Plugin](#memory-wiki-plugin)  
   4. [Memory Tools and CLI](#memory-tools-and-cli)  
4. [Custom Network Policies and Egress Control](#custom-network-policies-and-egress-control)  
   1. [Network Policy Architecture](#network-policy-architecture)  
   2. [Official and Community Policy Presets](#official-and-community-policy-presets)  
   3. [Policy Configuration Examples](#policy-configuration-examples)  
   4. [Egress Control and Operator Approval Flow](#egress-control-and-operator-approval-flow)  
5. [The Awesome OpenClaw Skills Ecosystem](#the-awesome-openclaw-skills-ecosystem)  
   1. [Scale and Diversity of Skills](#scale-and-diversity-of-skills)  
   2. [Skill Installation Methods](#skill-installation-methods)  
   3. [Skill Categories and Use Cases](#skill-categories-and-use-cases)  
   4. [Security Vetting and Risk Mitigation](#security-vetting-and-risk-mitigation)  
6. [Agent Recipes and Deployment Templates](#agent-recipes-and-deployment-templates)  
   1. [Remote GPU Assistant Recipe](#remote-gpu-assistant-recipe)  
   2. [Approval-First Web Agent](#approval-first-web-agent)  
   3. [Telegram Bridge Sandbox Agent](#telegram-bridge-sandbox-agent)  
   4. [Runtime Model-Switching Workflow](#runtime-model-switching-workflow)  
7. [Conclusion](#conclusion)  

---

## Introduction

This document serves as an advanced technical guide for bot specialists focusing on the engineering and deployment of autonomous AI agents using the OpenClaw gateway, NemoClaw sandbox platform, and OpenShell runtime environment. These technologies collectively enable secure, scalable, and extensible AI assistants with rich plugin ecosystems, sophisticated memory management, fine-grained network policy enforcement, and a vast repository of community-built skills.

The following sections provide deep technical insights, configuration examples, architectural explanations, and expert best practices for mastering advanced bot engineering with this stack.

---

## Advanced Plugin Development

Plugins are the extensibility backbone of OpenClaw and NemoClaw agents, enabling custom behaviors, integration with external services, and enhancement of agent capabilities. Understanding the plugin architecture, lifecycle, and distribution mechanisms is critical for building robust, maintainable, and secure AI assistants.

### Plugin Types: Native vs Bundle

OpenClaw supports two primary plugin formats, each with distinct characteristics and use cases:

| Aspect               | Native Plugins                                  | Bundle Plugins                             |
|----------------------|------------------------------------------------|--------------------------------------------|
| Format               | `openclaw.plugin.json` + runtime module (JS/TS) | Codex/Claude/Cursor-compatible directory layout |
| Execution Environment | In-process within OpenClaw runtime              | External or embedded, mapped to OpenClaw features |
| Development Complexity | Requires TypeScript/JavaScript coding expertise | Can be authored with minimal coding, often declarative |
| Capabilities         | Full access to OpenClaw APIs, lifecycle hooks, tools | Limited to defined extension points (skills, channels, etc.) |
| Distribution         | Packaged as npm modules or local paths          | Distributed as bundles via ClawHub or Git |
| Use Cases            | Complex integrations, custom inference, tool orchestration | Rapid skill development, channel adapters, simple tools |

Native plugins are ideal for advanced developers who need to implement complex logic, custom inference providers, or tightly integrated toolchains. Bundle plugins are better suited for rapid prototyping, skill sharing, and community contributions.

### Plugin Lifecycle Hooks

OpenClaw exposes a comprehensive set of lifecycle hooks that plugins can implement to intercept, modify, or extend agent behavior at various stages. These hooks enable fine-grained control over the agent's operation and integration with external systems.

| Hook Name            | Description                                                                                      | Typical Use Cases                                      |
|----------------------|------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| `before_model_resolve` | Override or select model/provider before inference resolution                                  | Dynamic model switching, custom routing               |
| `before_prompt_build`  | Modify or inject context/system prompts before sending to LLM                                  | Context enrichment, prompt engineering                 |
| `before_agent_start`   | Initialization logic before the agent run begins                                              | Setup, resource allocation                             |
| `before_agent_reply`   | Intercept or synthesize agent replies                                                        | Custom reply generation, filtering                     |
| `agent_end`            | Inspect or modify final message list after agent completes run                               | Logging, analytics, cleanup                            |
| `before_compaction` / `after_compaction` | Hooks around memory compaction phases                                              | Memory management, context optimization                |
| `before_tool_call` / `after_tool_call` | Intercept tool invocation and results                                              | Tool result transformation, validation                  |
| `before_install`       | Block or approve skill/plugin installation                                                    | Security enforcement, policy compliance                |
| `tool_result_persist`  | Transform tool results before transcript persistence                                          | Data sanitization, formatting                           |
| `message_received` / `message_sending` / `message_sent` | Hooks around message lifecycle events                                         | Message filtering, analytics                            |
| `session_start` / `session_end` | Lifecycle hooks for session management                                                | Session initialization, cleanup                         |
| `gateway_start` / `gateway_stop` | Hooks for gateway lifecycle events                                                  | Resource management, monitoring                         |

These hooks can be implemented as asynchronous functions within native plugins or declarative scripts in bundles, depending on the plugin type.

### Distribution and Management via ClawHub

ClawHub is the primary distribution platform for OpenClaw plugins and skills, hosting over 13,700 community-built skills as of early 2026. It provides a centralized registry, versioning, security scanning, and dependency management.

**Key Features:**

- **CLI Integration:** Install, update, list, and uninstall plugins using the `clawhub` CLI tool.  
- **Chat-based Installation:** Paste GitHub links or use `/plugin install clawhub:<package>` commands in chat interfaces for seamless skill deployment.  
- **Security Vetting:** Skills undergo VirusTotal scanning and Snyk vulnerability analysis to mitigate risks such as prompt injection or tool poisoning.  
- **Priority Resolution:** Workspace skills override local and bundled skills, enabling project-specific customization.

**Example CLI Commands:**

```bash
# Install a skill from ClawHub
clawhub install advanced-code-analyzer

# List installed skills
clawhub list

# Update all installed skills
clawhub update

# Uninstall a skill
clawhub uninstall advanced-code-analyzer
```

### Example: Creating a Native Plugin

Below is a simplified example of a native plugin that logs every agent reply and appends a custom signature.

```typescript
// src/index.ts
import { Plugin } from 'openclaw-plugin-sdk';

const MyLoggingPlugin: Plugin = {
  name: 'my-logging-plugin',
  version: '1.0.0',

  async before_agent_reply(context, reply) {
    console.log(`[Agent Reply] Session: ${context.sessionId}, Reply: ${reply.text}`);
    // Append a signature to the reply
    reply.text += "\n\n-- Powered by MyLoggingPlugin";
    return reply;
  },

  async agent_end(context, messages) {
    // Summarize the session messages for analytics
    const summary = messages.map(m => m.text).join('\n---\n');
    await context.log(summary);
  }
};

export default MyLoggingPlugin;
```

**Packaging and Installation:**

- Package as an npm module with `openclaw.plugin.json` manifest.  
- Publish to npm or install locally via path.  
- Register with OpenClaw CLI: `openclaw plugins install ./my-logging-plugin`.

---

## Advanced Memory System

Memory management is a cornerstone of autonomous agent intelligence in OpenClaw. The system supports multiple memory engines, background consolidation ("dreaming"), and a structured knowledge base ("Memory Wiki") to enable persistent, context-aware interactions.

### Memory Engines: QMD, Honcho, LanceDB

OpenClaw supports several memory backends, each optimized for different use cases:

| Engine   | Description                                                                                      | Strengths                                  | Use Cases                              |
|----------|------------------------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------|
| **Builtin** | SQLite-based default memory engine integrated into OpenClaw core                              | Simplicity, local-first, reliable          | General purpose, small to medium scale |
| **QMD**    | Advanced local-first sidecar engine with enhanced indexing and concurrency                     | High performance, local-first, scalable    | Large-scale local memory, fast queries |
| **Honcho** | AI-native cross-session memory engine designed for multi-agent environments                    | Cross-session consistency, AI-native indexing | Multi-agent coordination, shared memory |
| **LanceDB**| Vector database with OpenAI-compatible embeddings for semantic search and retrieval            | Semantic search, cloud integration          | Embedding-based retrieval, hybrid search |

Each engine supports semantic and keyword hybrid search, enabling agents to retrieve relevant context efficiently.

### Dreaming and Grounded Backfill

"Dreaming" is a background consolidation process that refines and promotes candidate memory entries into long-term memory. It operates on scheduled intervals (via cron jobs) and applies multiple filters:

- **Score Threshold:** Only high-confidence candidates are promoted.  
- **Recall Frequency:** Items frequently recalled are prioritized.  
- **Query Diversity Gates:** Ensures a diverse set of memories is retained.  
- **Grounded Backfill:** Fills gaps in historical notes with grounded evidence.

Dreaming outputs summaries to `DREAMS.md` for human review and integrates qualified items into `MEMORY.md`.

This process improves memory quality over time, reduces noise, and enhances agent contextual awareness.

### Memory Wiki Plugin

The Memory Wiki plugin compiles durable knowledge into a structured vault with the following features:

- **Claims and Evidence:** Structured assertions with supporting data.  
- **Contradiction Tracking:** Detects conflicting information for review.  
- **Freshness Tracking:** Flags stale or outdated knowledge.  
- **Dashboards:** Generated views for knowledge management.

The plugin exposes tools such as `wiki_search`, `wiki_get`, `wiki_apply`, and `wiki_lint` for interactive knowledge base management.

### Memory Tools and CLI

OpenClaw provides CLI tools for memory inspection and management:

```bash
# Check memory index status
openclaw memory status

# Search memory with a query
openclaw memory search "Explain Kubernetes pod lifecycle"

# Force rebuild memory index
openclaw memory index --force
```

Memory files are organized as:

- `MEMORY.md` — Long-term memory loaded at session start.  
- `memory/YYYY-MM-DD.md` — Daily notes auto-loaded for current and previous day.  
- `DREAMS.md` — Dreaming summaries for human review.

---

## Custom Network Policies and Egress Control

Network security and controlled egress are vital for sandboxed AI agents to prevent data leakage, unauthorized access, and compliance violations. NemoClaw and OpenShell implement layered network policy enforcement with operator approval workflows.

### Network Policy Architecture

Network policies are declarative YAML files applied per sandbox and enforced by the OpenShell gateway. They define allowed outbound hosts, HTTP methods, and URL paths with fine granularity.

The policy engine supports:

- **Outbound Host Whitelisting:** Restrict outbound connections to approved domains.  
- **HTTP Method and Path Filtering:** Allow only specific HTTP verbs and URL patterns.  
- **Hot Reloading:** Dynamic policy updates without sandbox restarts.  
- **Operator Approval Flow:** Unknown hosts or suspicious requests trigger operator approval workflows.

### Official and Community Policy Presets

NVIDIA maintains a set of official policy presets for common services:

| Preset Name | Description                      |
|-------------|---------------------------------|
| discord     | Discord API access               |
| docker      | Docker Hub access                |
| huggingface | Hugging Face API access          |
| jira        | Atlassian Jira API access        |
| npm         | NPM registry access              |
| outlook     | Microsoft Outlook API access     |
| pypi        | Python Package Index access      |
| slack       | Slack API access                 |
| telegram    | Telegram Bot API access          |

Community-contributed presets extend coverage to:

| Preset Name      | Description                          |
|------------------|-------------------------------------|
| gitlab           | GitLab API access                   |
| notion           | Notion API access                   |
| linear           | Linear.app API access               |
| confluence       | Atlassian Confluence API access    |
| teams            | Microsoft Teams API access          |
| zendesk          | Zendesk API access                  |
| sentry           | Sentry API access                   |
| stripe           | Stripe payment API access           |
| cloudflare       | Cloudflare API access               |
| google-workspace | Google Workspace API access          |
| aws              | Amazon Web Services API access      |
| gcp              | Google Cloud Platform API access    |
| vercel           | Vercel deployment API access        |
| supabase         | Supabase API access                 |
| neon             | Neon database API access             |
| algolia          | Algolia search API access            |
| airtable         | Airtable API access                 |
| hubspot          | HubSpot API access                  |

These presets provide vetted, secure baseline configurations for sandbox network policies.

### Policy Configuration Examples

A minimal read-only GitHub API access policy example:

```yaml
network:
  outbound:
    - host: api.github.com
      methods: [GET]
      paths: ["/**"]
```

A more complex policy allowing Slack and Discord outbound access with method restrictions:

```yaml
network:
  outbound:
    - host: slack.com
      methods: [POST, GET]
      paths: ["/api/**"]
    - host: discord.com
      methods: [POST]
      paths: ["/api/v*/channels/*/messages"]
```

Policies can be applied or updated dynamically using:

```bash
openshell policy set my-sandbox --policy ./policy.yaml
```

### Egress Control and Operator Approval Flow

NemoClaw implements an operator approval flow for egress requests to unknown or unapproved hosts. When an agent attempts to access a host not covered by the active policy, the request is intercepted and queued for operator review.

Operators receive notifications and can approve or deny access, triggering policy updates or request blocking. This mechanism enforces a zero-trust posture and mitigates risks from rogue or compromised agents.

---

## The Awesome OpenClaw Skills Ecosystem

The OpenClaw ecosystem boasts over 5,200 curated skills (from a total of 13,700+ community submissions), spanning diverse categories and use cases. Skills are modular extensions that augment agent capabilities, ranging from coding assistants to productivity enhancers.

### Scale and Diversity of Skills

| Category                 | Skill Count | Description                                    |
|--------------------------|-------------|------------------------------------------------|
| Coding Agents & IDEs      | 1,184       | Code completion, linting, debugging, refactoring |
| Web & Frontend Development | 919         | HTML/CSS/JS tooling, browser automation         |
| DevOps & Cloud           | 393         | Kubernetes, Docker, CI/CD pipelines             |
| Search & Research        | 345         | Web search, academic research, data scraping   |
| Browser & Automation     | 323         | Browser control, web scraping, task automation |
| Productivity & Tasks     | 205         | Task management, calendar integration           |
| AI & LLMs                | 176         | Model orchestration, prompt engineering          |
| CLI Utilities            | 180         | Shell tools, process management                   |
| Image & Video Generation | 170         | Image synthesis, video editing                    |
| Git & GitHub             | 167         | Git operations, pull request management          |
| Communication            | 146         | Chatbots, messaging integration                   |
| Transportation           | 110         | Ride sharing, logistics                            |
| PDF & Documents          | 105         | Document parsing, PDF generation                   |
| Marketing & Sales        | 103         | CRM integration, lead generation                   |
| Health & Fitness         | 87          | Wellness tracking, fitness coaching                 |
| Media & Streaming        | 86          | Media playback, streaming control                   |
| Notes & PKM             | 69          | Personal knowledge management                        |
| Calendar & Scheduling    | 65          | Scheduling assistants, reminders                    |
| Security & Passwords     | 54          | Password management, security auditing               |
| Shopping & E-commerce    | 51          | Online shopping, price tracking                       |
| Personal Development     | 50          | Self-improvement tools, coaching                      |
| Speech & Transcription   | 45          | Voice recognition, transcription                      |
| Apple Apps & Services    | 44          | Apple ecosystem integration                            |
| Smart Home & IoT         | 41          | Home automation, IoT device control                    |
| Gaming                   | 35          | Game bots, stats tracking                              |
| Self-Hosted & Automation | 33          | Self-hosted tools, automation workflows                |

### Skill Installation Methods

Skills can be installed and managed via multiple methods:

1. **ClawHub CLI:** The primary method for skill installation and updates.  
   ```bash
   clawhub install <skill-slug>
   ```
2. **Manual Installation:** Copy skill directories into the global `~/.openclaw/skills/` or workspace-specific `./skills/` folder.  
3. **Chat Interface:** Paste GitHub repository links or use chat commands like `/plugin install clawhub:<package>` to trigger automatic installation.

**Priority Resolution:** Skills in the workspace override local user skills, which in turn override bundled skills. This allows project-specific customization without affecting global installations.

### Security Vetting and Risk Mitigation

Given the open nature of the skill ecosystem, security is paramount. OpenClaw employs multiple layers of vetting:

- **VirusTotal Scanning:** Automated malware scanning of skill packages.  
- **Snyk Skill Security Scanner:** Static analysis for vulnerabilities and malicious code.  
- **Agent Trust Hub:** Centralized trust management for skill provenance and reputation.  
- **Runtime Sandboxing:** Skills execute within sandboxed environments with strict filesystem, network, and process policies to prevent unauthorized access or damage.  
- **Prompt Injection and Tool Poisoning Detection:** Heuristics and manual reviews to detect malicious prompt or tool manipulations.

Administrators are encouraged to maintain allowlists and denylists for skills and tools, leveraging OpenClaw's policy system to enforce security boundaries.

---

## Agent Recipes and Deployment Templates

NemoClaw and OpenClaw provide reusable agent recipes and deployment templates to accelerate common use cases and best practices.

### Remote GPU Assistant Recipe

This recipe enables a persistent remote sandboxed assistant with GPU acceleration:

- **Prerequisites:** NVIDIA GPU on host, NVIDIA drivers, NVIDIA Container Toolkit.  
- **Deployment:** Use `nemoclaw onboard` with GPU flags and OpenShell sandbox creation with `--gpu` option.  
- **Persistent Port Forwarding:** Configure systemd services with health-check keepalive scripts to maintain web UI access on port 18789.  
- **Remote Access:** Optional Cloudflare Tunnel for secure external access.

**Example Commands:**

```bash
# Setup GPU-enabled sandbox
openshell sandbox create --gpu -- claude

# Start persistent port forward
openshell forward start 18789 claude

# Onboard NemoClaw with GPU support
nemoclaw onboard
```

### Approval-First Web Agent

This agent enforces operator approval for unknown hosts:

- **Network Policy:** Default deny for unknown hosts with operator approval flow enabled.  
- **Operator Notifications:** Requests to unapproved hosts trigger alerts for manual approval.  
- **Policy Updates:** Approved hosts are added to network policies dynamically.

**Sample Network Policy:**

```yaml
network:
  outbound:
    - host: approved-host.com
      methods: [GET, POST]
      paths: ["/api/**"]
  approval_required: true
```

### Telegram Bridge Sandbox Agent

A sandboxed agent that acts as a Telegram bot bridge:

- **Sandbox Container:** Runs OpenClaw agent with NemoClaw plugin inside Landlock-secured container.  
- **Channel Messaging:** OpenShell-managed Telegram channel integration.  
- **Network Policy:** Restricts outbound connections to Telegram API endpoints.  
- **Deployment:** Uses official telegram policy preset and custom skill for message forwarding.

**Deployment Snippet:**

```bash
nemoclaw onboard
openshell policy set telegram-agent --policy telegram-policy.yaml
```

### Runtime Model-Switching Workflow

Allows switching inference models without restarting the agent:

- **Model Router:** Uses NVIDIA LLM Router v3 to route queries based on complexity and cost.  
- **Configuration:** Modify `pool-config.yaml` to add or remove models.  
- **Hot Reload:** Update inference provider settings dynamically with `openshell inference set`.  
- **Benefits:** Cost optimization, latency tuning, and fallback strategies.

**Sample Model Pool Config:**

```yaml
routing:
  method: prefill
  checkpoint: llm-router/checkpoints/prefill_router_qwen08b.pt
  tolerance: 0.20
  encoder: Qwen/Qwen3.5-0.8B
models:
  - name: nano
    litellm_model: "openai/nvidia/nvidia/Nemotron-3-Nano-30B-A3B"
    cost_per_m_input_tokens: 0.05
  - name: super
    litellm_model: "openai/nvidia/nvidia/nemotron-3-super-v3"
    cost_per_m_input_tokens: 0.10
```

---

## Conclusion

Mastering advanced bot engineering with OpenClaw, NemoClaw, and OpenShell requires a deep understanding of plugin development, memory management, network security, skill ecosystems, and deployment best practices. This document has provided a comprehensive technical foundation, including architecture insights, configuration examples, and expert recommendations.

By leveraging the layered security model, rich plugin lifecycle, sophisticated memory systems, and vast community skill repository, bot specialists can build autonomous AI agents that are powerful, secure, and adaptable to complex real-world scenarios.

---

# Appendix

### References

- [NemoClaw GitHub Repository](https://github.com/NVIDIA/NemoClaw)  
- [OpenShell GitHub Repository](https://github.com/NVIDIA/OpenShell)  
- [OpenClaw Documentation](https://docs.openclaw.ai)  
- [VoltAgent Awesome NemoClaw](https://github.com/VoltAgent/awesome-nemoclaw)  
- [VoltAgent Awesome OpenClaw Skills](https://github.com/VoltAgent/awesome-openclaw-skills)  

---

# End of Document: 34-bot-advanced.md