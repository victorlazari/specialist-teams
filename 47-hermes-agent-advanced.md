# Hermes Agent: Advanced Configuration and Deep Dive

Welcome to the advanced guide for Hermes Agent v0.14.0. This document serves as the definitive reference for operating Hermes Agent in production environments, covering advanced provider configurations, multi-model orchestration, deep skill development, background review systems, and complex backend setups.

## 1. Advanced Provider Configuration

In production environments, relying on a single model provider is a significant risk. Hermes Agent provides an advanced provider configuration system that supports automatic failover, credential pools, and auxiliary models to ensure high availability and optimal performance.

### Failover Mechanisms

The failover system in Hermes Agent is designed to seamlessly switch to alternative providers when the primary provider experiences downtime, rate limits, or other transient errors. This is configured in the `config.yaml` file under the `providers` section.

```yaml
providers:
  primary:
    name: openai
    model: gpt-4o
    timeout: 30
    retries: 3
  failover:
    - name: anthropic
      model: claude-3-5-sonnet-20240620
      timeout: 45
    - name: openrouter
      model: meta-llama/llama-3-70b-instruct
      timeout: 60
```

When a request to the primary provider fails, the agent's error classifier evaluates the `FailoverReason`. If the error is deemed recoverable (e.g., a 502 Bad Gateway or a 429 Too Many Requests), the agent automatically routes the request to the first failover provider. This process is transparent to the user and ensures continuous operation.

### Credential Pools

Managing API keys for multiple providers can be challenging, especially in distributed deployments. Hermes Agent introduces the concept of Credential Pools, which allow you to define multiple API keys for a single provider. The agent will round-robin through these keys or select them based on specific criteria, such as remaining quota or rate limit status.

```yaml
credential_pools:
  openai:
    strategy: round_robin
    keys:
      - env: OPENAI_API_KEY_1
      - env: OPENAI_API_KEY_2
      - env: OPENAI_API_KEY_3
```

This configuration ensures that if one API key hits a rate limit, the agent can immediately switch to another key within the pool, maximizing throughput and minimizing downtime.

### Auxiliary Models

Hermes Agent allows you to define auxiliary models for specific tasks. For example, you might use a large, capable model for complex reasoning and a smaller, faster model for simple text extraction or summarization.

```yaml
auxiliary_models:
  summarization:
    provider: groq
    model: llama3-8b-8192
  embedding:
    provider: openai
    model: text-embedding-3-small
```

By routing specific tasks to auxiliary models, you can optimize both cost and latency without sacrificing the quality of the primary agent loop.

## 2. Multi-Model Orchestration

Hermes Agent excels in environments where multiple models must collaborate to solve complex problems. The Mixture of Agents (MoA) tool is a prime example of this capability.

### Mixture of Agents Tool

The Mixture of Agents tool allows Hermes Agent to delegate sub-tasks to different models and synthesize their outputs. This is particularly useful for tasks that require diverse perspectives or specialized knowledge.

```python
# Example of invoking the Mixture of Agents tool
response = await agent.execute_tool(
    "mixture_of_agents",
    task="Analyze the impact of quantum computing on cryptography.",
    models=["gpt-4o", "claude-3-5-sonnet-20240620", "gemini-1.5-pro"],
    synthesizer="gpt-4o"
)
```

In this example, the agent sends the task to three different models simultaneously. Once all models have responded, the `synthesizer` model reviews their outputs and generates a comprehensive, unified response. This approach leverages the strengths of multiple models, mitigating individual biases and hallucinations.

## 3. Advanced Skills Development

Skills in Hermes Agent are procedural memories that allow the agent to learn and improve over time. Advanced skill development involves configuring settings, environment variables, conditional activation, and fallback mechanisms.

### Config Settings and Environment Variables

Skills can define their own configuration settings and required environment variables in their `SKILL.md` file. This ensures that the skill has all the necessary resources to execute successfully.

```markdown
---
name: advanced_data_analysis
version: 1.2.0
requires_env:
  - DATABASE_URL
  - REDIS_HOST
config:
  max_rows: 10000
  timeout: 120
---
```

When the agent loads this skill, it verifies that the required environment variables are present and applies the configuration settings. If any requirements are missing, the skill is disabled, and a warning is logged.

### Conditional Activation

To optimize context window usage, skills can be conditionally activated based on the current task or available toolsets. This is controlled by the `requires_toolsets` and `fallback_for_toolsets` directives.

```markdown
---
name: python_code_execution
requires_toolsets:
  - execute_code
fallback_for_toolsets:
  - terminal
---
```

In this example, the `python_code_execution` skill is only activated if the `execute_code` toolset is available. If the `execute_code` toolset is missing, but the `terminal` toolset is available, the skill can act as a fallback, using the terminal to execute Python scripts.

### Fallback Skills

Fallback skills are designed to handle situations where a primary skill fails or is unavailable. They provide a safety net, ensuring that the agent can still accomplish its goals, albeit perhaps less efficiently.

```markdown
---
name: manual_web_search
fallback_for_skills:
  - automated_web_scraper
---
```

If the `automated_web_scraper` skill encounters an error (e.g., due to a site structure change), the agent can seamlessly switch to the `manual_web_search` skill, which might use a different approach, such as interacting with a search engine via the browser tool.

## 4. Background Review System

Hermes Agent features a sophisticated background review system that operates asynchronously, analyzing the agent's performance and suggesting improvements.

### Memory Nudges

The background review system continuously monitors the agent's interactions and updates its memory. If the agent repeatedly struggles with a specific type of task, the review system can inject a "memory nudge" into the `MEMORY.md` file.

```markdown
# MEMORY.md
- The user prefers concise answers.
- **Nudge**: When generating Python code, always include type hints and docstrings.
```

These nudges guide the agent's future behavior, helping it adapt to the user's preferences and avoid repeating mistakes.

### Skill Improvement

In addition to memory nudges, the background review system can analyze the execution of skills and suggest improvements. If a skill frequently fails or produces suboptimal results, the review system can generate a revised version of the skill and propose it to the user for approval.

This self-improving capability is a core feature of Hermes Agent, allowing it to evolve and become more effective over time.

## 5. Context Engine Plugins and Memory Providers

Managing context is critical for long-running agent sessions. Hermes Agent supports context engine plugins and advanced memory providers, such as Honcho, to handle large volumes of information efficiently.

### Honcho Integration

Honcho is a powerful context engine that integrates seamlessly with Hermes Agent. It provides a static block of context that is injected into the prompt assembly pipeline, ensuring that the agent always has access to essential information.

```yaml
memory:
  provider: honcho
  honcho_api_key: ${HONCHO_API_KEY}
  app_id: hermes_production
```

By leveraging Honcho, Hermes Agent can maintain a deep understanding of the user's environment, preferences, and ongoing projects, even across multiple sessions and platforms.

## 6. Iteration Budget System

To prevent runaway processes and manage costs, Hermes Agent implements an Iteration Budget System. This system limits the number of iterations the agent can perform within a single conversation loop.

```yaml
agent:
  max_iterations: 15
  budget_exceeded_action: ask_user
```

If the agent reaches the maximum number of iterations without completing its task, it triggers the `budget_exceeded_action`. This can be configured to ask the user for permission to continue, terminate the task, or switch to a cheaper model.

This system is essential for production deployments, where unbounded agent loops can lead to significant financial costs and resource exhaustion.

## 7. Error Classification and Retry Logic

Robust error handling is a hallmark of production-ready systems. Hermes Agent features an advanced error classifier and retry logic to handle transient failures gracefully.

### Failover Reasons

The error classifier categorizes API errors into specific `FailoverReason` enums. This allows the agent to make intelligent decisions about how to handle the error.

- `RATE_LIMIT`: The provider's rate limit has been exceeded.
- `CONTEXT_LENGTH_EXCEEDED`: The prompt is too large for the model's context window.
- `SERVER_ERROR`: The provider is experiencing internal issues (e.g., 500, 502, 503).
- `UNAUTHORIZED`: The API key is invalid or expired.

### Jittered Backoff

When a transient error occurs (e.g., `RATE_LIMIT` or `SERVER_ERROR`), the agent employs a jittered backoff strategy before retrying the request. This prevents the "thundering herd" problem, where multiple agents retry simultaneously and overwhelm the provider.

```python
import random
import time

def jittered_backoff(attempt, base_delay=1.0, max_delay=60.0):
    delay = min(max_delay, base_delay * (2 ** attempt))
    jitter = random.uniform(0, 0.1 * delay)
    time.sleep(delay + jitter)
```

This sophisticated retry logic ensures that Hermes Agent remains resilient in the face of network instability and provider outages.

## 8. Advanced Docker Backend

For tasks that require execution in an isolated environment, Hermes Agent provides an advanced Docker backend. This backend supports GPU passthrough, persistent volumes, and fine-grained resource limits.

### GPU Passthrough

When executing tasks that require hardware acceleration (e.g., training machine learning models or running local inference), the Docker backend can be configured to pass through GPUs to the container.

```yaml
terminal:
  backend: docker
  docker_image: nvidia/cuda:12.2.0-base-ubuntu22.04
  docker_extra_args:
    - "--gpus"
    - "all"
```

This configuration allows the agent to leverage the full power of the host machine's GPUs, significantly accelerating compute-intensive tasks.

### Persistent Containers and Volumes

To maintain state across multiple executions, the Docker backend supports persistent containers and volumes. This is crucial for tasks that involve compiling code, downloading large datasets, or maintaining a complex environment.

```yaml
terminal:
  backend: docker
  container_persistent: true
  docker_volumes:
    - /host/data:/container/data
```

By using persistent containers and volumes, Hermes Agent can pick up exactly where it left off, avoiding the overhead of rebuilding the environment for every task.

### Resource Limits

In multi-tenant environments, it is essential to limit the resources consumed by the agent's containers. The Docker backend allows you to specify CPU, memory, and disk limits.

```yaml
terminal:
  backend: docker
  container_cpu: "2.0"
  container_memory: "4g"
  container_disk: "20g"
```

These limits ensure that the agent's activities do not impact the performance of other applications running on the same host.

## 9. Gateway Hooks and Extension Points

The Gateway system in Hermes Agent is highly extensible, allowing developers to intercept and modify messages as they flow through the system. This is achieved through Gateway Hooks.

### Pre-Message and Post-Message Hooks

Gateway Hooks can be registered to execute before a message is processed by the agent (`pre_message`) or after the agent has generated a response (`post_message`).

```python
@gateway.hook("pre_message")
async def log_incoming_message(event: MessageEvent):
    logger.info(f"Received message from {event.platform}: {event.content}")

@gateway.hook("post_message")
async def append_signature(response: str) -> str:
    return response + "

---
Sent by Hermes Agent"
```

These hooks provide a powerful mechanism for implementing custom logging, content filtering, and message formatting logic without modifying the core agent code.

## 10. Cross-Session Message Mirroring

In complex deployments, users may interact with Hermes Agent across multiple platforms (e.g., Slack, Telegram, and the CLI). Cross-Session Message Mirroring ensures that the user's context is synchronized across all these platforms.

When a user sends a message on Slack, the Gateway system can mirror that message to the user's active CLI session. This allows the user to seamlessly transition between platforms without losing the thread of the conversation.

```yaml
gateway:
  mirroring:
    enabled: true
    sync_platforms:
      - slack
      - telegram
      - cli
```

This feature is particularly useful for long-running tasks, where a user might initiate a task on their desktop CLI and monitor its progress on their mobile device via Telegram.

## 11. Credential Pool and Credential Sources

As mentioned earlier, Credential Pools allow the agent to manage multiple API keys for a single provider. However, managing these keys securely is equally important. Hermes Agent supports multiple Credential Sources, allowing you to retrieve keys from secure vaults.

### Supported Credential Sources

- **Environment Variables**: The default source, suitable for simple deployments.
- **AWS Secrets Manager**: Retrieves keys from AWS Secrets Manager, ideal for cloud deployments.
- **HashiCorp Vault**: Integrates with HashiCorp Vault for enterprise-grade secret management.

```yaml
credential_sources:
  - type: aws_secrets_manager
    region: us-east-1
    secret_id: hermes_production_keys
```

By integrating with secure credential sources, Hermes Agent ensures that sensitive API keys are never hardcoded in configuration files or exposed in plain text.

## 12. Plugin System

Hermes Agent features a robust plugin system that allows developers to extend its functionality with third-party integrations.

### Langfuse Observability

Observability is critical for understanding and optimizing agent performance. The Langfuse plugin integrates Hermes Agent with the Langfuse observability platform, providing detailed insights into token usage, latency, and agent behavior.

```yaml
plugins:
  langfuse:
    enabled: true
    public_key: ${LANGFUSE_PUBLIC_KEY}
    secret_key: ${LANGFUSE_SECRET_KEY}
    host: https://cloud.langfuse.com
```

With Langfuse enabled, every agent interaction is logged and analyzed, allowing you to identify bottlenecks, track costs, and improve the overall quality of the agent's responses.

### Spotify Integration

The plugin system is not limited to developer tools. The Spotify plugin, for example, allows Hermes Agent to interact with the user's Spotify account, controlling playback, searching for tracks, and managing playlists.

```yaml
plugins:
  spotify:
    enabled: true
    client_id: ${SPOTIFY_CLIENT_ID}
    client_secret: ${SPOTIFY_CLIENT_SECRET}
```

This demonstrates the versatility of the plugin system, enabling Hermes Agent to act as a personal assistant across a wide range of domains.

## 13. Advanced Prompt Caching

Prompt caching is a crucial optimization technique for reducing latency and costs, especially when dealing with large context windows. Hermes Agent supports advanced prompt caching mechanisms, including Anthropic's Cache Control.

### Anthropic Cache Control

When using Anthropic models (e.g., Claude 3.5 Sonnet), Hermes Agent can leverage the Cache Control API to cache specific parts of the prompt, such as the system instructions or large context files.

```python
# Example of applying cache control to a context file
context_block = {
    "type": "text",
    "text": large_document_content,
    "cache_control": {"type": "ephemeral"}
}
```

By caching static parts of the prompt, the agent can significantly reduce the number of tokens processed for each request, leading to faster response times and lower API costs.

## 14. Codex Responses Adapter

For environments that require strict adherence to specific output formats, Hermes Agent provides the Codex Responses Adapter. This adapter intercepts the agent's output and reformats it to match a predefined schema.

### Schema Enforcement

The Codex Responses Adapter uses a combination of prompt engineering and post-processing to ensure that the agent's output strictly conforms to the required schema.

```yaml
adapters:
  codex_responses:
    enabled: true
    schema:
      type: object
      properties:
        status:
          type: string
          enum: [success, failure]
        data:
          type: object
      required: [status, data]
```

This is particularly useful when integrating Hermes Agent with legacy systems or automated pipelines that expect data in a specific format.

## 15. LSP Integration (Language Server Protocol)

To enhance its coding capabilities, Hermes Agent integrates with the Language Server Protocol (LSP). This allows the agent to leverage the same intelligent code completion, error checking, and refactoring tools used by modern IDEs.

### LSP Features

When the agent is editing code, it can query the LSP server for information about the codebase.

- **Go to Definition**: The agent can navigate to the definition of a function or class to understand its implementation.
- **Find References**: The agent can find all references to a specific symbol, ensuring that refactoring operations are safe and comprehensive.
- **Diagnostics**: The agent can receive real-time feedback on syntax errors and type mismatches, allowing it to correct mistakes before executing the code.

```yaml
lsp:
  enabled: true
  servers:
    python: pylsp
    typescript: typescript-language-server
```

By integrating with LSP, Hermes Agent transforms from a simple text generator into a sophisticated, context-aware pair programmer.

## 16. Image Routing and Generation Backends

Hermes Agent supports multiple image generation backends, allowing it to create high-quality visual content. The Image Routing system intelligently selects the best backend based on the user's request and the available resources.

### Supported Backends

- **FAL**: A high-performance inference platform, ideal for generating images quickly and efficiently.
- **OpenAI (DALL-E 3)**: Provides high-quality, stylized images with excellent prompt adherence.
- **xAI (Grok Vision)**: Integrates with xAI's vision models for advanced image generation and analysis.

```yaml
image_routing:
  default_backend: fal
  rules:
    - condition: "requires high realism"
      backend: openai
    - condition: "requires fast generation"
      backend: fal
```

The Image Routing system evaluates the user's prompt and applies the configured rules to select the most appropriate backend. This ensures that the agent always delivers the best possible results, regardless of the specific requirements of the task.

---

## Conclusion

Hermes Agent v0.14.0 is a powerful, highly configurable AI agent designed for production environments. By leveraging its advanced provider configurations, multi-model orchestration capabilities, and sophisticated backend systems, you can build resilient, scalable, and highly effective AI-driven applications. Whether you are deploying Hermes Agent in a cloud environment, integrating it with enterprise systems, or using it as a personal assistant, the advanced features detailed in this document provide the tools you need to succeed.



## 17. Deep Dive into the Error Classification System

The error classification system in Hermes Agent is not just a simple switch statement; it is a sophisticated engine designed to parse, understand, and react to a wide variety of failure modes across dozens of different API providers. Each provider has its own unique way of formatting error messages, HTTP status codes, and rate limit headers. The Hermes Agent error classifier normalizes these disparate signals into a unified `FailoverReason` enum.

### Normalization Pipeline

When an HTTP request to a provider fails, the response is passed through the normalization pipeline. This pipeline consists of several stages:

1.  **HTTP Status Code Analysis**: The most basic level of classification. A 429 is almost universally a rate limit, while a 500 is a server error. However, some providers return 400 Bad Request for context length errors, requiring deeper inspection.
2.  **Header Inspection**: The pipeline examines headers like `Retry-After`, `x-ratelimit-reset`, and `x-ratelimit-remaining`. If these headers are present, the error is immediately classified as a `RATE_LIMIT`, and the backoff duration is extracted directly from the headers, overriding the default jittered backoff calculation.
3.  **Body Parsing**: If the status code and headers are inconclusive, the pipeline parses the JSON body of the error response. It uses a set of provider-specific regex patterns to identify common error strings. For example, OpenAI might return `"error": {"code": "context_length_exceeded"}`, while Anthropic might return `"type": "error", "error": {"type": "invalid_request_error", "message": "prompt is too long"}`.
4.  **Fallback Heuristics**: If all else fails, the pipeline uses heuristics based on the size of the prompt and the time taken for the request to fail. A request that fails immediately with a large payload is likely a context length issue, while a request that times out after 60 seconds is likely a server-side timeout.

### Handling Context Length Exceeded

When the error classifier identifies a `CONTEXT_LENGTH_EXCEEDED` error, the agent does not simply failover to another provider. Instead, it triggers the Context Compression engine. The agent will attempt to compress the conversation history, summarize older messages, or drop less relevant context files before retrying the request with the same provider. Only if compression fails to reduce the prompt size below the limit will the agent failover to a provider with a larger context window (e.g., switching from a 32k model to a 128k model).

## 18. The Anatomy of a Skill: Procedural Memory in Action

Skills in Hermes Agent are more than just static scripts; they represent the agent's procedural memory. When the agent learns a new way to solve a problem, it can encode that knowledge into a new skill, which is then saved to disk and loaded in future sessions.

### The SKILL.md Structure

A skill is defined by a directory containing a `SKILL.md` file and any associated scripts or assets. The `SKILL.md` file is the heart of the skill, containing metadata, configuration, and the actual instructions for the agent.

```markdown
---
name: advanced_git_bisect
version: 1.0.0
description: Automates the process of finding the commit that introduced a bug using git bisect.
author: Hermes Agent
created_at: 2023-10-27T10:00:00Z
requires_toolsets:
  - terminal
  - execute_code
requires_env:
  - GITHUB_TOKEN
config:
  max_steps: 20
  test_script_timeout: 300
---

# Advanced Git Bisect Skill

## Overview
This skill allows you to automatically find the commit that introduced a bug by running a test script across the commit history.

## Instructions
1.  **Identify the Bad Commit**: Ask the user for the commit hash where the bug is known to exist (or use `HEAD`).
2.  **Identify a Good Commit**: Ask the user for a commit hash where the bug is known NOT to exist.
3.  **Create the Test Script**: Write a script (e.g., `test.sh`) that returns exit code 0 if the bug is absent, and exit code 1 if the bug is present.
4.  **Start Bisect**: Run `git bisect start <bad_commit> <good_commit>`.
5.  **Run Bisect**: Run `git bisect run ./test.sh`.
6.  **Analyze Results**: Once the bisect is complete, report the offending commit to the user and run `git bisect reset`.

## Error Handling
- If the test script times out, abort the bisect and inform the user.
- If the repository is in a dirty state, stash the changes before starting the bisect.
```

### Skill Provenance and Trust

Because skills can execute arbitrary code, security is paramount. Hermes Agent tracks the provenance of every skill. Skills downloaded from the official Skills Hub (agentskills.io) are cryptographically signed and verified upon installation. Skills created by the agent itself are marked as "self-generated" and may require user approval before they can be executed, depending on the agent's security configuration.

## 19. Advanced Memory Management: The Bounded Curation Strategy

The `MEMORY.md` file is the agent's personal scratchpad, but it has a strict character limit (typically 2,200 characters, or roughly 800 tokens). This limit ensures that the memory snapshot remains small enough to be efficiently cached by the provider's prefix caching mechanisms.

To manage this limited space, Hermes Agent employs a Bounded Curation Strategy.

### The Curation Process

When the agent wants to add a new piece of information to its memory, it uses the `memory` tool. If the addition would exceed the character limit, the memory manager intercepts the request and forces the agent to curate its existing memory.

The agent is presented with its current memory and the new information it wants to add. It must then rewrite the memory, summarizing older points, removing obsolete information, and integrating the new data, all while staying under the limit.

```python
# Internal representation of the curation prompt
curation_prompt = f"""
Your memory is full. You must add the following new information:
{new_information}

Here is your current memory:
{current_memory}

Rewrite your memory to include the new information. You must remove or summarize older, less important information to stay under the {char_limit} character limit.
"""
```

This forced curation ensures that the agent's memory remains relevant and concise, preventing it from becoming a bloated, unstructured mess.

## 20. The Kanban Orchestration System

For complex, multi-step projects, a single agent loop is often insufficient. Hermes Agent includes a Kanban Orchestration System that allows it to manage tasks across multiple sub-agents, tracking progress on a virtual Kanban board.

### Board Structure

The Kanban board is stored in the session database and consists of standard columns: `To Do`, `In Progress`, `Review`, and `Done`.

### Task Delegation

When the primary agent identifies a complex project, it breaks it down into smaller tasks and adds them to the `To Do` column. It can then use the `delegate_task` tool to spawn sub-agents, assigning each sub-agent a specific task from the board.

```python
# Example of delegating a task
await agent.execute_tool(
    "delegate_task",
    task_id="TASK-102",
    description="Implement the user authentication API endpoints.",
    context_files=["architecture.md", "database_schema.sql"],
    model="claude-3-5-sonnet-20240620"
)
```

### Crash Recovery

The Kanban system is designed for resilience. If a sub-agent crashes or times out, the task is automatically moved back to the `To Do` column, and the primary agent is notified. The primary agent can then reassign the task, perhaps providing additional context or selecting a different model. This ensures that complex projects can recover from individual failures and run to completion.

## 21. Deep Dive into the Prompt Assembly Pipeline

The prompt assembly pipeline is the heart of Hermes Agent. It is responsible for gathering all the necessary context, formatting it correctly, and presenting it to the model. The pipeline consists of 10 distinct layers, each serving a specific purpose.

### Layer 1: Agent Identity (SOUL.md)

The foundation of the prompt is the `SOUL.md` file. This file defines the agent's core identity, its overarching goals, and its fundamental rules of engagement. It sets the tone for the entire interaction.

### Layer 2: Tool-Aware Behavior Guidance

This layer dynamically generates instructions based on the tools currently available to the agent. If the `browser` toolset is active, this layer includes specific guidance on how to navigate pages, handle popups, and extract information. If the `terminal` toolset is active, it includes rules about command safety and formatting.

### Layer 3: Honcho Static Block

If the Honcho context engine is enabled, this layer injects the static context retrieved from the Honcho API. This might include enterprise-wide policies, project-specific guidelines, or user preferences stored in the Honcho database.

### Layer 4: Optional System Message

This layer allows for dynamic, session-specific system messages to be injected. For example, if the user started the session with a specific directive (e.g., "Act as a senior database administrator"), that directive is placed here.

### Layer 5: Frozen MEMORY Snapshot

The agent's personal notes from `MEMORY.md` are injected here. The snapshot is "frozen" at the start of the session, meaning it does not change during the conversation loop unless explicitly updated by the agent. This freezing is crucial for maximizing prefix cache hit rates.

### Layer 6: Frozen USER Profile Snapshot

Similar to the memory snapshot, the `USER.md` profile is injected here. This contains information about the user's preferences, environment, and past interactions.

### Layer 7: Skills Index

This layer lists all the currently active skills. It provides the agent with a menu of its procedural memories, allowing it to know what specialized workflows it can execute.

### Layer 8: Context Files

Any files explicitly added to the context (e.g., `AGENTS.md`, `.cursorrules`, or files added via the CLI) are injected here. The pipeline ensures that these files are formatted clearly, often using XML-like tags to separate different documents.

### Layer 9: Timestamp and Session ID

Providing the current timestamp and session ID gives the agent temporal awareness and allows it to reference the specific session in its logs or tool calls.

### Layer 10: Platform Hint

The final layer provides a hint about the platform the user is interacting from (e.g., "The user is communicating via Slack"). This allows the agent to tailor its formatting (e.g., using Slack-specific markdown) and tone appropriately.

## 22. The Tirith Security Module

Security is a primary concern when running autonomous agents. The Tirith Security Module is a comprehensive security framework built into Hermes Agent, designed to prevent malicious actions and protect the host environment.

### DANGEROUS_PATTERNS System

At the core of Tirith is the `DANGEROUS_PATTERNS` system. This is a highly curated list of regular expressions that match potentially destructive commands.

```python
DANGEROUS_PATTERNS = [
    r"rm\s+-rf\s+/",          # Prevent root deletion
    r"mkfs",                  # Prevent formatting disks
    r"dd\s+if=.*of=/dev/",    # Prevent raw disk writes
    r">\s*/dev/sda",          # Prevent overwriting block devices
    r"chmod\s+-R\s+777\s+/",  # Prevent recursive permission changes
    r"wget\s+.*\s+\|\s+sh",   # Prevent piping remote scripts to shell
]
```

Before any command is executed via the `terminal` tool, it is scanned against these patterns. If a match is found, the command is blocked, and the agent is notified of the security violation.

### Command Approval Flow

For commands that are not explicitly blocked but are still considered sensitive (e.g., modifying system configuration files or installing packages), Tirith enforces a Command Approval Flow.

When the agent attempts to execute a sensitive command, the execution is paused, and a request is sent to the user for approval. In the CLI, this appears as an interactive prompt. In the Gateway system, it sends a message to the user with "Approve" and "Deny" buttons. The agent remains suspended until the user responds.

### Container Bypass Logic

Tirith is intelligent enough to know when it is running in a fully isolated environment. If the agent is running inside a secure Docker container, a Singularity HPC environment, or a Modal cloud sandbox, Tirith can be configured to bypass certain checks. For example, running `rm -rf /` inside an ephemeral, unprivileged Docker container is harmless to the host, so Tirith may allow it (or at least downgrade it from a hard block to an approval request) if the `container_bypass` flag is enabled.

## 23. Supply Chain Security: The Mini Shai-Hulud Incident

Hermes Agent takes supply chain security extremely seriously, a policy instituted after the infamous "Mini Shai-Hulud" incident in the broader AI agent community, where a compromised transitive dependency allowed an attacker to exfiltrate API keys from thousands of agent deployments.

### Exact-Pinned Dependencies

To mitigate this risk, Hermes Agent uses exact-pinned dependencies. Every single package, including transitive dependencies, is pinned to a specific, audited version in the `requirements.txt` and `pyproject.toml` files.

```text
# requirements.txt
openai==2.24.0
python-dotenv==1.2.2
fire==0.7.1
httpx[socks]==0.28.1
rich==14.3.3
tenacity==9.1.4
pyyaml==6.0.3
```

The Hermes Agent development team employs automated tools to monitor these specific versions for vulnerabilities (using the OSV database). When a vulnerability is found, the dependency is updated, audited, and a new patch release of Hermes Agent is issued. Users are strongly advised never to run `pip install --upgrade` on the Hermes Agent environment, as this breaks the exact-pinning and introduces supply chain risk.

## 24. Advanced Gateway Configuration: Home Channels and Delivery Preferences

The Gateway system connects Hermes Agent to over 20 different messaging platforms. Advanced configuration of this system allows for complex routing and delivery behaviors.

### Home Channels

In platforms like Discord or Slack, the agent can be configured with "Home Channels". These are specific channels where the agent is always active and listening, without needing to be explicitly mentioned.

```yaml
gateway:
  discord:
    enabled: true
    token: ${DISCORD_TOKEN}
    home_channels:
      - "123456789012345678" # #agent-ops
      - "987654321098765432" # #dev-chat
```

In home channels, the agent monitors all conversation and can proactively interject if it sees a problem it can solve, or if a user asks a question that falls within its domain of expertise.

### Delivery Preferences

Different platforms have different constraints on message size and formatting. The Gateway system allows you to configure delivery preferences for each platform.

```yaml
gateway:
  telegram:
    delivery:
      max_message_length: 4000
      split_strategy: word_boundary
      code_block_handling: truncate_and_upload
```

In this example, if the agent generates a response longer than Telegram's 4000-character limit, the Gateway will split the message at word boundaries. If the message contains a massive code block, instead of splitting it across dozens of messages, the Gateway will truncate the code block in the chat and automatically upload the full code as a file attachment.

## 25. The Migration Path: From OpenClaw to Hermes

Hermes Agent is the spiritual and technical successor to OpenClaw. Recognizing that many users have significant investments in OpenClaw deployments, Hermes Agent includes a comprehensive migration tool: `hermes claw migrate`.

### Migration Process

The migration tool performs a multi-step process to convert an OpenClaw environment into a Hermes Agent environment.

1.  **Configuration Translation**: It reads the OpenClaw `config.json` and translates it into the Hermes `config.yaml` format, mapping legacy settings to their modern equivalents.
2.  **Memory Conversion**: It converts OpenClaw's unstructured memory files into the structured `MEMORY.md` and `USER.md` formats used by Hermes.
3.  **Session Database Migration**: It extracts conversation history from OpenClaw's storage and imports it into Hermes Agent's SQLite FTS5 database, preserving the session lineage.
4.  **Skill Adaptation**: It analyzes OpenClaw custom tools and attempts to wrap them in the Hermes `SKILL.md` format, allowing them to function as procedural memories.

```bash
# Example migration command
hermes claw migrate --source ~/.openclaw --dest ~/.hermes --backup-first --auto-approve
```

This built-in migration path ensures that users can upgrade to the advanced capabilities of Hermes Agent without losing their historical data or custom configurations.

## 26. Conclusion and Future Directions

Hermes Agent v0.14.0 represents a significant leap forward in autonomous agent technology. Its architecture is designed not just for simple chat interactions, but for robust, long-running, and secure operations in complex production environments.

From the sophisticated error classification and multi-model orchestration to the deep procedural memory of the skills system and the rigorous security of the Tirith module, every component has been engineered to handle the realities of real-world deployment.

As the AI landscape continues to evolve, Hermes Agent is positioned to adapt. Future developments will focus on expanding the Kanban orchestration system for massive multi-agent swarms, deepening the integration with enterprise context engines, and further refining the self-improving capabilities of the background review system. By mastering the advanced configurations detailed in this document, operators can ensure that their Hermes Agent deployments remain at the cutting edge of autonomous capability.
