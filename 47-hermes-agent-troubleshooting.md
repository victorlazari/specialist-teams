# Hermes Agent Troubleshooting Guide

Welcome to the definitive troubleshooting reference for Hermes Agent v0.14.0. This guide is designed for operators running Hermes Agent in production environments, covering everything from provider connection errors to complex multi-agent orchestration issues.

## 1. Provider Connection Errors

When Hermes Agent fails to connect to one of its 30+ supported AI providers, the error classification system will attempt to diagnose the issue. Here are the most common provider connection errors and their resolutions.

### API Key Issues

**Symptoms:**
- `AuthenticationError` or `401 Unauthorized` in the logs.
- The agent immediately fails upon startup or when switching models.

**Resolution:**
1. Verify the API key in your `~/.hermes/.env` file.
2. Ensure the key corresponds to the correct provider (e.g., `OPENAI_API_KEY` for OpenAI, `ANTHROPIC_API_KEY` for Anthropic).
3. If using a credential pool, check that the pool has not been exhausted or rate-limited.
4. Run `hermes doctor` to validate all configured API keys.

### Rate Limits

**Symptoms:**
- `RateLimitError` or `429 Too Many Requests`.
- The agent experiences jittered backoff and eventually fails.

**Resolution:**
1. Check your provider's dashboard for rate limit quotas.
2. Implement the Nous rate guard if using Nous Research models.
3. Configure failover providers in `config.yaml` to automatically switch to an auxiliary model when rate limits are hit.
4. Increase the iteration budget or adjust the backoff parameters in the configuration.

### Context Too Small

**Symptoms:**
- `ContextLengthExceeded` or `400 Bad Request` related to token limits.
- The agent fails when processing large files or long conversation histories.

**Resolution:**
1. Ensure the model's `context_length` in `config.yaml` matches the provider's specifications.
2. Enable context compression to automatically summarize older messages.
3. Manually trigger context compression using the `/compact` slash command.
4. Switch to a model with a larger context window (e.g., Claude 3 Opus or GPT-4 Turbo).

## 2. Ollama Context Limit Errors

Ollama requires specific configuration to handle the minimum 64K token context required by Hermes Agent.

**Symptoms:**
- The agent fails to load the model or crashes during inference.
- Error messages indicating insufficient context size.

**Resolution:**
1. Ensure your Ollama installation is up to date.
2. In `config.yaml`, set `ollama_num_ctx` to at least `65536`.
3. Verify that your hardware has sufficient VRAM to support the requested context size.
4. If VRAM is limited, consider using a smaller model or reducing the context size, though this may impact agent performance.

## 3. Docker Backend Issues

The Docker backend provides a persistent container for the agent to execute commands safely.

### Container Not Starting

**Symptoms:**
- `DockerException` or timeouts when initializing the terminal backend.
- The agent falls back to the local backend.

**Resolution:**
1. Ensure the Docker daemon is running and accessible.
2. Check the `docker_image` setting in `config.yaml` and verify the image exists locally or can be pulled.
3. Review the `docker_extra_args` for any conflicting or invalid flags.

### Permission Denied

**Symptoms:**
- `PermissionError` when the agent attempts to read or write files within the container.

**Resolution:**
1. Verify the `docker_mount` and `docker_volumes` settings in `config.yaml`.
2. Ensure the host directories have the correct permissions for the Docker user.
3. If using SELinux or AppArmor, check for any security profiles blocking access.

### GPU Not Detected

**Symptoms:**
- The agent cannot utilize GPU acceleration for tasks like image generation or local inference.

**Resolution:**
1. Ensure the NVIDIA Container Toolkit is installed and configured correctly.
2. Add `--gpus all` to the `docker_extra_args` in `config.yaml`.
3. Verify that the Docker image supports GPU acceleration (e.g., CUDA base image).

## 4. SSH Backend Issues

The SSH backend allows Hermes Agent to execute commands on remote servers using a persistent shell.

### Connection Refused

**Symptoms:**
- `SSHException` or `Connection refused` when initializing the terminal backend.

**Resolution:**
1. Verify the SSH server is running on the target machine.
2. Check the SSH port and hostname in the `ssh` settings of `config.yaml`.
3. Ensure any firewalls or security groups allow inbound SSH traffic from the agent's host.

### Key Auth Failures

**Symptoms:**
- `Authentication failed` or `Permission denied (publickey)`.

**Resolution:**
1. Verify the path to the private key in the `ssh` settings.
2. Ensure the public key is added to the `~/.ssh/authorized_keys` file on the target machine.
3. Check the permissions of the private key file (should be `600`).

### Persistent Shell Not Working

**Symptoms:**
- Commands execute slowly or fail to maintain state between executions.

**Resolution:**
1. Ensure `ControlMaster` is enabled in your SSH configuration (`~/.ssh/config`).
2. Verify the `ControlPath` directory exists and has the correct permissions.
3. Check for any SSH timeouts or keepalive settings that might be dropping the connection.

## 5. Gateway Issues

The Gateway system connects Hermes Agent to 20+ messaging platforms.

### Platform Not Connecting

**Symptoms:**
- The agent fails to receive or send messages on a specific platform.
- Errors in the gateway logs indicating connection timeouts or authentication failures.

**Resolution:**
1. Verify the platform credentials (e.g., bot tokens, API keys) in `~/.hermes/.env`.
2. Check the platform configuration in `config.yaml` under the `gateway` section.
3. Ensure the platform's API is accessible and not blocked by firewalls.
4. Run `hermes gateway status` to check the health of all configured platforms.

### DM Pairing Failures

**Symptoms:**
- Users cannot pair with the agent via direct messages.
- The authorization flow fails or times out.

**Resolution:**
1. Verify the `home_channels` configuration in `config.yaml`.
2. Ensure the user is initiating the pairing request from an authorized channel.
3. Check the gateway logs for any errors during the pairing process.
4. If using global allow-all, ensure the security implications are understood and accepted.

### Session Key Conflicts

**Symptoms:**
- Messages from different users or channels are mixed up.
- The agent responds to the wrong context.

**Resolution:**
1. Verify the session key resolution logic in the gateway adapter.
2. Ensure each platform generates unique session keys for different users and channels.
3. Check the SQLite database for any corrupted or overlapping session entries.

## 6. Memory Issues

The memory system manages the agent's personal notes and user profiles.

### Memory Full

**Symptoms:**
- The agent cannot add new memories.
- Warnings about exceeding the 2,200 character limit for `MEMORY.md` or 1,375 character limit for `USER.md`.

**Resolution:**
1. Manually review and edit the memory files in `~/.hermes/memories/`.
2. Use the `/memory` slash command to inspect and manage memories.
3. Instruct the agent to consolidate or summarize its memories.

### Entries Not Persisting

**Symptoms:**
- Memories added during a session are lost upon restart.

**Resolution:**
1. Verify the permissions of the `~/.hermes/memories/` directory.
2. Check the agent logs for any errors during the memory save operation.
3. Ensure the session storage is functioning correctly and not corrupted.

### Frozen Snapshot Stale

**Symptoms:**
- The agent uses outdated memory information despite recent updates.

**Resolution:**
1. Understand that memory snapshots are frozen at session start for prefix caching.
2. Restart the session or use the `/reset` command to force a new snapshot.
3. If real-time memory updates are critical, consider disabling prefix caching or using a dynamic memory provider like Honcho.

## 7. Skills Issues

Skills are procedural memories that allow the agent to learn and improve.

### Skill Not Loading

**Symptoms:**
- The agent cannot access a specific skill.
- Errors in the logs indicating skill parsing or loading failures.

**Resolution:**
1. Verify the `SKILL.md` format is correct and includes all required metadata.
2. Check the `skills` configuration in `config.yaml` for correct paths.
3. Run `hermes skills list` to see all loaded skills and their status.
4. Ensure any required dependencies for the skill are installed.

### Conditional Activation Not Working

**Symptoms:**
- A skill activates when it shouldn't, or fails to activate when needed.

**Resolution:**
1. Review the `fallback_for_toolsets` and `requires_toolsets` settings in the skill metadata.
2. Ensure the agent's current toolset matches the skill's requirements.
3. Check the prompt assembly logs to see which skills were injected into the context.

### Env Vars Missing

**Symptoms:**
- A skill fails to execute due to missing environment variables.

**Resolution:**
1. Verify the required environment variables are defined in the skill metadata.
2. Ensure the variables are set in `~/.hermes/.env` or passed through the environment.
3. Check the environment variable passthrough security settings to ensure the variables are allowed.

## 8. MCP Issues

The Model Context Protocol (MCP) integrates external tools and servers.

### Server Not Starting

**Symptoms:**
- The agent cannot connect to an MCP server.
- Errors indicating the server process failed to start or crashed.

**Resolution:**
1. Verify the `command` and `args` in the MCP server configuration.
2. Ensure the server executable is in the system PATH or provide an absolute path.
3. Check the server logs for any startup errors or missing dependencies.

### Tool Discovery Failing

**Symptoms:**
- The agent connects to the MCP server but cannot discover any tools.

**Resolution:**
1. Verify the MCP server implements the tool discovery protocol correctly.
2. Check the `tool_filter` configuration to ensure the tools are not being excluded.
3. Review the agent logs for any errors during the tool discovery phase.

### OAuth Timeouts

**Symptoms:**
- The agent times out while waiting for OAuth authentication.

**Resolution:**
1. Ensure the OAuth flow is configured correctly and the redirect URIs match.
2. Increase the timeout settings for the MCP server in `config.yaml`.
3. If possible, use a headless OAuth flow or pre-authenticate the server.

## 9. Cron Issues

The cron system schedules and executes background jobs.

### Jobs Not Firing

**Symptoms:**
- Scheduled jobs do not execute at the expected times.

**Resolution:**
1. Verify the cron schedule syntax in `jobs.json`.
2. Ensure the `hermes cron` daemon is running.
3. Check the cron logs for any errors or missed executions.

### Delivery Failures

**Symptoms:**
- The job executes, but the results are not delivered to the platform.

**Resolution:**
1. Verify the platform configuration and credentials.
2. Ensure the target channel or user is accessible and authorized.
3. Check the gateway logs for any delivery errors.

### Timezone Problems

**Symptoms:**
- Jobs execute at the wrong time relative to the user's timezone.

**Resolution:**
1. Ensure the system timezone is set correctly.
2. Specify the timezone explicitly in the cron schedule if supported.
3. Review the croniter documentation for timezone handling.

## 10. Session Issues

Session storage manages the conversation history and lineage.

### Resume Not Working

**Symptoms:**
- The agent fails to resume a previous session.
- `hermes --continue` starts a new session instead.

**Resolution:**
1. Verify the session ID is correct and exists in the SQLite database.
2. Check the database for any corruption or missing entries.
3. Ensure the session lineage tracking is functioning correctly.

### FTS5 Search Empty

**Symptoms:**
- The `/search` command returns no results despite matching keywords.

**Resolution:**
1. Verify the SQLite database was compiled with FTS5 support.
2. Rebuild the FTS5 index using the appropriate database command.
3. Check the search query syntax for any errors.

### Corruption Recovery

**Symptoms:**
- The SQLite database is corrupted, causing the agent to crash or fail to load sessions.

**Resolution:**
1. Restore the database from a backup if available.
2. Use the SQLite `.recover` command to attempt data extraction.
3. If recovery fails, delete the database file and allow the agent to recreate it (this will lose all session history).

## 11. Context Compression Failures

Context compression reduces the token count of long conversations.

**Symptoms:**
- The agent fails to compress the context, leading to token limit errors.
- The compressed context loses critical information.

**Resolution:**
1. Verify the compression algorithms (trajectory, conversation) are enabled in `config.yaml`.
2. Adjust the compression thresholds and parameters to balance token savings and information retention.
3. Use manual compression feedback to guide the agent on what information to keep.

## 12. Browser Tool Issues

Browser tools allow the agent to navigate and interact with web pages.

### CDP Connection

**Symptoms:**
- The agent cannot connect to the browser via the Chrome DevTools Protocol (CDP).

**Resolution:**
1. Ensure Chrome or Chromium is installed and accessible.
2. Verify the browser configuration in `config.yaml`.
3. Check for any conflicting processes using the CDP port.

### Vision Routing

**Symptoms:**
- The agent fails to analyze browser snapshots or routes them to the wrong provider.

**Resolution:**
1. Verify the vision provider configuration in `config.yaml`.
2. Ensure the selected provider supports image analysis.
3. Check the image routing system logs for any errors during provider selection.

## 13. Terminal Tool Issues

Terminal tools allow the agent to execute commands and interact with the system.

### Command Timeout

**Symptoms:**
- Commands execute but time out before completion.

**Resolution:**
1. Increase the `timeout` setting in the terminal configuration.
2. Instruct the agent to run long-running commands in the background or use the `process` tool.
3. Check the command output for any prompts or interactive elements blocking execution.

### Approval Stuck

**Symptoms:**
- The agent waits indefinitely for command approval.

**Resolution:**
1. Verify the command approval flow (CLI interactive, gateway async, smart approval).
2. Ensure the user is receiving the approval request and responding correctly.
3. Check the gateway logs for any delivery or response parsing errors.

### ANSI Stripping

**Symptoms:**
- The command output contains raw ANSI escape codes, making it difficult to read.

**Resolution:**
1. Ensure the terminal backend is configured to strip ANSI codes if necessary.
2. Instruct the agent to use commands that disable color output (e.g., `--no-color`).
3. Check the TUI gateway configuration for any ANSI handling settings.

## 14. Supply Chain Security

Hermes Agent uses exact-pinned dependencies to ensure supply chain security.

### Exact-Pinned Deps

**Symptoms:**
- Installation fails due to dependency conflicts or missing packages.

**Resolution:**
1. Ensure you are using the exact versions specified in `requirements.txt` or `pyproject.toml`.
2. Avoid manually upgrading dependencies unless necessary and tested.
3. Use a virtual environment to isolate the agent's dependencies from the system.

### The Mini Shai-Hulud Incident

**Symptoms:**
- Concerns about malicious packages or vulnerabilities in the dependency tree.

**Resolution:**
1. Review the OSV vulnerability checking logs for any reported issues.
2. Keep the agent updated to the latest version to receive security patches.
3. Monitor the Nous Research security advisories for any critical updates.

## 15. Performance Issues

Performance issues can impact the agent's responsiveness and efficiency.

### Slow Startup

**Symptoms:**
- The agent takes a long time to initialize and become ready.

**Resolution:**
1. Verify the terminal backend initialization time (e.g., Docker container startup).
2. Check the prompt assembly pipeline for any slow operations (e.g., loading large context files).
3. Ensure the SQLite database is optimized and not overly large.

### High Token Usage

**Symptoms:**
- The agent consumes a large number of tokens, leading to high costs or rate limits.

**Resolution:**
1. Enable context compression to reduce the size of the conversation history.
2. Review the `SOUL.md` and context files to ensure they are concise and relevant.
3. Use a smaller or more efficient model for routine tasks.

### Prompt Too Large

**Symptoms:**
- The assembled prompt exceeds the model's context limit.

**Resolution:**
1. Check the prompt assembly logs to identify the largest components.
2. Reduce the size of the memory snapshots or context files.
3. Manually trigger context compression or start a new session.

## 16. Cross-Platform Issues

Hermes Agent supports multiple operating systems and environments.

### Windows Native

**Symptoms:**
- The agent fails to execute certain terminal commands or interact with the file system correctly.

**Resolution:**
1. Ensure the terminal backend is compatible with Windows (e.g., local backend with PowerShell).
2. Be aware of path separator differences (`\` vs `/`) and instruct the agent accordingly.
3. Consider using WSL2 for a more consistent Linux-like environment.

### WSL2

**Symptoms:**
- Issues with Docker integration or file permissions.

**Resolution:**
1. Ensure Docker Desktop is configured to integrate with your WSL2 distribution.
2. Verify file permissions and ownership within the WSL2 environment.
3. Check for any networking issues between WSL2 and the Windows host.

### macOS

**Symptoms:**
- The `computer_use` tool fails to control the desktop or interact with applications.

**Resolution:**
1. Ensure the agent has the necessary Accessibility and Screen Recording permissions in System Settings.
2. Verify the `apple` skill category is loaded and configured correctly.
3. Check the agent logs for any AppleScript or UI scripting errors.

---

This troubleshooting guide covers the most common issues encountered when operating Hermes Agent. For further assistance, consult the official documentation, join the Nous Research community, or review the source code and issue tracker on GitHub.


## 17. Advanced Provider Configuration Troubleshooting

When dealing with advanced provider configurations, such as failover chains and credential pools, troubleshooting can become more complex.

### Failover Chain Failures

**Symptoms:**
- The agent fails to switch to an auxiliary model when the primary model encounters an error.
- The failover process loops infinitely or exhausts the iteration budget.

**Resolution:**
1. Review the `FailoverReason` enum in the error classifier logs to understand why the primary model failed.
2. Ensure the auxiliary models in the failover chain are correctly configured and have valid credentials.
3. Check the `config.yaml` for any circular dependencies in the failover configuration.
4. Increase the iteration budget if the failover process requires more attempts to succeed.

### Credential Pool Exhaustion

**Symptoms:**
- The agent fails to authenticate with a provider despite having multiple keys in the credential pool.
- The credential pool rotation logic fails to select a valid key.

**Resolution:**
1. Verify the status of all keys in the credential pool using `hermes doctor`.
2. Ensure the credential sources (e.g., environment variables, secret managers) are accessible and returning valid keys.
3. Check the rate limit tracking logs to see if all keys in the pool have been rate-limited.
4. Implement a backoff strategy or alert mechanism when the credential pool is nearing exhaustion.

## 18. Multi-Model Orchestration (Mixture of Agents)

The Mixture of Agents tool allows Hermes Agent to orchestrate multiple models for complex tasks.

### Orchestration Timeouts

**Symptoms:**
- The Mixture of Agents tool times out while waiting for responses from sub-agents.
- The overall task execution is significantly delayed.

**Resolution:**
1. Increase the timeout settings for the Mixture of Agents tool in `config.yaml`.
2. Ensure the sub-agents are using efficient models and are not overloaded with complex prompts.
3. Check the provider connection logs for any latency issues with the selected models.
4. Consider running the sub-agents in parallel if supported by the orchestration logic.

### Sub-Agent Failures

**Symptoms:**
- One or more sub-agents fail to complete their assigned tasks, causing the overall orchestration to fail.

**Resolution:**
1. Review the error logs for the specific sub-agents that failed.
2. Ensure the sub-agents have the necessary tools and context to complete their tasks.
3. Implement retry logic or fallback mechanisms for critical sub-agent tasks.
4. Adjust the task delegation strategy to ensure tasks are assigned to the most capable models.

## 19. Advanced Skills Development Troubleshooting

Developing and debugging advanced skills requires a deep understanding of the skills system.

### Skill Provenance and Trust Issues

**Symptoms:**
- The agent refuses to load a skill due to provenance or trust verification failures.
- The Skills Guard prevents the execution of a potentially malicious skill.

**Resolution:**
1. Verify the source of the skill and ensure it comes from a trusted repository or the Skills Hub.
2. Check the skill metadata for any missing or invalid cryptographic signatures.
3. Review the Skills Guard logs to understand why the skill was flagged as malicious.
4. If developing a custom skill, ensure it adheres to the security guidelines and does not contain any dangerous patterns.

### Background Review System Failures

**Symptoms:**
- The background review system fails to generate memory or skill nudges.
- The post-turn hooks cause the agent loop to hang or crash.

**Resolution:**
1. Verify the background review system is enabled in `config.yaml`.
2. Check the logs for any errors during the execution of the post-turn hooks.
3. Ensure the model used for background review has sufficient context and capabilities to analyze the conversation history.
4. Adjust the frequency or triggers for the background review system to reduce overhead.

## 20. Context Engine Plugins and Memory Providers

Hermes Agent supports advanced context engine plugins like Honcho for dynamic memory management.

### Honcho Integration Issues

**Symptoms:**
- The agent fails to connect to the Honcho server or retrieve dynamic context blocks.
- The Honcho static block is missing from the assembled prompt.

**Resolution:**
1. Verify the Honcho server URL and credentials in `config.yaml`.
2. Ensure the Honcho server is running and accessible from the agent's environment.
3. Check the prompt assembly logs to see if the Honcho plugin was successfully invoked.
4. Review the Honcho server logs for any errors during context retrieval.

### Memory Provider Conflicts

**Symptoms:**
- The agent experiences conflicts or inconsistencies when using multiple memory providers (e.g., SQLite and Honcho).

**Resolution:**
1. Ensure the memory providers are configured correctly and do not overlap in their responsibilities.
2. Review the memory manager internals to understand how different providers are prioritized and merged.
3. If conflicts persist, consider disabling one of the memory providers or implementing a custom conflict resolution strategy.

## 21. Iteration Budget System Troubleshooting

The iteration budget system prevents the agent from getting stuck in infinite loops.

### Budget Exhaustion

**Symptoms:**
- The agent terminates a task prematurely due to exhausting its iteration budget.
- The logs indicate `IterationBudgetExceeded`.

**Resolution:**
1. Increase the iteration budget in `config.yaml` for complex tasks that require more steps.
2. Review the agent's trajectory to identify any inefficient or repetitive actions.
3. Instruct the agent to break down the task into smaller, more manageable sub-tasks.
4. Ensure the tools used by the agent are functioning correctly and returning actionable results.

### Budget Tracking Errors

**Symptoms:**
- The iteration budget is not tracked correctly, allowing the agent to exceed the configured limit.

**Resolution:**
1. Verify the iteration budget tracking logic in the agent loop internals.
2. Ensure the budget is decremented correctly after each action or tool call.
3. Check for any edge cases or exceptions that might bypass the budget tracking mechanism.

## 22. Advanced Docker Backend Troubleshooting

The advanced Docker backend supports features like GPU passthrough and persistent volumes.

### GPU Passthrough Failures

**Symptoms:**
- The agent cannot access the GPU within the Docker container, despite configuring `--gpus all`.

**Resolution:**
1. Verify the host machine has the correct NVIDIA drivers and Container Toolkit installed.
2. Ensure the Docker daemon is configured to use the `nvidia` runtime by default.
3. Check the container logs for any CUDA or driver initialization errors.
4. Test the GPU passthrough manually using a simple CUDA container (e.g., `docker run --gpus all nvidia/cuda:11.0-base nvidia-smi`).

### Persistent Volume Issues

**Symptoms:**
- Data written to persistent volumes within the container is lost upon restart.
- The agent encounters permission errors when accessing persistent volumes.

**Resolution:**
1. Verify the `docker_volumes` configuration in `config.yaml` and ensure the host paths are correct.
2. Check the permissions of the host directories and ensure they are accessible by the Docker user.
3. Ensure the container is not being removed or recreated with different volume mounts.
4. Use Docker named volumes instead of bind mounts for better portability and permission management.

## 23. Gateway Hooks and Extension Points

The Gateway system provides hooks and extension points for custom integrations.

### Hook Execution Failures

**Symptoms:**
- Custom gateway hooks fail to execute or cause the gateway to crash.

**Resolution:**
1. Review the custom hook code for any syntax errors or unhandled exceptions.
2. Ensure the hooks are registered correctly with the gateway runner.
3. Check the gateway logs for any errors during hook execution.
4. Implement proper error handling and logging within the custom hooks to facilitate debugging.

### Cross-Session Message Mirroring Issues

**Symptoms:**
- Messages are not mirrored correctly across different sessions or platforms.
- The mirroring logic causes message duplication or infinite loops.

**Resolution:**
1. Verify the cross-session message mirroring configuration and logic.
2. Ensure the session keys are resolved correctly and do not overlap.
3. Implement deduplication mechanisms to prevent infinite loops when mirroring messages between platforms.
4. Check the gateway logs for any errors during the mirroring process.

## 24. Plugin System Troubleshooting

The plugin system allows Hermes Agent to integrate with external services like Langfuse and Spotify.

### Langfuse Observability Issues

**Symptoms:**
- The agent fails to send telemetry data to Langfuse.
- The Langfuse dashboard does not show any traces or spans for the agent's execution.

**Resolution:**
1. Verify the Langfuse API keys and endpoint URL in `~/.hermes/.env`.
2. Ensure the Langfuse plugin is enabled in `config.yaml`.
3. Check the observability logs for any errors during telemetry transmission.
4. Verify the agent's network configuration allows outbound connections to the Langfuse server.

### Spotify Plugin Failures

**Symptoms:**
- The agent cannot control Spotify playback or retrieve track information.

**Resolution:**
1. Verify the Spotify API credentials and OAuth tokens in `~/.hermes/.env`.
2. Ensure the Spotify plugin is enabled and configured correctly.
3. Check the plugin logs for any authentication or API errors.
4. Verify the user's Spotify account is active and accessible.

## 25. Advanced Prompt Caching

Prompt caching improves performance and reduces costs by caching frequently used prompt segments.

### Anthropic Cache Control Issues

**Symptoms:**
- The agent fails to utilize Anthropic's prompt caching features.
- The API responses indicate that the cache was not hit.

**Resolution:**
1. Verify the Anthropic model supports prompt caching (e.g., Claude 3.5 Sonnet).
2. Ensure the prompt assembly pipeline correctly inserts the `cache_control` blocks.
3. Check the prompt caching logs to see if the cache was successfully populated and retrieved.
4. Review the Anthropic API documentation for any specific requirements or limitations regarding prompt caching.

## 26. Codex Responses Adapter

The Codex Responses adapter allows Hermes Agent to interface with systems expecting Codex-style responses.

### Adapter Parsing Errors

**Symptoms:**
- The adapter fails to parse the agent's responses into the expected Codex format.
- The downstream system rejects the adapter's output.

**Resolution:**
1. Verify the adapter configuration and mapping logic.
2. Ensure the agent's responses adhere to the expected structure and format.
3. Check the adapter logs for any parsing or validation errors.
4. Implement custom parsing logic if the default adapter does not support the specific Codex format required.

## 27. LSP Integration (Language Server Protocol)

LSP integration allows Hermes Agent to provide intelligent code completion and analysis.

### LSP Server Connection Failures

**Symptoms:**
- The agent cannot connect to the LSP server for a specific language.
- The LSP features (e.g., go to definition, find references) are unavailable.

**Resolution:**
1. Verify the LSP server is installed and accessible in the agent's environment.
2. Ensure the LSP integration configuration in `config.yaml` specifies the correct command and arguments for the server.
3. Check the LSP logs for any connection or initialization errors.
4. Verify the agent has the necessary permissions to execute the LSP server process.

## 28. Image Routing and Generation Backends

Hermes Agent supports multiple image generation backends, including FAL, OpenAI, and xAI.

### Provider Selection Failures

**Symptoms:**
- The image routing system fails to select the appropriate provider for a given prompt.
- The agent defaults to a suboptimal provider or fails entirely.

**Resolution:**
1. Verify the image routing configuration in `config.yaml`.
2. Ensure the selected providers have valid credentials and are accessible.
3. Check the image routing logs to understand why a specific provider was selected or rejected.
4. Adjust the routing logic or provider priorities to better suit the specific use case.

### Backend Registry Errors

**Symptoms:**
- The agent cannot find or load a specific image generation backend from the registry.

**Resolution:**
1. Verify the backend is correctly registered in the image routing system.
2. Ensure any required dependencies for the backend are installed.
3. Check the backend registry logs for any initialization or loading errors.
4. If developing a custom backend, ensure it implements the required interface and is registered correctly.

## 29. Kanban Orchestration Troubleshooting

Kanban orchestration allows Hermes Agent to manage multi-agent task boards and complex workflows.

### Task Board Synchronization Issues

**Symptoms:**
- The task board state is not synchronized correctly across multiple agents.
- Agents overwrite each other's updates or work on the same task simultaneously.

**Resolution:**
1. Verify the Kanban orchestration configuration and synchronization logic.
2. Ensure the task board storage mechanism (e.g., SQLite, external database) supports concurrent access and locking.
3. Check the orchestration logs for any synchronization or locking errors.
4. Implement robust conflict resolution mechanisms to handle concurrent updates.

### Crash Recovery Failures

**Symptoms:**
- The orchestration system fails to recover from a crash or unexpected termination.
- Tasks are left in an inconsistent or incomplete state.

**Resolution:**
1. Verify the crash recovery mechanisms are enabled and configured correctly.
2. Ensure the task board state is persisted regularly and can be restored from a backup.
3. Check the recovery logs for any errors during the restoration process.
4. Implement idempotent task execution to ensure tasks can be safely retried after a crash.

## 30. Conclusion

Troubleshooting Hermes Agent requires a systematic approach and a deep understanding of its architecture and components. By following the guidelines and resolutions outlined in this guide, operators can effectively diagnose and resolve a wide range of issues, ensuring the reliable and efficient operation of Hermes Agent in production environments.


## 31. Deep Dive: Debugging the Agent Loop

When standard troubleshooting steps fail, operators may need to dive deep into the agent loop internals (`run_conversation` flow, 3,900 lines) to identify the root cause of an issue.

### Tracing the Execution Flow

**Symptoms:**
- The agent exhibits unexpected behavior or gets stuck in a specific state.
- The logs do not provide sufficient information to diagnose the issue.

**Resolution:**
1. Enable debug logging in `config.yaml` to capture detailed execution traces.
2. Review the `run_conversation` flow to understand the sequence of operations (e.g., prompt assembly, API call, tool dispatch).
3. Insert custom logging statements or breakpoints in the agent loop code to inspect the state at specific points.
4. Analyze the trajectory and conversation history to identify any patterns or anomalies that might be causing the issue.

### Analyzing the Prompt Assembly Pipeline

**Symptoms:**
- The assembled prompt is incorrect or missing critical information.
- The agent fails to utilize specific skills or context files.

**Resolution:**
1. Review the 10 layers of the prompt assembly pipeline (e.g., SOUL.md, tool guidance, frozen memory snapshot).
2. Check the prompt assembly logs to see which components were included or excluded.
3. Verify the formatting and structure of the individual components (e.g., SKILL.md, AGENTS.md).
4. Ensure the context compression algorithms are not overly aggressive and removing important information.

## 32. Deep Dive: Debugging the Tool Registry and Dispatch System

The tool registry and dispatch system are responsible for managing and executing the agent's tools.

### Tool Registration Failures

**Symptoms:**
- A specific tool is not available to the agent.
- The tool registry logs indicate registration errors.

**Resolution:**
1. Verify the tool code is located in the correct directory (`tools/`) and follows the required structure.
2. Ensure the tool is correctly registered in `tools/registry.py`.
3. Check for any syntax errors or missing dependencies in the tool code.
4. Verify the tool metadata (e.g., name, description, parameters) is accurate and complete.

### Tool Dispatch Errors

**Symptoms:**
- The agent attempts to call a tool, but the dispatch system fails to execute it.
- The tool execution results in an unhandled exception.

**Resolution:**
1. Review the tool dispatch logs to understand why the execution failed.
2. Verify the arguments passed to the tool match the expected parameters.
3. Check the tool code for any logic errors or edge cases that might cause exceptions.
4. Ensure the tool has the necessary permissions and access to external resources (e.g., file system, network).

## 33. Deep Dive: Debugging the Session Storage Internals

The session storage system manages the conversation history and lineage using SQLite and FTS5.

### Database Schema Issues

**Symptoms:**
- The agent fails to initialize the session storage or encounters database errors.
- The SQLite schema is incompatible with the current version of Hermes Agent.

**Resolution:**
1. Verify the SQLite schema matches the expected structure for the current version.
2. Check the database migration logs for any errors during schema updates.
3. If necessary, manually inspect the database schema using the SQLite command-line tool.
4. Consider recreating the database if the schema is severely corrupted or incompatible.

### FTS5 Indexing Failures

**Symptoms:**
- The full-text search functionality (`/search`) is slow or returns inaccurate results.
- The FTS5 index is out of sync with the session data.

**Resolution:**
1. Verify the FTS5 index is being updated correctly when new messages are added.
2. Check the database logs for any indexing errors or performance bottlenecks.
3. Rebuild the FTS5 index manually using the appropriate SQLite commands.
4. Optimize the database configuration (e.g., cache size, synchronous mode) to improve indexing performance.

## 34. Deep Dive: Debugging the Context Compression Algorithms

Context compression is critical for managing long conversations and avoiding token limit errors.

### Trajectory Compression Issues

**Symptoms:**
- The trajectory compression algorithm fails to summarize the agent's actions effectively.
- Important context is lost during compression, leading to degraded performance.

**Resolution:**
1. Review the trajectory compression logic and parameters in `config.yaml`.
2. Ensure the model used for compression has sufficient capabilities to summarize complex actions.
3. Check the compression logs to see the original and compressed trajectories.
4. Adjust the compression thresholds to balance token savings and information retention.

### Conversation Compression Issues

**Symptoms:**
- The conversation compression algorithm fails to summarize the user's messages effectively.
- The compressed conversation history lacks coherence or context.

**Resolution:**
1. Review the conversation compression logic and parameters in `config.yaml`.
2. Ensure the model used for compression can accurately capture the user's intent and key information.
3. Check the compression logs to see the original and compressed conversation histories.
4. Use manual compression feedback to guide the agent on what information to keep.

## 35. Deep Dive: Debugging the Gateway Message Routing

The gateway message routing system handles the flow of messages between the agent and messaging platforms.

### Two-Level Message Guard Failures

**Symptoms:**
- Malicious or inappropriate messages bypass the message guard and reach the agent.
- Legitimate messages are incorrectly blocked by the message guard.

**Resolution:**
1. Review the two-level message guard configuration and rules.
2. Ensure the regex patterns and filtering logic are accurate and up to date.
3. Check the message guard logs to understand why specific messages were blocked or allowed.
4. Adjust the sensitivity of the message guard to balance security and usability.

### Interrupt System Issues

**Symptoms:**
- The agent fails to process interrupt signals or cancel ongoing tasks.
- The interrupt system causes the agent to crash or enter an inconsistent state.

**Resolution:**
1. Verify the interrupt system configuration and logic.
2. Ensure the messaging platform supports interrupt signals and transmits them correctly.
3. Check the interrupt system logs for any errors during signal processing.
4. Implement robust cancellation mechanisms within the agent's tools and tasks to handle interrupts gracefully.

## 36. Deep Dive: Debugging the Memory Manager Internals

The memory manager handles the agent's personal notes and user profiles.

### Bounded Curation Failures

**Symptoms:**
- The memory manager fails to enforce the character limits for `MEMORY.md` and `USER.md`.
- The memory files grow indefinitely, leading to token limit errors.

**Resolution:**
1. Review the bounded curation logic and parameters in `config.yaml`.
2. Ensure the memory manager correctly identifies and removes outdated or redundant information.
3. Check the memory manager logs for any errors during the curation process.
4. Manually review and edit the memory files to ensure they remain within the limits.

### Disk Persistence Issues

**Symptoms:**
- Memories are not saved to disk correctly, leading to data loss upon restart.
- The memory manager encounters permission or I/O errors when writing to disk.

**Resolution:**
1. Verify the permissions of the `~/.hermes/memories/` directory and ensure it is writable by the agent.
2. Check the memory manager logs for any disk persistence errors.
3. Ensure the disk has sufficient free space and is not experiencing hardware issues.
4. Implement backup mechanisms to protect against data loss.

## 37. Deep Dive: Debugging the Skill Manager Internals

The skill manager handles the discovery, loading, and tracking of skills.

### Discovery and Loading Failures

**Symptoms:**
- The skill manager fails to discover or load skills from the configured directories.
- The logs indicate parsing or validation errors for specific skills.

**Resolution:**
1. Verify the skill directories (`skills/` and `optional-skills/`) exist and contain valid `SKILL.md` files.
2. Ensure the skill manager configuration in `config.yaml` specifies the correct paths.
3. Check the skill manager logs for any discovery or loading errors.
4. Validate the `SKILL.md` files against the required schema and format.

### Usage Analytics Issues

**Symptoms:**
- The skill manager fails to track skill usage or generate accurate analytics.
- The usage analytics data is corrupted or incomplete.

**Resolution:**
1. Verify the usage analytics configuration and logic.
2. Ensure the skill manager correctly records each skill invocation and its outcome.
3. Check the usage analytics logs for any tracking or reporting errors.
4. Review the analytics data to identify any patterns or anomalies that might indicate tracking issues.

## 38. Deep Dive: Debugging the Error Classifier

The error classifier categorizes API errors and determines the appropriate failover strategy.

### API Error Parsing Failures

**Symptoms:**
- The error classifier fails to parse API errors correctly, leading to incorrect failover decisions.
- The logs indicate unhandled exceptions or unknown error types.

**Resolution:**
1. Review the error classifier logic and the `FailoverReason` enum.
2. Ensure the error classifier supports the specific API error formats used by the configured providers.
3. Check the error classifier logs for any parsing or classification errors.
4. Update the error classifier to handle new or modified API error formats.

## 39. Deep Dive: Debugging the Model Metadata System

The model metadata system provides information about the capabilities and limits of different models.

### Context Probing Failures

**Symptoms:**
- The model metadata system fails to determine the correct context limit for a specific model.
- The agent encounters token limit errors due to inaccurate context probing.

**Resolution:**
1. Verify the context probing logic and configuration.
2. Ensure the model metadata system correctly queries the provider's API or documentation for context limits.
3. Check the context probing logs for any errors or inconsistencies.
4. Manually configure the context limit in `config.yaml` if the probing mechanism fails.

### Token Estimation Issues

**Symptoms:**
- The token estimation logic is inaccurate, leading to unexpected token limit errors or inefficient prompt assembly.

**Resolution:**
1. Review the token estimation logic and the specific tokenizers used for different models.
2. Ensure the tokenizers are up to date and accurately reflect the model's tokenization rules.
3. Check the token estimation logs for any discrepancies between estimated and actual token counts.
4. Adjust the token estimation parameters or switch to a more accurate tokenizer if necessary.

## 40. Final Thoughts on Troubleshooting

Troubleshooting a complex system like Hermes Agent requires patience, persistence, and a deep understanding of its architecture. By leveraging the detailed logs, diagnostic tools, and deep dive sections provided in this guide, operators can effectively identify and resolve even the most challenging issues. Remember to consult the official documentation and community resources for additional support and guidance.
