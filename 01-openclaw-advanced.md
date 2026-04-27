# OpenClaw Advanced Guide

> **Role:** OpenClaw Platform Specialist — Advanced Patterns
> **Domain:** Plugin development, security hardening, scaling, remote access, formal verification
> **Official Documentation:** [docs.openclaw.ai](https://docs.openclaw.ai/)

---

## 1. Advanced Plugin Development

### 1.1 Plugin Architecture Deep Dive

OpenClaw plugins are packages that register any combination of channels, model providers, tools, skills, and speech capabilities. The plugin system follows a lifecycle-aware architecture where plugins are loaded at Gateway startup and can hook into every phase of message processing and agent execution. Plugins are the primary mechanism for extending OpenClaw beyond its built-in capabilities [1].

A plugin must export a registration function that receives the Gateway context and registers its capabilities. The registration function has access to the full Gateway API, including channel management, session storage, event emission, and configuration access. Plugins can declare dependencies on other plugins, and the Gateway resolves the dependency graph at startup to ensure correct loading order.

### 1.2 Custom Channel Adapter Development

Creating a custom channel adapter requires implementing the Channel interface, which defines methods for connecting to the external service, receiving messages, sending responses, and handling media attachments. The adapter must normalize incoming messages into OpenClaw's internal message format and convert outgoing messages back into the platform-native format [1].

Key considerations for channel adapter development include connection lifecycle management (reconnection logic, backoff strategies), message deduplication (preventing duplicate processing of the same message), rate limiting (respecting platform API limits), and media transcoding (converting between platform-specific media formats).

### 1.3 Custom Tool Development

Custom tools extend the agent's capabilities by providing new actions it can perform. Each tool must define a JSON Schema for its input parameters, implement the execution logic, and return structured output. Tools can be synchronous or asynchronous, and the agent loop handles both patterns transparently [1].

```typescript
// Example custom tool registration
export function register(gateway) {
  gateway.registerTool({
    name: 'database_query',
    description: 'Execute a read-only SQL query against the analytics database',
    inputSchema: {
      type: 'object',
      properties: {
        query: { type: 'string', description: 'SQL SELECT query' },
        database: { type: 'string', enum: ['analytics', 'reporting'] }
      },
      required: ['query']
    },
    execute: async ({ query, database }) => {
      if (!/^\s*SELECT/i.test(query)) {
        throw new Error('Only SELECT queries are allowed');
      }
      const results = await executeQuery(database || 'analytics', query);
      return { rows: results.rows, rowCount: results.rowCount };
    }
  });
}
```

### 1.4 Skill Authoring Best Practices

Skills are markdown files (`SKILL.md`) injected into the system prompt to shape agent behavior. Advanced skill authoring involves structuring the markdown to maximize the model's adherence to instructions. Best practices include using clear section headers, providing explicit examples of desired behavior, defining error handling procedures, and specifying output formats. Skills should be modular and composable, allowing operators to combine multiple skills for different use cases [1].

---

## 2. Security Hardening and Threat Model

### 2.1 Authentication Layers

OpenClaw implements multiple authentication layers that can be combined for defense in depth [2]:

| Layer | Mechanism | Use Case |
|-------|-----------|----------|
| **Token Auth** | Shared secret in `Authorization` header | API clients, automation |
| **Password Auth** | Password-based WebSocket connect | Human operators |
| **Pairing** | Device identity + operator approval | New device onboarding |
| **Trusted Proxy** | Header-based identity from reverse proxy | Behind nginx/Caddy/Traefik |
| **Tailscale** | WireGuard-based mesh VPN | Zero-config remote access |

### 2.2 Network Security

The Gateway defaults to binding on loopback (`127.0.0.1`), preventing external network access unless explicitly configured. For production deployments, the recommended approach is to keep the loopback binding and use a reverse proxy (nginx, Caddy, or Traefik) with TLS termination for external access. Alternatively, Tailscale integration provides zero-config encrypted remote access without exposing ports to the public internet [2].

### 2.3 MITRE ATLAS Threat Model

OpenClaw's documentation includes a formal threat model based on the MITRE ATLAS framework, which maps adversarial techniques specific to AI/ML systems. The threat model covers prompt injection attacks (direct and indirect), model manipulation, data poisoning through tool results, session hijacking, and privilege escalation through tool abuse. Each threat is mapped to specific mitigations implemented in the Gateway [2].

Key mitigations include tool allow/deny lists that restrict which tools an agent can use, session isolation that prevents cross-session data leakage, input validation at the Gateway level before messages reach the agent, and output filtering that can detect and block sensitive data in responses.

### 2.4 Formal Verification

OpenClaw includes formal verification models for its security properties. These models mathematically prove that certain security invariants hold under all possible execution paths. The formal verification covers properties such as session isolation (no information flow between sessions), tool access control (deny list always overrides allow list), and authentication enforcement (no unauthenticated access to protected endpoints) [2].

### 2.5 Sandboxing and Process Isolation

For deployments requiring additional security, OpenClaw supports running agents inside sandboxed environments. The sandbox isolates the agent's filesystem, network access, and process capabilities. This is particularly important when agents have access to tools like `exec` that can execute arbitrary commands. The sandbox configuration supports Landlock (filesystem restrictions), seccomp (system call filtering), and network namespace isolation [2].

---

## 3. Scaling and High Availability

### 3.1 Single-Instance Optimization

The primary scaling strategy for OpenClaw is vertical scaling of the single Gateway instance. The Gateway is designed to handle hundreds of concurrent sessions on modest hardware. Key optimization parameters include session compaction frequency (reducing memory usage for long conversations), tool timeout configuration (preventing runaway tool executions), and model provider connection pooling [2].

### 3.2 Resource Requirements

| Deployment Size | CPU | RAM | Disk | Concurrent Sessions |
|----------------|-----|-----|------|-------------------|
| Personal (1-5 users) | 2 cores | 4 GB | 10 GB | 10-20 |
| Team (5-20 users) | 4 cores | 8 GB | 20 GB | 50-100 |
| Organization (20-100 users) | 8 cores | 16 GB | 50 GB | 200-500 |

### 3.3 Multi-Gateway Topology

For organizations requiring horizontal scaling or geographic distribution, OpenClaw supports multi-gateway topologies where each Gateway instance handles a subset of channels or users. Session state can be shared through external storage backends, and a load balancer distributes incoming connections across Gateway instances. However, the constraint that only one Gateway per host can maintain channel sessions (especially WhatsApp via Baileys) must be carefully managed [2].

### 3.4 Monitoring and Observability

The Gateway emits health, heartbeat, and presence events that can be consumed by monitoring systems. The `openclaw gateway status --json` command provides machine-readable status output suitable for integration with Prometheus, Grafana, or custom monitoring dashboards. Key metrics to monitor include active session count, agent run duration, tool execution latency, model provider response times, and channel connection status [2].

---

## 4. Remote Access and Deployment Patterns

### 4.1 Tailscale Integration

OpenClaw provides first-class integration with Tailscale for zero-config remote access. Tailscale creates a WireGuard-based mesh VPN that connects all devices in a tailnet, enabling secure access to the Gateway from anywhere without exposing ports to the public internet. The integration supports Tailscale Serve for automatic HTTPS certificate provisioning and Tailscale Funnel for public access when needed [2].

### 4.2 Reverse Proxy Configuration

For production deployments behind a reverse proxy, OpenClaw supports trusted-proxy authentication where the proxy forwards authenticated user identity via headers. The Gateway validates these headers and maps them to internal user identities. This pattern is commonly used with nginx, Caddy, or Traefik in containerized deployments [2].

### 4.3 Remote GPU Deployment

For agents that require GPU-accelerated inference (e.g., running local models via Ollama or vLLM), OpenClaw supports deployment on remote GPU instances. The Gateway can be configured to route inference requests to a remote GPU server while maintaining the channel connections on a lightweight frontend instance. This separation allows cost-effective scaling where GPU resources are only used for inference [2].

### 4.4 Containerized Deployment

OpenClaw can be deployed in Docker containers for consistent, reproducible deployments. The containerized deployment includes the Gateway process, all configured channel adapters, and the tool runtime. Persistent volumes are used for session storage and configuration. Docker Compose templates are available for common deployment patterns including single-instance, multi-gateway, and GPU-accelerated configurations [2].

---

## 5. Advanced Agent Patterns

### 5.1 Sub-Agent Orchestration

OpenClaw supports sub-agent orchestration where a primary agent can spawn and coordinate secondary agents for specialized tasks. The `subagents` tool allows the primary agent to create sub-agents with specific configurations, delegate tasks, and aggregate results. This pattern is useful for complex workflows that require different expertise or tool access levels [3].

### 5.2 Session Model Override

The `session_status` tool provides a lightweight mechanism for agents to override the model used for a specific session. This enables dynamic model selection based on task complexity — for example, using a faster, cheaper model for simple queries and switching to a more capable model for complex reasoning tasks [3].

### 5.3 Cross-Channel Messaging

The `message` tool enables agents to send messages across any connected channel, not just the channel where the conversation originated. This enables patterns like receiving a request via Telegram and posting the result to a Slack channel, or broadcasting notifications across multiple platforms simultaneously [1].

### 5.4 Cron-Driven Automation

The `cron` tool enables agents to schedule recurring tasks. Combined with the `gateway` tool (which can inspect and modify Gateway configuration), agents can implement self-managing automation workflows. For example, an agent could schedule a daily code review, monitor system health metrics, or generate periodic reports [1].

### 5.5 Node Integration

Nodes (macOS, iOS, Android, headless) extend the agent's reach to physical devices. Through the `nodes` tool, agents can discover paired devices and execute device-specific commands such as `canvas.*` (present content, evaluate JavaScript, take snapshots), `camera.*` (capture photos/video), `screen.record` (record screen), and `location.get` (retrieve device location). This enables powerful IoT and automation scenarios [2].

---

## 6. Configuration Reference

### 6.1 Core Configuration Structure

The configuration file at `~/.openclaw/openclaw.json` follows a hierarchical structure:

```
openclaw.json
├── agents
│   ├── defaults
│   │   ├── model.primary
│   │   ├── model.fallback
│   │   └── tools
│   └── <agentId>
│       ├── provider
│       ├── model
│       ├── systemPrompt
│       ├── tools
│       └── skills
├── channels
│   └── <channelName>
│       ├── enabled
│       └── <channel-specific config>
├── providers
│   └── <providerName>
│       └── apiKey
├── tools
│   ├── allow
│   ├── deny
│   └── byProvider
├── gateway
│   ├── port (default: 18789)
│   ├── bind (default: 127.0.0.1)
│   ├── auth
│   └── reload (off|hot|restart|hybrid)
└── sessions
    ├── storage
    ├── compaction
    └── ttl
```

### 6.2 Environment Variables

OpenClaw supports environment variable overrides for sensitive configuration values. Provider API keys, channel tokens, and authentication secrets can all be specified via environment variables, keeping them out of the configuration file. The naming convention follows `OPENCLAW_<SECTION>_<KEY>` format [1].

---

## 7. Troubleshooting Guide

### 7.1 Common Issues and Resolutions

| Issue | Diagnosis | Resolution |
|-------|-----------|------------|
| Gateway fails to start | Check `openclaw doctor` output | Verify Node.js version, port availability, config syntax |
| Channel not connecting | Check `openclaw gateway status --deep` | Verify credentials, network access, platform API status |
| Agent not responding | Check agent logs via `openclaw logs --follow` | Verify model provider API key, check rate limits |
| Tool execution failing | Check tool-specific error in agent events | Verify tool permissions in allow/deny lists |
| Session state lost | Check session storage backend | Verify disk space, file permissions |
| High memory usage | Check active session count | Configure session TTL, increase compaction frequency |
| WebSocket disconnections | Check network stability | Configure reconnection backoff, check proxy timeouts |
| Pairing rejected | Check device identity in logs | Re-pair device, verify signature version |
| Hot reload not applying | Check reload mode in config | Ensure `hybrid` or `hot` mode is set |
| VoiceClaw not connecting | Check `/voiceclaw/realtime` endpoint | Verify Gemini Live credentials, WebSocket path |

### 7.2 Log Analysis

The `openclaw logs --follow` command streams real-time logs from the Gateway. Logs are structured JSON, making them suitable for parsing with tools like `jq`. Key log fields include `level` (error, warn, info, debug), `component` (gateway, channel, agent, tool), `sessionId`, `runId`, and `message` [2].

### 7.3 Performance Profiling

For performance issues, the Gateway supports Node.js profiling through the `--inspect` flag. This enables connection with Chrome DevTools for CPU profiling, heap snapshots, and event loop analysis. Common performance bottlenecks include excessive session history (resolved by compaction), synchronous tool executions blocking the event loop, and memory leaks in long-running sessions [2].

---

## 8. Production Deployment Checklist

| Item | Status | Notes |
|------|--------|-------|
| Node.js version verified (22.14+ or 24) | ☐ | Use `node -v` to confirm |
| Daemon installed and auto-start configured | ☐ | `openclaw onboard --install-daemon` |
| All API keys stored in environment variables | ☐ | Never hardcode in config |
| Allowlists configured for all channels | ☐ | Restrict access to authorized users |
| SSL/TLS enabled for webhooks | ☐ | Use reverse proxy or Tailscale |
| Session compaction configured | ☐ | Prevent unbounded memory growth |
| Tool allow/deny lists configured | ☐ | Principle of least privilege |
| Audit logging enabled | ☐ | Ship to centralized logging |
| Content policies defined | ☐ | Enforce compliance at gateway level |
| Monitoring and alerting configured | ☐ | Health checks, error rates, latency |
| Backup strategy for configuration and sessions | ☐ | Regular automated backups |
| Hot reload mode set to `hybrid` | ☐ | Safe default for production |
| Pairing approval workflow documented | ☐ | Ensure only authorized devices connect |
| MITRE ATLAS mitigations reviewed | ☐ | Address all identified threat vectors |

---

## References

[1]: [OpenClaw Documentation](https://docs.openclaw.ai/)
[2]: [OpenClaw Gateway & Ops](https://docs.openclaw.ai/gateway)
[3]: [OpenClaw Agent Loop](https://docs.openclaw.ai/concepts/agent-loop)
