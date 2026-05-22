# Hermes Agent CLI Reference Guide

## 1. Introduction to Hermes Agent CLI

The Hermes Agent (v0.14.0) by NousResearch is a powerful, self-improving AI agent designed to operate across multiple environments, from local terminals to cloud sandboxes. At the core of its operation is a comprehensive Command Line Interface (CLI) that provides operators with granular control over the agent's behavior, configuration, and execution. This document serves as the definitive reference for the Hermes Agent CLI, detailing every command, flag, environment variable, and configuration schema required to operate the agent in production environments.

The CLI is built to be intuitive yet powerful, offering commands for starting interactive chat sessions, managing background cron jobs, configuring the agent's toolset, and diagnosing system health. Whether you are running Hermes locally, within a persistent Docker container, or across a distributed SSH cluster, the CLI provides the necessary tools to manage the agent's lifecycle effectively.

This reference guide is structured to provide both high-level overviews and deep technical dives into each component of the CLI. It covers the primary `hermes` command and its subcommands, the interactive slash commands available during chat sessions, the extensive list of environment variables, and the complete schema for the `config.yaml` file. By mastering these tools, operators can fully leverage the capabilities of the Hermes Agent, ensuring robust, secure, and efficient AI operations.

## 2. Core CLI Commands

The primary entry point for interacting with the Hermes Agent is the `hermes` command. This command serves as a multiplexer, routing user input to the appropriate subsystem based on the provided subcommands and flags. Below is a detailed breakdown of the core CLI commands and their respective functionalities.

### 2.1 `hermes` (Default Behavior)

When invoked without any subcommands, the `hermes` command initiates an interactive chat session with the agent. This is the most common way to interact with Hermes, allowing users to issue natural language instructions, which the agent then translates into actionable tasks using its configured toolset.

**Usage:**
```bash
hermes [options]
```

**Options:**
- `--continue`, `-c`: Resumes the previous session. This is particularly useful when a session was interrupted or when the user wants to pick up exactly where they left off. The agent retrieves the session state from its SQLite database, restoring the conversation history and context.
- `--tui`: Launches the Terminal User Interface (TUI). The TUI provides a modern, visually rich interface with modal overlays, making it easier to manage complex interactions, view tool execution logs, and navigate the agent's memory.
- `--debug`: Enables debug logging. This flag is essential for troubleshooting, as it outputs detailed information about the agent's internal state, including prompt assembly, tool dispatch, and API responses.
- `--version`: Displays the current version of the Hermes Agent (e.g., `0.14.0`).

### 2.2 `hermes chat`

The `hermes chat` command is an explicit way to start an interactive session. It functions identically to the default `hermes` command but is often used in scripts or aliases for clarity.

**Usage:**
```bash
hermes chat [options]
```

**Options:**
- `--session-id <id>`: Starts or resumes a specific session by its unique identifier. This allows operators to manage multiple concurrent sessions and switch between them seamlessly.
- `--system-prompt <text>`: Overrides the default system prompt for the session. This is useful for testing specific behaviors or temporarily altering the agent's persona without modifying the `SOUL.md` file.

### 2.3 `hermes model`

The `hermes model` command is used to manage the agent's language model configuration. Hermes supports over 30 AI providers, and this command allows operators to quickly switch between models, configure provider settings, and test model connectivity.

**Usage:**
```bash
hermes model [subcommand]
```

**Subcommands:**
- `list`: Displays a list of all supported providers and models. This includes both local models (e.g., Ollama, LM Studio) and cloud-based APIs (e.g., OpenAI, Anthropic, OpenRouter).
- `set <provider> <model>`: Sets the active provider and model. For example, `hermes model set openai gpt-4o` configures the agent to use OpenAI's GPT-4o model.
- `status`: Shows the currently active model, its context length, and the status of the provider connection.

### 2.4 `hermes tools`

The `hermes tools` command manages the agent's toolset. Hermes comes with over 40 built-in tools across 15 categories, and this command allows operators to enable, disable, and configure these tools.

**Usage:**
```bash
hermes tools [subcommand]
```

**Subcommands:**
- `list`: Lists all available tools, categorized by their function (e.g., Web, Terminal, Browser, Media). It also indicates which tools are currently enabled.
- `enable <tool_name>`: Enables a specific tool. For example, `hermes tools enable browser_vision` activates the browser vision capabilities.
- `disable <tool_name>`: Disables a specific tool. This is useful for restricting the agent's capabilities in sensitive environments.
- `config <tool_name>`: Displays or modifies the configuration for a specific tool.

### 2.5 `hermes config`

The `hermes config` command is the primary interface for managing the agent's configuration settings. It interacts directly with the `~/.hermes/config.yaml` file, allowing operators to view, modify, and reset configuration values without manually editing the file.

**Usage:**
```bash
hermes config [subcommand]
```

**Subcommands:**
- `show`: Displays the current configuration in a readable format.
- `set <key> <value>`: Updates a specific configuration key. For example, `hermes config set terminal.backend docker` changes the terminal backend to Docker.
- `get <key>`: Retrieves the value of a specific configuration key.
- `reset`: Resets the configuration to its default state. This is a destructive operation and should be used with caution.
- `migrate`: Migrates the configuration from an older version of Hermes or OpenClaw to the current schema.

### 2.6 `hermes setup`

The `hermes setup` command launches the first-time setup wizard. This interactive wizard guides the user through the process of configuring the agent, including selecting a primary AI provider, entering API keys, and choosing a terminal backend.

**Usage:**
```bash
hermes setup
```

The wizard covers the following steps:
1. **Provider Selection**: The user selects their preferred AI provider from the list of supported options.
2. **API Key Configuration**: The wizard prompts the user to enter the necessary API keys, which are securely stored in the `~/.hermes/.env` file.
3. **Terminal Backend**: The user chooses the terminal backend (e.g., local, docker, ssh) that best suits their environment.
4. **Tool Configuration**: The wizard allows the user to enable or disable specific tool categories based on their needs.

### 2.7 `hermes doctor`

The `hermes doctor` command is a diagnostic tool that checks the health of the Hermes Agent installation. It verifies dependencies, tests provider connections, and ensures that the configuration files are valid.

**Usage:**
```bash
hermes doctor
```

The diagnostic checks include:
- **Dependency Verification**: Ensures that all required Python packages are installed and match the exact-pinned versions specified in the requirements file.
- **Provider Connectivity**: Tests the connection to the configured AI provider and verifies that the API keys are valid.
- **Configuration Validation**: Checks the `config.yaml` and `.env` files for syntax errors or missing required fields.
- **Backend Status**: Verifies that the selected terminal backend (e.g., Docker daemon) is running and accessible.

### 2.8 `hermes cron`

The `hermes cron` command manages the agent's scheduled jobs. Hermes includes a robust cron system that allows operators to schedule tasks to run at specific intervals, with the results delivered via the messaging gateway.

**Usage:**
```bash
hermes cron [subcommand]
```

**Subcommands:**
- `create`: Creates a new cron job. The user is prompted to provide a schedule (in standard cron syntax), a task description, and the target delivery platform.
- `list`: Lists all active and paused cron jobs, including their next scheduled run time.
- `update <job_id>`: Modifies an existing cron job.
- `pause <job_id>`: Temporarily suspends a cron job without deleting it.
- `resume <job_id>`: Resumes a paused cron job.
- `run <job_id>`: Manually triggers a cron job to run immediately, bypassing the schedule.
- `remove <job_id>`: Permanently deletes a cron job.

### 2.9 `hermes gateway`

The `hermes gateway` command manages the messaging gateway, which connects the agent to over 20 messaging platforms (e.g., Telegram, Discord, Slack).

**Usage:**
```bash
hermes gateway [subcommand]
```

**Subcommands:**
- `start`: Starts the gateway service, allowing the agent to receive and respond to messages from configured platforms.
- `stop`: Stops the gateway service.
- `status`: Displays the status of the gateway service and lists the active platform connections.
- `config`: Opens the gateway configuration interface.

### 2.10 `hermes skills`

The `hermes skills` command manages the agent's skills system. Skills are procedural memories that the agent creates and improves over time, allowing it to perform complex tasks more efficiently.

**Usage:**
```bash
hermes skills [subcommand]
```

**Subcommands:**
- `list`: Lists all installed skills, categorized by their source (bundled vs. optional).
- `install <skill_name>`: Installs a new skill from the Skills Hub (agentskills.io) or a local directory.
- `remove <skill_name>`: Uninstalls a skill.
- `hub`: Opens an interactive interface for browsing and installing skills from the Skills Hub.

### 2.11 `hermes claw migrate`

The `hermes claw migrate` command provides a seamless migration path for users upgrading from OpenClaw, the predecessor to Hermes Agent.

**Usage:**
```bash
hermes claw migrate [options]
```

**Options:**
- `--dry-run`: Simulates the migration process without making any changes to the file system. This is useful for verifying that the migration will succeed before committing to it.
- `--force`: Forces the migration, overwriting any existing Hermes configuration files.
- `--backup`: Creates a backup of the OpenClaw configuration before migrating.

## 3. Interactive Slash Commands

During an interactive chat session, operators can use slash commands to control the agent's behavior, manage the session state, and interact with the underlying system. These commands provide a quick and efficient way to perform common tasks without leaving the chat interface.

### 3.1 Session Management

- `/new`: Starts a new session, clearing the conversation history and resetting the context. This is useful when switching to a completely different task.
- `/reset`: Resets the current session, clearing the conversation history but retaining the session ID and metadata.
- `/stop`: Stops the current tool execution or generation process. This is essential when the agent is stuck in a loop or executing a long-running task that is no longer needed.
- `/status`: Displays the current status of the session, including the active model, token usage, and enabled tools.

### 3.2 Command Approval

Hermes includes a robust security system that requires user approval for potentially dangerous commands (e.g., `rm -rf`, `docker rm`). The following slash commands are used to manage this approval flow:

- `/approve`: Approves the pending command, allowing the agent to execute it.
- `/deny`: Denies the pending command, preventing its execution. The agent will be notified of the denial and can attempt an alternative approach.

### 3.3 Configuration and Tools

- `/model <provider> <model>`: Switches the active AI provider and model on the fly. For example, `/model anthropic claude-3-opus-20240229`.
- `/tools`: Opens an interactive menu for enabling and disabling tools within the current session.
- `/cron`: Opens an interactive menu for managing cron jobs.

### 3.4 Memory and Context

- `/memory`: Displays the agent's current memory state, including the `MEMORY.md` and `USER.md` snapshots.
- `/compact`: Manually triggers the context compression algorithm. This is useful when the conversation history is approaching the model's context limit and the operator wants to free up space without losing critical information.
- `/save`: Saves the current session state to the SQLite database. While Hermes automatically saves the state periodically, this command ensures that all recent changes are persisted immediately.

### 3.5 Data Export and Import

- `/export <format>`: Exports the current session history to a file. Supported formats include `json`, `markdown`, and `html`.
- `/import <file>`: Imports a previously exported session history, allowing the operator to resume a past conversation or analyze it using the agent's tools.

## 4. Environment Variables Reference

Hermes Agent relies heavily on environment variables for configuring sensitive information, such as API keys, and for overriding default settings in containerized or CI/CD environments. These variables are typically stored in the `~/.hermes/.env` file but can also be exported directly in the shell.

### 4.1 Core Configuration

- `HERMES_HOME`: Specifies the base directory for Hermes configuration and data files. Default: `~/.hermes`.
- `HERMES_ENV`: Defines the execution environment (e.g., `development`, `production`, `testing`). This variable affects logging verbosity and error handling.
- `HERMES_DEBUG`: Enables debug logging when set to `true` or `1`.
- `HERMES_LOG_LEVEL`: Sets the logging level (e.g., `DEBUG`, `INFO`, `WARNING`, `ERROR`). Default: `INFO`.

### 4.2 Provider API Keys

Hermes supports over 30 AI providers, each requiring its own API key. Below are the environment variables for the most common providers:

- `OPENAI_API_KEY`: API key for OpenAI models (e.g., GPT-4o).
- `ANTHROPIC_API_KEY`: API key for Anthropic models (e.g., Claude 3).
- `GOOGLE_API_KEY`: API key for Google Gemini models.
- `OPENROUTER_API_KEY`: API key for OpenRouter, which provides access to a wide range of models.
- `XAI_API_KEY`: API key for xAI's Grok models.
- `TOGETHER_API_KEY`: API key for Together AI.
- `GROQ_API_KEY`: API key for Groq's high-speed inference API.
- `MISTRAL_API_KEY`: API key for Mistral AI.
- `COHERE_API_KEY`: API key for Cohere.
- `PERPLEXITY_API_KEY`: API key for Perplexity AI.

### 4.3 Terminal Backend Configuration

- `HERMES_TERMINAL_BACKEND`: Specifies the terminal backend to use (e.g., `local`, `docker`, `ssh`, `modal`). Default: `local`.
- `HERMES_DOCKER_IMAGE`: Specifies the Docker image to use when the Docker backend is enabled. Default: `ubuntu:latest`.
- `HERMES_SSH_HOST`: The hostname or IP address of the remote server when using the SSH backend.
- `HERMES_SSH_USER`: The username for the SSH connection.
- `HERMES_SSH_KEY`: The path to the SSH private key file.

### 4.4 Gateway and Platform Configuration

- `HERMES_GATEWAY_ENABLED`: Enables the messaging gateway when set to `true`.
- `TELEGRAM_BOT_TOKEN`: The token for the Telegram bot integration.
- `DISCORD_BOT_TOKEN`: The token for the Discord bot integration.
- `SLACK_BOT_TOKEN`: The token for the Slack bot integration.
- `WHATSAPP_API_KEY`: The API key for the WhatsApp integration.
- `MATRIX_HOMESERVER`: The URL of the Matrix homeserver.
- `MATRIX_ACCESS_TOKEN`: The access token for the Matrix integration.

### 4.5 Observability and Telemetry

- `LANGFUSE_PUBLIC_KEY`: The public key for Langfuse observability integration.
- `LANGFUSE_SECRET_KEY`: The secret key for Langfuse observability integration.
- `LANGFUSE_HOST`: The URL of the Langfuse instance.
- `HERMES_TELEMETRY_ENABLED`: Enables anonymous telemetry collection when set to `true`. Default: `false`.

### 4.6 Tool Gateway Configuration

- `MCP_SERVER_URL`: The URL of the Model Context Protocol (MCP) server.
- `MCP_API_KEY`: The API key for authenticating with the MCP server.
- `SPOTIFY_CLIENT_ID`: The client ID for the Spotify integration.
- `SPOTIFY_CLIENT_SECRET`: The client secret for the Spotify integration.
- `GITHUB_TOKEN`: The personal access token for GitHub integration.

## 5. Config.yaml Complete Schema

The `~/.hermes/config.yaml` file is the central configuration repository for the Hermes Agent. It defines the agent's behavior, toolset, terminal backend, and more. Below is a comprehensive breakdown of the schema, including all available sections and their respective keys.

### 5.1 Model Configuration

This section defines the primary AI provider and model settings.

```yaml
model:
  provider: "openai" # The active AI provider (e.g., openai, anthropic, ollama)
  name: "gpt-4o" # The specific model to use
  context_length: 128000 # The maximum context length supported by the model
  temperature: 0.7 # The sampling temperature (0.0 to 2.0)
  ollama_num_ctx: 65536 # The context window size for Ollama models (minimum 64K required)
  failover:
    enabled: true # Enables automatic failover to a secondary provider
    secondary_provider: "anthropic"
    secondary_model: "claude-3-haiku-20240307"
```

### 5.2 Terminal Configuration

This section configures the terminal backend, which determines where and how the agent executes commands.

```yaml
terminal:
  backend: "docker" # The active backend (local, docker, ssh, singularity, modal, daytona, vercel_sandbox)
  cwd: "/workspace" # The default working directory for the terminal session
  timeout: 300 # The maximum execution time for a single command (in seconds)
  
  # Docker-specific settings
  docker_image: "ubuntu:22.04" # The Docker image to use for the container
  docker_mount: "/host/path:/container/path" # Volume mounts
  docker_volumes:
    - "hermes_data:/data"
  docker_extra_args: "--privileged" # Additional arguments passed to the `docker run` command
  container_cpu: "2.0" # CPU limit for the container
  container_memory: "4g" # Memory limit for the container
  container_disk: "20g" # Disk space limit for the container
  container_persistent: true # Whether the container should persist across sessions
  
  # SSH-specific settings
  ssh:
    host: "remote.server.com"
    user: "ubuntu"
    key_path: "~/.ssh/id_rsa"
    port: 22
    control_master: true # Enables SSH multiplexing for faster connections
```

### 5.3 MCP Servers Configuration

This section configures the Model Context Protocol (MCP) servers, which allow the agent to interact with external tools and services.

```yaml
mcp_servers:
  - name: "local_tools"
    command: "python3"
    args: ["/path/to/mcp_server.py"]
    env:
      DEBUG: "1"
    url: "http://localhost:8080"
    headers:
      Authorization: "Bearer ${MCP_API_KEY}"
    tool_filter:
      - "web_search"
      - "read_file"
```

### 5.4 Skills Configuration

This section manages the agent's skills system, including paths to skill directories and specific skill settings.

```yaml
skills:
  paths:
    - "~/.hermes/skills" # Directory for bundled skills
    - "~/.hermes/optional-skills" # Directory for optional skills
  config:
    auto_update: true # Automatically update skills from the Skills Hub
    fallback_enabled: true # Enable fallback skills when primary tools fail
    hub_url: "https://agentskills.io/api/v1"
```

### 5.5 Memory Configuration

This section configures the agent's memory system, including character limits and storage providers.

```yaml
memory:
  provider: "sqlite" # The storage provider (sqlite, honcho)
  char_limits:
    memory_md: 2200 # Maximum characters for MEMORY.md (~800 tokens)
    user_md: 1375 # Maximum characters for USER.md (~500 tokens)
  compression:
    enabled: true
    threshold: 0.8 # Trigger compression when context reaches 80% of the limit
    strategy: "trajectory" # The compression strategy (trajectory, conversation)
```

### 5.6 Gateway Configuration

This section configures the messaging gateway, defining which platforms the agent connects to and how it handles messages.

```yaml
gateway:
  enabled: true
  platforms:
    - "telegram"
    - "discord"
  home_channels:
    telegram: "-1001234567890" # The primary channel for notifications
    discord: "123456789012345678"
  session_reset:
    timeout: 3600 # Reset the session after 1 hour of inactivity
  delivery_preferences:
    format: "markdown" # The preferred message format (markdown, html, plain)
    chunk_size: 4000 # Maximum characters per message chunk
```

### 5.7 Cron Configuration

This section configures the cron scheduling system.

```yaml
cron:
  enabled: true
  jobs_file: "~/.hermes/cron/jobs.json" # Path to the jobs definition file
  timezone: "UTC" # The default timezone for cron schedules
  delivery_platform: "telegram" # The default platform for delivering cron job results
```

### 5.8 Browser Configuration

This section configures the browser automation tools.

```yaml
browser:
  headless: true # Run the browser in headless mode
  viewport:
    width: 1920
    height: 1080
  user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
  timeout: 30000 # Maximum time to wait for page loads (in milliseconds)
```

### 5.9 Toolset Configuration

This section defines which tools are enabled or disabled by default.

```yaml
toolset:
  enabled_categories:
    - "web"
    - "terminal"
    - "browser"
    - "media"
  disabled_tools:
    - "computer_use" # Disable macOS desktop control by default for security
    - "execute_code"
```

## 6. Advanced CLI Usage and Best Practices

To fully harness the power of the Hermes Agent CLI, operators should adopt advanced usage patterns and best practices. This section covers techniques for optimizing workflows, managing complex environments, and ensuring system security.

### 6.1 Scripting and Automation

The Hermes CLI is designed to be highly scriptable. By combining the `hermes chat` command with input redirection and shell scripting, operators can automate complex tasks.

**Example: Automated Code Review**
```bash
#!/bin/bash
# review_code.sh

REPO_DIR="/path/to/repo"
PR_DIFF=$(cd $REPO_DIR && git diff main...HEAD)

echo "Review the following code changes and provide feedback:" > prompt.txt
echo "$PR_DIFF" >> prompt.txt

hermes chat --system-prompt "You are an expert code reviewer." < prompt.txt > review_output.md
```

This script extracts the diff of a pull request, constructs a prompt, and pipes it into the Hermes CLI. The agent's response is then saved to a markdown file.

### 6.2 Managing Multiple Profiles

In production environments, operators often need to manage multiple instances of the Hermes Agent, each with its own configuration and memory state. The CLI supports this through the use of profiles.

By setting the `HERMES_HOME` environment variable, operators can isolate the configuration and data for different profiles.

**Example: Switching Profiles**
```bash
# Profile 1: Development
export HERMES_HOME="~/.hermes_dev"
hermes config set model.name gpt-4o-mini

# Profile 2: Production
export HERMES_HOME="~/.hermes_prod"
hermes config set model.name gpt-4o
```

This approach ensures that experimental changes in the development profile do not affect the production environment.

### 6.3 Security Considerations

When operating the Hermes Agent, especially with powerful terminal backends like Docker or SSH, security is paramount. The CLI provides several mechanisms to mitigate risks.

1. **Command Approval Flow**: Always keep the DANGEROUS_PATTERNS approval system enabled. This system intercepts potentially destructive commands (e.g., `rm -rf /`) and requires explicit user approval via the `/approve` slash command.
2. **Principle of Least Privilege**: When configuring the Docker backend, use the `docker_extra_args` setting to drop unnecessary capabilities (e.g., `--cap-drop ALL`) and restrict resource usage (`container_cpu`, `container_memory`).
3. **Environment Variable Passthrough**: Be cautious when passing environment variables to the terminal backend. Only pass variables that are strictly necessary for the task at hand to prevent credential leakage.

### 6.4 Troubleshooting with `hermes doctor` and `--debug`

When issues arise, the `hermes doctor` command should be the first step in the troubleshooting process. It quickly identifies common problems, such as missing dependencies or invalid API keys.

If `hermes doctor` does not reveal the issue, operators should run the CLI with the `--debug` flag. This provides a detailed trace of the agent's internal operations, including:
- The exact prompt assembled by the `prompt_builder.py`.
- The raw API request sent to the provider.
- The raw API response, including token usage and stop reasons.
- The tool dispatch logic and execution results.

By analyzing this debug output, operators can pinpoint the root cause of complex issues, such as context window overflow or tool execution failures.

## 7. Conclusion

The Hermes Agent CLI is a robust and versatile toolset that empowers operators to manage, configure, and interact with the agent across a wide range of environments. By mastering the core commands, interactive slash commands, environment variables, and configuration schemas detailed in this reference guide, users can unlock the full potential of the Hermes Agent, building secure, efficient, and highly capable AI-driven workflows.

## 8. Deep Dive: Environment Variables and System Integration

The environment variables in Hermes Agent are not just simple configuration toggles; they represent a deep integration layer that allows the agent to adapt to complex deployment scenarios. Understanding how these variables interact with the underlying system is crucial for advanced operators.

### 8.1 Advanced Provider Configuration via Environment Variables

While the `config.yaml` file handles the primary provider setup, environment variables offer a dynamic way to override these settings, which is particularly useful in containerized environments like Kubernetes or Docker Swarm.

- `HERMES_PROVIDER_OVERRIDE`: This variable forces the agent to use a specific provider, ignoring the `config.yaml` setting. This is useful for emergency failovers or testing.
- `HERMES_MODEL_OVERRIDE`: Similar to the provider override, this forces the use of a specific model.
- `HERMES_API_BASE_URL`: For providers that support custom endpoints (e.g., OpenAI-compatible APIs like vLLM or LM Studio), this variable sets the base URL for API requests.
- `HERMES_API_VERSION`: Specifies the API version to use, which is critical when providers introduce breaking changes in newer API versions.

### 8.2 Terminal Backend Deep Dive

The terminal backend is one of the most powerful features of Hermes Agent, allowing it to execute commands in isolated environments. The environment variables associated with this feature provide granular control over the execution context.

- `HERMES_DOCKER_NETWORK`: Specifies the Docker network the container should join. This is essential when the agent needs to interact with other services running in the same Docker environment (e.g., a database or a web server).
- `HERMES_DOCKER_USER`: Defines the user context within the Docker container. Running as a non-root user (e.g., `HERMES_DOCKER_USER=1000:1000`) significantly enhances security.
- `HERMES_SSH_PORT`: While the default SSH port is 22, many secure environments use custom ports. This variable allows operators to specify the correct port for the SSH backend.
- `HERMES_SSH_TIMEOUT`: Sets the connection timeout for SSH sessions, preventing the agent from hanging indefinitely if the remote server is unresponsive.

### 8.3 Gateway and Platform Integration Details

The messaging gateway transforms Hermes from a local CLI tool into a distributed, multi-platform agent. The environment variables configuring this gateway must be managed carefully to ensure secure and reliable communication.

- `HERMES_GATEWAY_POLLING_INTERVAL`: For platforms that do not support webhooks (e.g., some older Telegram bot implementations), this variable sets how often the agent polls for new messages.
- `HERMES_GATEWAY_WEBHOOK_URL`: For platforms that support webhooks, this variable defines the endpoint where the platform should send incoming messages.
- `HERMES_GATEWAY_WEBHOOK_PORT`: Specifies the port the internal webhook server should listen on.
- `HERMES_GATEWAY_WEBHOOK_SECRET`: A cryptographic secret used to verify the authenticity of incoming webhook requests, preventing spoofing attacks.

### 8.4 Observability and Telemetry Deep Dive

In production environments, monitoring the agent's performance and behavior is critical. Hermes integrates with Langfuse and other observability platforms, configured via environment variables.

- `LANGFUSE_RELEASE`: Tags the telemetry data with a specific release version, allowing operators to track performance changes across different versions of the agent or its configuration.
- `LANGFUSE_ENVIRONMENT`: Tags the data with the environment name (e.g., `production`, `staging`), enabling environment-specific dashboards and alerts.
- `HERMES_LOG_FORMAT`: Specifies the format of the log output. Options typically include `text` (for human readability) and `json` (for ingestion into log management systems like ELK or Splunk).
- `HERMES_LOG_FILE`: Directs the log output to a specific file instead of standard output, which is useful for long-running background processes.

## 9. Deep Dive: Config.yaml Schema and Advanced Customization

The `config.yaml` file is the heart of Hermes Agent's configuration. While the basic schema covers most use cases, advanced operators can leverage its full depth to customize the agent's behavior extensively.

### 9.1 Advanced Model Configuration

The `model` section of `config.yaml` supports advanced parameters that fine-tune the AI's generation process.

```yaml
model:
  provider: "openai"
  name: "gpt-4o"
  context_length: 128000
  temperature: 0.7
  top_p: 0.9 # Nucleus sampling parameter
  frequency_penalty: 0.2 # Penalizes new tokens based on their existing frequency in the text
  presence_penalty: 0.1 # Penalizes new tokens based on whether they appear in the text so far
  stop_sequences: ["User:", "Observation:"] # Custom sequences where the model should stop generating
  max_tokens: 4096 # The maximum number of tokens to generate in a single response
  failover:
    enabled: true
    secondary_provider: "anthropic"
    secondary_model: "claude-3-haiku-20240307"
    retry_attempts: 3 # Number of times to retry the primary provider before failing over
    retry_delay: 2 # Delay in seconds between retry attempts
```

### 9.2 Advanced Terminal Configuration

The `terminal` section can be configured to support complex execution environments, such as HPC clusters or specialized cloud sandboxes.

```yaml
terminal:
  backend: "singularity" # High-Performance Computing (HPC) container backend
  cwd: "/scratch/workspace"
  timeout: 3600 # 1 hour timeout for long-running scientific computations
  
  singularity:
    image: "/opt/images/ubuntu-22.04.sif"
    bind_mounts:
      - "/data/datasets:/datasets:ro" # Read-only mount for datasets
      - "/scratch/output:/output:rw" # Read-write mount for results
    extra_args: "--nv" # Enable NVIDIA GPU support in Singularity
    
  modal:
    app_name: "hermes-sandbox"
    gpu: "A10G" # Request a specific GPU type in the Modal cloud
    cpu: 4.0
    memory: 16384 # 16 GB RAM
```

### 9.3 Advanced MCP Servers Configuration

The Model Context Protocol (MCP) allows Hermes to integrate with external tools seamlessly. The configuration can handle complex authentication and routing scenarios.

```yaml
mcp_servers:
  - name: "enterprise_database"
    command: "node"
    args: ["/opt/mcp/db_server.js"]
    env:
      DB_HOST: "db.internal.corp"
      DB_PORT: "5432"
    url: "http://localhost:8081"
    headers:
      Authorization: "Bearer ${ENTERPRISE_DB_TOKEN}"
      X-Tenant-ID: "tenant-42"
    tool_filter:
      - "query_database"
      - "list_tables"
    timeout: 60 # Custom timeout for database queries
    retry_policy:
      max_retries: 2
      backoff_factor: 1.5
```

### 9.4 Advanced Skills Configuration

The skills system is highly customizable, allowing operators to define how skills are discovered, loaded, and executed.

```yaml
skills:
  paths:
    - "~/.hermes/skills"
    - "/var/lib/hermes/shared_skills" # A shared directory for team-wide skills
  config:
    auto_update: true
    fallback_enabled: true
    hub_url: "https://agentskills.io/api/v1"
    strict_mode: true # Only load skills that have a valid cryptographic signature
    max_execution_time: 120 # Maximum time a skill is allowed to run
    allowed_env_vars: # Restrict which environment variables skills can access
      - "GITHUB_TOKEN"
      - "AWS_REGION"
```

### 9.5 Advanced Memory Configuration

The memory system can be tuned to balance context retention with token usage efficiency.

```yaml
memory:
  provider: "sqlite"
  char_limits:
    memory_md: 4000 # Increased limit for complex tasks
    user_md: 2000
  compression:
    enabled: true
    threshold: 0.85
    strategy: "conversation" # Compress the conversation history rather than the trajectory
    preserve_system_prompt: true # Ensure the system prompt is never compressed
    summarization_model: "gpt-4o-mini" # Use a smaller, faster model for summarization tasks
  storage:
    db_path: "~/.hermes/memories/hermes.db"
    backup_interval: 86400 # Backup the database every 24 hours
    max_backups: 7 # Keep the last 7 backups
```

### 9.6 Advanced Gateway Configuration

The gateway configuration can handle complex routing and session management for multi-user environments.

```yaml
gateway:
  enabled: true
  platforms:
    - "slack"
    - "matrix"
  home_channels:
    slack: "C1234567890"
    matrix: "!roomid:matrix.org"
  session_reset:
    timeout: 7200 # 2 hours
    notify_on_reset: true # Send a message to the user when their session is reset
  delivery_preferences:
    format: "markdown"
    chunk_size: 3500
    rate_limit:
      messages_per_minute: 20 # Prevent the agent from spamming the platform
      burst_size: 5
  authorization:
    mode: "allowlist" # Only allow specific users to interact with the agent
    allowed_users:
      - "U12345678" # Slack user ID
      - "@alice:matrix.org" # Matrix user ID
```

### 9.7 Advanced Cron Configuration

The cron system can be configured to handle complex scheduling and error recovery.

```yaml
cron:
  enabled: true
  jobs_file: "~/.hermes/cron/jobs.json"
  timezone: "America/New_York"
  delivery_platform: "slack"
  error_handling:
    retry_on_failure: true
    max_retries: 3
    notify_on_failure: true # Send an alert if a cron job fails repeatedly
    failure_channel: "C0987654321" # A dedicated channel for cron alerts
  execution:
    max_concurrent_jobs: 5 # Prevent cron jobs from overwhelming the system
    timeout: 600 # Maximum execution time for a single cron job
```

### 9.8 Advanced Browser Configuration

The browser automation tools can be fine-tuned for specific web scraping or testing scenarios.

```yaml
browser:
  headless: true
  viewport:
    width: 1920
    height: 1080
  user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
  timeout: 30000
  proxy:
    server: "http://proxy.corp.internal:8080"
    bypass_list: ["localhost", "127.0.0.1", "*.corp.internal"]
  security:
    ignore_https_errors: false # Enforce strict SSL validation
    disable_javascript: false # Allow JavaScript execution
    block_ads: true # Improve performance by blocking ad networks
```

### 9.9 Advanced Toolset Configuration

The toolset configuration allows operators to define granular access controls and execution policies for individual tools.

```yaml
toolset:
  enabled_categories:
    - "web"
    - "terminal"
    - "browser"
    - "media"
  disabled_tools:
    - "computer_use"
    - "execute_code"
  tool_policies:
    terminal:
      require_approval: true # Always require approval for terminal commands
      allowed_commands: ["ls", "cat", "grep", "echo"] # Restrict the terminal to safe commands
    web_search:
      max_results: 10 # Limit the number of search results returned
      safe_search: true # Enable safe search filtering
    image_generate:
      provider: "openai" # Use a specific provider for image generation
      max_resolutions: ["1024x1024"] # Restrict the allowed image resolutions
```

## 10. Troubleshooting and Diagnostics

Operating a complex AI agent like Hermes in production environments inevitably leads to challenges. The CLI provides a suite of tools and commands designed to diagnose and resolve issues quickly.

### 10.1 Using `hermes doctor` Effectively

The `hermes doctor` command is the first line of defense when troubleshooting. It performs a comprehensive health check of the system.

**Common Issues Identified by `hermes doctor`:**
- **Missing Dependencies**: If a required Python package is missing or the wrong version is installed, `hermes doctor` will flag it. This is common when upgrading Hermes or moving to a new environment. The solution is usually to run `pip install -r requirements.txt` to ensure exact-pinned dependencies are met.
- **Invalid API Keys**: The doctor checks the validity of the configured API keys by making a lightweight request to the provider. If the key is invalid, expired, or lacks sufficient permissions, the doctor will report an error.
- **Backend Unreachable**: If the Docker daemon is not running or the SSH server is unreachable, the doctor will fail the backend check. Operators must ensure the underlying infrastructure is operational.
- **Configuration Syntax Errors**: If the `config.yaml` file contains invalid YAML syntax, the doctor will pinpoint the line number and error type.

### 10.2 Debugging with `--debug`

When `hermes doctor` reports a healthy system but the agent behaves unexpectedly, the `--debug` flag is essential. It provides a verbose trace of the agent's internal state.

**Key Areas to Inspect in Debug Output:**
- **Prompt Assembly**: Verify that the `prompt_builder.py` is correctly assembling the system prompt. Check that the `SOUL.md`, `MEMORY.md`, and `USER.md` snapshots are included and that the context files (e.g., `.cursorrules`) are loaded.
- **Tool Dispatch**: Ensure that the agent is correctly identifying and dispatching tools. If the agent attempts to use a tool that is disabled or misconfigured, the debug output will show the failure reason.
- **API Responses**: Inspect the raw API responses from the provider. Look for rate limit errors (HTTP 429), context length exceeded errors (HTTP 400), or server errors (HTTP 500). The debug output will also show the exact token usage, helping operators optimize their prompts.

### 10.3 Log Analysis

For background processes, such as the messaging gateway or cron jobs, operators must rely on log files. Hermes logs are typically stored in `~/.hermes/logs/`.

**Log Analysis Techniques:**
- **Grep for Errors**: Use `grep -i error ~/.hermes/logs/hermes.log` to quickly identify critical issues.
- **Trace Session IDs**: Each log entry is tagged with a session ID. Operators can use `grep <session_id> ~/.hermes/logs/hermes.log` to trace the complete lifecycle of a specific interaction.
- **Monitor Token Usage**: Log files contain detailed token usage statistics. Operators can parse these logs to monitor costs and identify inefficient interactions.

### 10.4 Common Troubleshooting Scenarios

**Scenario 1: The Agent is Stuck in a Loop**
- **Symptom**: The agent repeatedly executes the same tool or generates the same response.
- **Solution**: Use the `/stop` slash command to interrupt the agent. Then, use `/compact` to compress the context, as loops are often caused by the model becoming confused by a long, repetitive conversation history. If the issue persists, use `/new` to start a fresh session.

**Scenario 2: Docker Backend Fails to Start**
- **Symptom**: The agent reports an error when attempting to execute a terminal command.
- **Solution**: Check the Docker daemon status (`systemctl status docker`). Ensure the `hermes_data` volume exists and has the correct permissions. Verify that the `docker_image` specified in `config.yaml` is available locally or can be pulled from a registry.

**Scenario 3: Gateway Messages are Delayed or Dropped**
- **Symptom**: Users report that the agent is slow to respond or misses messages entirely.
- **Solution**: Check the gateway logs for rate limit errors from the messaging platform (e.g., Telegram's HTTP 429 Too Many Requests). Adjust the `rate_limit` settings in `config.yaml` to comply with the platform's restrictions. Ensure the `HERMES_GATEWAY_POLLING_INTERVAL` is not set too high.

## 11. Migration from OpenClaw

For users upgrading from OpenClaw, the predecessor to Hermes Agent, the `hermes claw migrate` command provides a streamlined migration path. This section details the migration process and the changes operators can expect.

### 11.1 The Migration Process

The migration process involves translating the OpenClaw configuration files, memory state, and session history into the new Hermes format.

1. **Backup**: Before migrating, operators should create a backup of their OpenClaw data (`cp -r ~/.openclaw ~/.openclaw_backup`).
2. **Dry Run**: Run `hermes claw migrate --dry-run` to simulate the migration. This will output a report detailing which files will be modified and any potential conflicts.
3. **Execute Migration**: Run `hermes claw migrate` to perform the actual migration. The command will translate the `config.json` (OpenClaw) to `config.yaml` (Hermes), migrate the SQLite database schema, and update the memory files.

### 11.2 Key Differences Between OpenClaw and Hermes

Operators familiar with OpenClaw will notice several significant changes in Hermes Agent:

- **Configuration Format**: Hermes uses YAML for its configuration file (`config.yaml`), whereas OpenClaw used JSON (`config.json`). YAML provides a more readable and flexible format, especially for complex nested structures.
- **Terminal Backends**: Hermes introduces a modular terminal backend system, supporting Docker, SSH, Singularity, and cloud sandboxes (Modal, Daytona). OpenClaw relied primarily on local execution.
- **Skills System**: Hermes features a robust procedural memory system (Skills), allowing the agent to learn and improve over time. This replaces OpenClaw's static script execution model.
- **Messaging Gateway**: The Hermes gateway supports over 20 platforms natively, whereas OpenClaw required third-party adapters for most platforms.
- **Context Compression**: Hermes includes advanced context compression algorithms (trajectory and conversation), ensuring the agent can maintain long-running sessions without exceeding the model's context limit.

### 11.3 Post-Migration Verification

After migrating, operators should verify the integrity of the system:

1. Run `hermes doctor` to ensure all dependencies and configurations are valid.
2. Run `hermes config show` to verify that the settings were translated correctly.
3. Start an interactive session (`hermes chat`) and use the `/memory` command to confirm that the `MEMORY.md` and `USER.md` files were migrated successfully.
4. Test the terminal backend by asking the agent to execute a simple command (e.g., `ls -la`).

## 12. Extending the CLI

The Hermes Agent CLI is designed to be extensible, allowing developers to add custom commands and functionalities. This is achieved through the plugin system and the underlying `fire` library.

### 12.1 Creating Custom CLI Commands

Developers can add custom commands by creating a new Python module in the `cli/plugins/` directory. The module must define a class that exposes the desired methods.

**Example: Custom Status Command**
```python
# cli/plugins/custom_status.py

class CustomStatusPlugin:
    def __init__(self, agent):
        self.agent = agent

    def system_health(self):
        """Displays a detailed system health report."""
        import psutil
        cpu = psutil.cpu_percent()
        memory = psutil.virtual_memory().percent
        disk = psutil.disk_usage('/').percent
        
        print(f"CPU Usage: {cpu}%")
        print(f"Memory Usage: {memory}%")
        print(f"Disk Usage: {disk}%")
        print(f"Active Sessions: {len(self.agent.session_manager.get_active_sessions())}")
```

Once the plugin is created, it must be registered in the `cli/registry.py` file. The new command will then be available via the CLI (e.g., `hermes custom_status system_health`).

### 12.2 Integrating with External Tools

The CLI can be integrated with external tools and scripts to create powerful workflows. For example, operators can use `jq` to parse the JSON output of the `/export` command, or use `awk` to analyze the log files.

**Example: Analyzing Token Usage**
```bash
grep "Token usage" ~/.hermes/logs/hermes.log | awk '{total += $10} END {print "Total Tokens Used: " total}'
```

This simple script calculates the total number of tokens used by parsing the log file, providing valuable insights into the agent's operational costs.

## 13. Final Thoughts

The Hermes Agent CLI is a comprehensive and powerful interface that provides operators with the tools necessary to manage, configure, and extend the agent in production environments. By understanding the core commands, advanced configuration options, and troubleshooting techniques detailed in this reference guide, operators can ensure that their Hermes Agent deployments are secure, efficient, and highly capable. Whether running locally, in a Docker container, or across a distributed SSH cluster, the CLI is the key to unlocking the full potential of the Hermes Agent.
