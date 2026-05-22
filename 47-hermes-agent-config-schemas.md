# Hermes Agent Configuration Schemas and Reference

## 1. Introduction to Hermes Agent Configuration

The Hermes Agent (version 0.14.0), developed by NousResearch, is a highly configurable, self-improving AI agent designed to operate across a multitude of environments. Its configuration system is designed to be robust, flexible, and secure, separating sensitive credentials from operational settings. This document serves as the definitive reference for configuring Hermes Agent in production environments, detailing every aspect of its configuration schemas, environment variables, and context files.

Hermes Agent utilizes a dual-configuration approach to ensure security and flexibility. Operational settings, such as model preferences, terminal backends, and toolset configurations, are stored in a structured YAML file (`config.yaml`). Sensitive information, including API keys, database passwords, and cryptographic secrets, are strictly managed through environment variables, typically loaded from a `.env` file. This separation of concerns aligns with twelve-factor app methodology and prevents accidental leakage of credentials in version control systems.

The configuration files are typically located in the `~/.hermes/` directory, which serves as the central hub for the agent's state, memories, and settings. Understanding the interplay between `config.yaml`, `.env`, and the various context files (like `SOUL.md` and `AGENTS.md`) is crucial for deploying Hermes Agent effectively. This guide will walk you through each configuration component, providing real-world examples, schema definitions, and best practices for production deployments.

Whether you are configuring a local instance for personal use, deploying a persistent Docker container for development, or setting up a multi-agent Kanban board orchestrated via a messaging gateway, this reference provides the necessary details to tailor Hermes Agent to your specific requirements.

## 2. Complete `config.yaml` Schema

The `config.yaml` file is the primary configuration file for Hermes Agent. It defines the operational parameters for the agent, including model selection, terminal backend settings, memory limits, and gateway preferences. The file is structured into logical sections, each governing a specific subsystem of the agent.

Below is a comprehensive example of a production-ready `config.yaml` file, illustrating the complete schema and all available sections:

```yaml
# ~/.hermes/config.yaml

version: "1.0"

model:
  provider: "openai"
  model_name: "gpt-4o"
  context_length: 128000
  ollama_num_ctx: 65536
  temperature: 0.7
  max_tokens: 4096
  top_p: 0.95
  frequency_penalty: 0.0
  presence_penalty: 0.0

terminal:
  backend: "docker"
  cwd: "/workspace"
  timeout: 300
  docker_image: "ubuntu:22.04"
  docker_mount: "/home/user/projects:/workspace"
  docker_volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
  docker_extra_args: "--cap-drop ALL --security-opt no-new-privileges"
  container_cpu: "2.0"
  container_memory: "4g"
  container_disk: "20g"
  container_persistent: true
  ssh_host: "dev-server.internal"
  ssh_user: "ubuntu"
  ssh_key: "~/.ssh/id_rsa"
  ssh_port: 22

mcp_servers:
  - name: "github-mcp"
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_TOKEN: "${GITHUB_TOKEN}"
    tool_filter: ["create_issue", "list_repositories"]
  - name: "postgres-mcp"
    url: "http://localhost:8080/mcp"
    headers:
      Authorization: "Bearer ${MCP_SECRET}"

skills:
  paths:
    - "~/.hermes/skills"
    - "/opt/hermes/shared-skills"
  config:
    auto_update: true
    trust_level: "verified_only"
    fallback_for_toolsets: ["web", "terminal"]

memory:
  provider: "sqlite"
  user_char_limit: 1375
  agent_char_limit: 2200
  snapshot_retention: 7
  compression_threshold: 0.8

gateway:
  platforms: ["telegram", "discord", "slack"]
  home_channels:
    telegram: "-1001234567890"
    discord: "123456789012345678"
  session_reset_timeout: 3600
  delivery_preferences:
    chunk_size: 2000
    markdown_support: true

cron:
  jobs_file: "~/.hermes/cron/jobs.json"
  timezone: "UTC"
  max_concurrent_jobs: 5

browser:
  headless: true
  viewport_width: 1280
  viewport_height: 720
  user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
  timeout: 30000

toolset:
  enabled:
    - "web"
    - "terminal"
    - "browser"
    - "media"
    - "memory"
  disabled:
    - "computer_use"
```

This schema provides a declarative way to configure the agent's behavior. The `hermes config set <key> <value>` command can be used to programmatically update these values, ensuring that the configuration remains consistent and easily manageable.

## 3. Model Configuration

The `model` section of `config.yaml` dictates how Hermes Agent interacts with Large Language Models (LLMs). Hermes Agent supports over 30 providers, including OpenAI, Anthropic, OpenRouter, Ollama, and many others. The configuration must specify the provider, the specific model name, and various generation parameters.

### Provider and Model Selection

The `provider` key specifies the API backend to use. Valid values include `openai`, `anthropic`, `openrouter`, `ollama`, `gemini`, `xai`, `groq`, `together`, and more. The `model_name` key specifies the exact model identifier expected by the chosen provider.

```yaml
model:
  provider: "anthropic"
  model_name: "claude-3-5-sonnet-20241022"
```

### Context Length and Token Management

Hermes Agent requires a minimum context window of 64K tokens to function effectively, given its complex prompt assembly and memory systems. The `context_length` parameter informs the agent of the model's maximum capacity, allowing it to trigger context compression algorithms when the limit is approached.

For local models running via Ollama, the `ollama_num_ctx` parameter is crucial. It explicitly sets the context window size allocated by the Ollama server. If this is set too low, the agent will fail with context limit errors.

```yaml
model:
  provider: "ollama"
  model_name: "llama3.1:8b-instruct-q8_0"
  context_length: 128000
  ollama_num_ctx: 65536 # Must be at least 65536
```

### Generation Parameters

Standard LLM generation parameters are supported to fine-tune the agent's output:

- `temperature`: Controls randomness (0.0 to 2.0). Lower values produce more deterministic output. Default is typically 0.7.
- `max_tokens`: The maximum number of tokens to generate in a single response.
- `top_p`: Nucleus sampling parameter.
- `frequency_penalty`: Penalizes new tokens based on their existing frequency in the text.
- `presence_penalty`: Penalizes new tokens based on whether they appear in the text so far.

Properly configuring these parameters is essential for balancing creativity and reliability, especially when the agent is executing complex tool chains or writing code.

## 4. Terminal Configuration

The `terminal` section configures the execution environment where Hermes Agent runs shell commands and interacts with the file system. Hermes Agent supports multiple backends, ranging from local execution to isolated cloud sandboxes.

### Backend Selection

The `backend` key determines the execution environment. Supported backends include:

- `local`: Executes commands directly on the host machine. (Default, but least secure).
- `docker`: Executes commands within a persistent Docker container. (Recommended for most use cases).
- `ssh`: Executes commands on a remote server via SSH.
- `singularity`: Executes commands in a Singularity container (common in HPC environments).
- `modal`: Executes commands in a Modal cloud sandbox.
- `daytona`: Executes commands in a Daytona cloud workspace.
- `vercel_sandbox`: Executes commands in a Vercel microVM.

### Docker Backend Configuration

When using the `docker` backend, Hermes Agent provides extensive configuration options to secure and customize the container environment.

```yaml
terminal:
  backend: "docker"
  docker_image: "ubuntu:22.04"
  docker_mount: "/home/user/projects:/workspace"
  docker_volumes:
    - "/var/run/docker.sock:/var/run/docker.sock" # Docker-in-Docker support
  docker_extra_args: "--cap-drop ALL --security-opt no-new-privileges --pids-limit 100"
  container_cpu: "2.0"
  container_memory: "4g"
  container_disk: "20g"
  container_persistent: true
```

- `docker_image`: The base image for the container.
- `docker_mount`: Maps a host directory to the container's workspace.
- `docker_volumes`: Additional volume mounts.
- `docker_extra_args`: Crucial for security hardening. `--cap-drop ALL` removes unnecessary Linux capabilities, and `--security-opt no-new-privileges` prevents privilege escalation.
- `container_persistent`: If `true`, the container remains running across agent invocations, preserving state and improving performance.

### SSH Backend Configuration

The `ssh` backend utilizes SSH ControlMaster for persistent, high-performance connections to remote servers.

```yaml
terminal:
  backend: "ssh"
  ssh_host: "192.168.1.100"
  ssh_user: "developer"
  ssh_key: "~/.ssh/id_ed25519"
  ssh_port: 2222
  timeout: 600
```

The `timeout` parameter specifies the maximum duration (in seconds) a command is allowed to run before being terminated. This prevents runaway processes from hanging the agent.

## 5. MCP Servers Configuration

The Model Context Protocol (MCP) allows Hermes Agent to seamlessly integrate with external tools and data sources. The `mcp_servers` section defines the connections to these servers.

Hermes Agent supports both `stdio` (standard input/output) and `http` (Server-Sent Events) MCP servers.

### Stdio MCP Servers

Stdio servers are executed as child processes. The configuration requires the command to run and its arguments.

```yaml
mcp_servers:
  - name: "sqlite-mcp"
    command: "uvx"
    args: ["mcp-server-sqlite", "--db-path", "/var/lib/data.db"]
    env:
      DEBUG: "1"
```

### HTTP MCP Servers

HTTP servers are accessed over the network. The configuration requires the URL and any necessary authentication headers.

```yaml
mcp_servers:
  - name: "internal-api-mcp"
    url: "https://api.internal.company.com/mcp"
    headers:
      Authorization: "Bearer ${INTERNAL_API_TOKEN}"
      X-Custom-Header: "HermesAgent"
```

### Tool Filtering

To adhere to the principle of least privilege, you can restrict which tools from an MCP server are exposed to the agent using the `tool_filter` array.

```yaml
mcp_servers:
  - name: "github-mcp"
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    tool_filter: ["search_repositories", "read_issue"] # Only these tools will be available
```

This is a critical security feature, preventing the agent from accidentally executing destructive actions (like deleting repositories) if the MCP server exposes them.

## 6. Skills Configuration

Skills are procedural memories that allow Hermes Agent to learn and execute complex workflows. The `skills` section configures how these skills are discovered, loaded, and managed.

```yaml
skills:
  paths:
    - "~/.hermes/skills"
    - "/var/lib/hermes/shared-skills"
  config:
    auto_update: true
    trust_level: "verified_only"
    fallback_for_toolsets: ["web", "terminal"]
```

- `paths`: An array of directories where the agent should look for skills. Each skill is a directory containing a `SKILL.md` file.
- `auto_update`: If true, the agent will periodically check the Skills Hub (agentskills.io) for updates to installed skills.
- `trust_level`: Defines the security posture for loading skills. `verified_only` ensures only cryptographically signed skills from trusted authors are loaded.
- `fallback_for_toolsets`: Specifies which toolsets should trigger conditional activation of fallback skills if the primary tools fail.

Skills can also define their own configuration settings within their `SKILL.md` files, which are parsed and injected into the agent's environment at runtime.

## 7. Memory Configuration

Hermes Agent maintains a persistent memory system, consisting of a user profile (`USER.md`) and the agent's personal notes (`MEMORY.md`). The `memory` section configures the limits and behavior of this system.

```yaml
memory:
  provider: "sqlite"
  user_char_limit: 1375
  agent_char_limit: 2200
  snapshot_retention: 7
  compression_threshold: 0.8
```

- `provider`: The storage backend for memories. `sqlite` is the default, utilizing FTS5 for full-text search capabilities.
- `user_char_limit`: The maximum size of the `USER.md` file (1,375 characters is approximately 500 tokens).
- `agent_char_limit`: The maximum size of the `MEMORY.md` file (2,200 characters is approximately 800 tokens).
- `snapshot_retention`: The number of days to retain frozen memory snapshots. These snapshots are taken at the start of each session and are optimized for prefix caching.
- `compression_threshold`: The context window utilization percentage (e.g., 80%) that triggers automatic context compression algorithms.

## 8. Gateway Configuration

The Gateway system allows Hermes Agent to connect to over 20 messaging platforms (Telegram, Discord, Slack, etc.). The `gateway` section configures these connections.

```yaml
gateway:
  platforms: ["telegram", "discord"]
  home_channels:
    telegram: "-100987654321"
    discord: "987654321098765432"
  session_reset_timeout: 3600
  delivery_preferences:
    chunk_size: 2000
    markdown_support: true
```

- `platforms`: An array of enabled messaging platforms. The corresponding API keys must be set in the `.env` file.
- `home_channels`: Specifies the default channels or groups where the agent should report status updates or errors.
- `session_reset_timeout`: The duration of inactivity (in seconds) before a conversation session is automatically reset, clearing the short-term context.
- `delivery_preferences`: Configures how messages are formatted and delivered. `chunk_size` ensures messages do not exceed platform limits, and `markdown_support` enables rich text formatting.

## 9. Cron Configuration

Hermes Agent includes a built-in cron scheduler for executing recurring tasks. The `cron` section configures the scheduler's behavior.

```yaml
cron:
  jobs_file: "~/.hermes/cron/jobs.json"
  timezone: "America/New_York"
  max_concurrent_jobs: 5
```

- `jobs_file`: The path to the JSON file containing the scheduled jobs.
- `timezone`: The timezone used for evaluating cron expressions.
- `max_concurrent_jobs`: Limits the number of jobs that can run simultaneously to prevent resource exhaustion.

The `jobs.json` file contains entries defining the schedule, the prompt to execute, and the target platform for delivery. The CLI commands (`hermes cron create`, `hermes cron list`, etc.) manage this file.

## 10. Browser and Toolset Configuration

### Browser Configuration

The `browser` section configures the automated browser tools used for web navigation and visual analysis.

```yaml
browser:
  headless: true
  viewport_width: 1920
  viewport_height: 1080
  user_agent: "HermesAgent/0.14.0"
  timeout: 30000
```

These settings control the Playwright instance used by the agent. Running in `headless` mode is recommended for server environments.

### Toolset Configuration

The `toolset` section allows administrators to explicitly enable or disable specific categories of tools.

```yaml
toolset:
  enabled:
    - "web"
    - "terminal"
    - "browser"
  disabled:
    - "computer_use"
    - "media"
```

This is a critical security control. For example, disabling the `computer_use` toolset prevents the agent from attempting to control the host operating system's GUI, which is essential for headless server deployments.

## 11. `.env` File Format and Secret Variables

The `.env` file is strictly reserved for sensitive information, such as API keys, access tokens, and database passwords. It should never be committed to version control. Hermes Agent uses the `python-dotenv` library to load these variables into the environment at runtime.

A typical `~/.hermes/.env` file looks like this:

```env
# Core API Keys
OPENAI_API_KEY=sk-proj-...
ANTHROPIC_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-v1-...
XAI_API_KEY=xai-...

# Gateway Platform Tokens
TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz
DISCORD_BOT_TOKEN=<your-discord-bot-token-here>
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...

# External Service Integrations
GITHUB_TOKEN=ghp_...
SPOTIFY_CLIENT_ID=...
SPOTIFY_CLIENT_SECRET=...

# Observability and Monitoring
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_HOST=https://cloud.langfuse.com
```

Hermes Agent's configuration system supports environment variable passthrough. This means you can reference variables defined in the `.env` file within your `config.yaml` or MCP server configurations using the `${VARIABLE_NAME}` syntax.

## 12. Environment Variables Complete Reference

Hermes Agent recognizes over 100 environment variables. They are organized into several categories for clarity.

### Core Provider Keys

These variables authenticate the agent with various LLM providers.

- `OPENAI_API_KEY`: Required for OpenAI models (GPT-4o, etc.).
- `ANTHROPIC_API_KEY`: Required for Anthropic models (Claude 3.5 Sonnet, etc.).
- `OPENROUTER_API_KEY`: Required for OpenRouter routing.
- `GEMINI_API_KEY`: Required for Google Gemini models.
- `XAI_API_KEY`: Required for xAI Grok models.
- `GROQ_API_KEY`: Required for Groq inference.
- `TOGETHER_API_KEY`: Required for Together AI.
- `OLLAMA_HOST`: The URL of the Ollama server (defaults to `http://localhost:11434`).

### Terminal and Execution Environment

These variables override settings in the `terminal` section of `config.yaml`.

- `HERMES_BACKEND`: Overrides the terminal backend (e.g., `docker`, `ssh`).
- `HERMES_CWD`: Overrides the default working directory.
- `HERMES_TIMEOUT`: Overrides the command execution timeout.
- `DOCKER_HOST`: Specifies the Docker daemon socket.

### Gateway and Platform Integration

These variables configure the messaging gateway adapters.

- `TELEGRAM_BOT_TOKEN`: The token provided by BotFather.
- `DISCORD_BOT_TOKEN`: The token from the Discord Developer Portal.
- `SLACK_BOT_TOKEN`: The OAuth token for the Slack workspace.
- `SLACK_APP_TOKEN`: The App-Level token for Socket Mode connections.
- `MATRIX_HOMESERVER`: The URL of the Matrix homeserver.
- `MATRIX_ACCESS_TOKEN`: The access token for the Matrix account.

### Observability and Telemetry

These variables configure integration with observability platforms like Langfuse.

- `LANGFUSE_PUBLIC_KEY`: Langfuse public key.
- `LANGFUSE_SECRET_KEY`: Langfuse secret key.
- `LANGFUSE_HOST`: The Langfuse API endpoint.
- `HERMES_DEBUG`: Set to `1` or `true` to enable verbose debug logging.
- `HERMES_LOG_LEVEL`: Sets the logging level (`DEBUG`, `INFO`, `WARNING`, `ERROR`).

### Tool Gateway and External Services

These variables provide credentials for specific tools.

- `TAVILY_API_KEY`: Required for the `web_search` tool if using Tavily.
- `EXA_API_KEY`: Required for the `web_search` tool if using Exa.
- `FAL_KEY`: Required for image and video generation tools using FAL.ai.
- `GITHUB_TOKEN`: Used by GitHub-related skills and MCP servers.

## 13. Context Files: SOUL.md, AGENTS.md, .cursorrules, .hermes.md

Hermes Agent's prompt assembly pipeline incorporates several context files to define the agent's identity, behavior, and project-specific instructions.

### SOUL.md

The `SOUL.md` file defines the core identity and persona of the agent. It is the first layer in the prompt assembly pipeline.

```markdown
# SOUL.md
You are Hermes, an autonomous AI agent designed to assist with software development and system administration.
You are concise, highly technical, and prioritize security.
You always verify file contents before modifying them.
```

Customizing `SOUL.md` allows you to tailor the agent's tone and fundamental directives to your specific use case.

### AGENTS.md

The `AGENTS.md` file is used in multi-agent scenarios, particularly when utilizing the Kanban orchestration system. It defines the roles and responsibilities of different sub-agents.

```markdown
# AGENTS.md
- **Architect**: Responsible for system design and selecting technologies.
- **Developer**: Responsible for writing code and implementing features.
- **Reviewer**: Responsible for code review and security auditing.
```

### .cursorrules and .hermes.md

These files provide project-specific context. When Hermes Agent operates within a directory containing a `.cursorrules` or `.hermes.md` file, it automatically reads and incorporates their contents into the system prompt.

```markdown
# .hermes.md
Project: E-commerce Backend
Language: Python 3.11
Framework: FastAPI
Database: PostgreSQL

Rules:
- All database queries must use SQLAlchemy ORM.
- All endpoints must have comprehensive docstrings.
- Do not use raw SQL strings.
```

This mechanism ensures that the agent adheres to project-specific coding standards and architectural guidelines without requiring manual prompting for every interaction.

## 14. Profile System (Multi-instance Support)

Hermes Agent supports a profile system, allowing you to run multiple isolated instances of the agent on the same machine. Each profile maintains its own configuration, memory, and session database.

By default, Hermes Agent uses the `default` profile, storing data in `~/.hermes/`.

You can specify a different profile using the `--profile` flag or the `HERMES_PROFILE` environment variable.

```bash
# Run the agent using the 'work' profile
hermes --profile work chat

# Run the agent using the 'personal' profile
HERMES_PROFILE=personal hermes chat
```

When a new profile is specified, Hermes Agent creates a new directory structure (e.g., `~/.hermes_work/`) containing a fresh `config.yaml`, `.env`, and memory database. This is invaluable for separating work and personal tasks, or for testing different configurations without affecting your primary agent instance.

The profile system ensures complete isolation, preventing cross-contamination of memories, credentials, and configuration settings between different agent personas.

## 15. Advanced Configuration Scenarios

### Configuring for High Availability

In production environments, you may want to configure Hermes Agent for high availability. This involves setting up multiple instances of the agent behind a load balancer, sharing a common database and memory store.

```yaml
# config.yaml for HA setup
memory:
  provider: "postgres"
  connection_string: "postgresql://user:password@db.internal:5432/hermes"
```

By switching the memory provider from `sqlite` to `postgres`, multiple agent instances can share the same memory and session state, allowing for seamless failover and load distribution.

### Configuring for Strict Security

For environments with strict security requirements, you can configure Hermes Agent to operate with minimal privileges and restricted network access.

```yaml
# config.yaml for strict security
terminal:
  backend: "docker"
  docker_extra_args: "--network none --read-only --tmpfs /tmp"
toolset:
  enabled:
    - "terminal"
  disabled:
    - "web"
    - "browser"
    - "computer_use"
```

This configuration disables network access for the Docker container, mounts the root filesystem as read-only, and disables all tools that could potentially interact with external services or the host operating system.

### Configuring for Custom Toolsets

Hermes Agent allows you to define custom toolsets by grouping specific tools together. This is useful for creating specialized agents with a limited set of capabilities.

```yaml
# config.yaml for custom toolsets
toolset:
  custom_sets:
    - name: "data_analysis"
      tools: ["read_file", "execute_code", "vision_analyze"]
  enabled:
    - "data_analysis"
```

This configuration creates a custom toolset named `data_analysis` containing only the tools necessary for data analysis tasks, and enables it for the agent.

## 16. Troubleshooting Configuration Issues

When configuring Hermes Agent, you may encounter various issues. Here are some common problems and their solutions:

- **Invalid YAML Syntax**: Ensure that your `config.yaml` file is properly formatted. Use a YAML validator to check for syntax errors.
- **Missing Environment Variables**: Verify that all required environment variables are set in your `.env` file or exported in your shell environment.
- **Permission Denied**: Ensure that the agent has the necessary permissions to read the configuration files and access the specified directories (e.g., `~/.hermes/`).
- **Context Limit Errors**: If you encounter context limit errors, check the `context_length` and `ollama_num_ctx` settings in your `config.yaml` file. Ensure they match the capabilities of your chosen model.
- **Docker Backend Issues**: If the Docker backend fails to start, verify that the Docker daemon is running and that the agent has permission to access the Docker socket. Check the `docker_extra_args` for any conflicting options.

By carefully reviewing your configuration files and consulting the Hermes Agent documentation, you can resolve most configuration issues and ensure a stable and secure deployment.
