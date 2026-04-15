# NVIDIA NemoClaw Specialist Guide

## Introduction to NVIDIA NemoClaw

NVIDIA NemoClaw is an open-source reference stack designed to simplify the deployment of OpenClaw always-on assistants within a secure, hardened environment [1]. By wrapping OpenClaw in NVIDIA OpenShell containers, NemoClaw introduces policy-based privacy and security guardrails that give operators granular control over their agents' behavior and data handling. This architecture enables self-evolving claws to run more safely across various environments, including cloud instances, on-premise servers, RTX PCs, and DGX Spark [1].

The core value proposition of NemoClaw lies in its ability to pair open-source and hosted models, such as NVIDIA Nemotron, with a hardened sandbox, routed inference, and declarative egress policies [1]. While the underlying sandbox runtime is provided by NVIDIA OpenShell, NemoClaw adds the essential blueprint, the `nemoclaw` CLI, onboarding workflows, and related tooling to establish a reference standard for running OpenClaw securely [1].

## Key Features and Capabilities

NemoClaw addresses the inherent risks of autonomous AI agents, which can typically make arbitrary network requests, access the host filesystem, and call any inference endpoint [1]. Without appropriate guardrails, these capabilities create significant security, cost, and compliance risks as agents operate unattended [1]. NemoClaw mitigates these risks through several key features.

| Feature | Description |
| --- | --- |
| Guided Onboarding | Validates credentials, selects providers, and creates a working sandbox in a single command. |
| Hardened Blueprint | A security-first Dockerfile with capability drops, least-privilege network rules, and declarative policy. |
| State Management | Safe migration of agent state across machines with credential stripping and integrity verification. |
| Channel Messaging | OpenShell-managed processes connect Telegram, Discord, Slack, and similar platforms to the sandboxed agent. |
| Routed Inference | Provider-routed model calls through the OpenShell gateway, transparent to the agent. |
| Layered Protection | Network, filesystem, process, and inference controls that can be hot-reloaded or locked at creation. |

## Architectural Overview

The architecture of NemoClaw is divided into two primary components: a TypeScript plugin that integrates with the OpenClaw CLI, and a Python blueprint that orchestrates OpenShell resources [3].

### System Components

NVIDIA OpenShell serves as the general-purpose agent runtime, providing sandbox containers, a credential-storing gateway, inference proxying, and policy enforcement [3]. However, OpenShell has no opinions about the specific workloads running inside it. NemoClaw acts as the opinionated reference stack built on top of OpenShell, handling what goes into the sandbox and making the setup accessible and repeatable [3].

The system architecture involves the host machine orchestrating the OpenShell environment, which in turn isolates and runs the NemoClaw sandbox [4]. The `nemoclaw` CLI is the primary entry point for setting up and managing these sandboxed OpenClaw agents, delegating the heavy lifting to the versioned blueprint [2].

### Plugin and Blueprint Architecture

NemoClaw's functionality is split between the plugin and the blueprint to maintain stability while allowing for independent evolution [2].

The plugin is a TypeScript package that registers an inference provider and the `/nemoclaw` slash command inside the sandbox [2]. It handles user interaction and delegates orchestration work to the blueprint [2]. The plugin runs in-process with the OpenClaw gateway inside the sandbox [3].

The blueprint is a versioned Python artifact containing all the logic for creating sandboxes, applying policies, and configuring inference [2]. The plugin resolves, verifies, and executes the blueprint as a subprocess [2]. This separation ensures that the orchestration logic can evolve on its own release cadence without destabilizing the core plugin [2].

### Sandbox Creation Lifecycle

When an operator runs `nemoclaw onboard`, NemoClaw creates an OpenShell sandbox that runs OpenClaw in an isolated container [2]. The blueprint orchestrates this process through the OpenShell CLI in a defined lifecycle [3]:

1. **Resolve**: The plugin locates the blueprint artifact and checks the version against `min_openshell_version` and `min_openclaw_version` constraints.
2. **Verify**: The plugin checks the artifact digest against the expected value to ensure supply chain safety.
3. **Plan**: The runner determines what OpenShell resources to create or update, such as the gateway, providers, sandbox, inference route, and policy.
4. **Apply**: The runner executes the plan by calling `openshell` CLI commands.
5. **Status**: The runner reports the current state of the deployment.

## Inference Routing and Configuration

A critical security feature of NemoClaw is its handling of inference requests. Inference requests from the agent never leave the sandbox directly [2]. Instead, OpenShell intercepts every inference call and routes it to the configured provider [2].

During the onboarding process, NemoClaw validates the selected provider and model, configures the OpenShell route, and bakes the matching model reference into the sandbox image [2]. The sandbox then communicates exclusively with `inference.local`, while the host machine retains the actual provider credentials and manages the upstream endpoint connection [2]. This architecture ensures that the agent never possesses or has access to the provider API keys [3].

NemoClaw supports a variety of inference providers, including NVIDIA Endpoints, OpenAI, Anthropic, Google Gemini, and local Ollama deployments [1]. For sensitive workloads, utilizing local Ollama is recommended to keep data on-premise, while NVIDIA Endpoints provide a strong balance of capability and trust for general use cases [5].

## Ecosystem Integration: NemoClaw vs. OpenShell

Understanding the relationship between OpenClaw, OpenShell, and NemoClaw is essential for deploying the right stack for a given use case [4].

OpenClaw is the assistant itself, comprising the runtime, tools, memory, and behavior inside the container [4]. OpenShell is the execution environment, providing the sandbox lifecycle, network, filesystem, and process policy, inference routing, and the operator-facing CLI for those primitives [4]. NemoClaw is the NVIDIA reference stack that implements this definition on the host, driving OpenShell APIs and CLI to create and configure the sandbox that runs OpenClaw in a documented, repeatable way [4].

### Value Addition Over Community Sandbox

While OpenShell provides a community sandbox for OpenClaw, NemoClaw adds significant security hardening, automation, and lifecycle tooling [4].

| Capability | OpenShell Community Sandbox | NemoClaw Reference Stack |
| --- | --- | --- |
| Credential Handling | Manual creation of providers. | Automatic creation during onboarding; filters sensitive host environment variables from the sandbox creation command. |
| Image Hardening | Includes standard system tools. | Strips build toolchains and network probes to reduce attack surface. |
| Filesystem Policy | Bundled policy for OpenClaw. | Restrictive read-only and read-write layout; prevents agent from writing and executing scripts or staging data. |
| Inference Setup | Manual configuration or in-sandbox wizard. | Validates credentials from the host and configures routing automatically; credentials stay on the host. |
| Blueprint Versioning | Uses whatever image version is currently published. | Downloads versioned artifact, checks compatibility, and verifies digest before applying. |

## Best Practices for Deployment

Deploying NemoClaw requires adherence to specific best practices to maintain the integrity and security of the environment.

### Policy Management

The sandbox starts with a default policy that controls network egress, filesystem access, process privileges, and inference routing [2]. When the agent attempts to reach an unlisted host, OpenShell blocks the request and surfaces it in the Terminal User Interface (TUI) for operator approval [2]. Approved endpoints persist for the current session but are not saved to the baseline policy file, ensuring that temporary access does not become a permanent security loophole [2].

### Credential Protection

NemoClaw keeps its operator-facing state on the host rather than inside the sandbox [3]. This includes the `NEMOCLAW_DISABLE_DEVICE_AUTH` environment variable, which configures optional services and local access [3]. For normal setup and reconfiguration, operators should prefer using `nemoclaw onboard` over editing configuration files by hand [3].

> **Note:** Do not treat `NEMOCLAW_DISABLE_DEVICE_AUTH` as a runtime setting for an already-created sandbox. It is intended for initial configuration and should be managed carefully [3].

## Conclusion

NVIDIA NemoClaw represents a significant advancement in the secure deployment of autonomous AI agents. By leveraging the isolation capabilities of OpenShell and adding a robust layer of policy enforcement, lifecycle management, and credential protection, NemoClaw enables organizations to harness the power of OpenClaw assistants while mitigating the associated risks.

For advanced topics, including in-depth troubleshooting, scaling strategies, detailed security hardening, and edge case management, please refer to the child file: `nemoclaw-advanced.md`.

## References

[1] NVIDIA. "Overview — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/overview.html
[2] NVIDIA. "How NemoClaw Works — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/how-it-works.html
[3] NVIDIA. "Architecture — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/reference/architecture.html
[4] NVIDIA. "Ecosystem — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/ecosystem.html
[5] NVIDIA. "Security Best Practices — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/security/best-practices.html


## Comprehensive Architecture Breakdown

The NemoClaw architecture is designed to enforce strict separation of concerns, ensuring that the AI agent operates within a confined environment while the host machine manages critical security and orchestration tasks [3].

### Host Machine Operations

The host machine is responsible for orchestrating the entire lifecycle of the sandbox [3]. This involves the `nemoclaw` CLI, which serves as the primary interface for operators [2]. The CLI handles tasks such as guided setup, provider selection, credential validation, and deployment [3]. By keeping these operations on the host, NemoClaw ensures that sensitive information, such as provider API keys, is never exposed to the sandbox environment [3].

The host also manages the blueprint, which is a versioned Python artifact [3]. The blueprint contains the hardened Dockerfile, network policies, presets, and security configurations necessary to create the sandbox [3]. This declarative approach to infrastructure ensures that deployments are reproducible and consistent across different environments [2].

### Sandbox Container Environment

The sandbox container is the isolated environment where the OpenClaw agent runs [3]. This container is built from a hardened image, specifically `ghcr.io/nvidia/openshell-community/sandboxes/openclaw` [3]. Inside the sandbox, the NemoClaw plugin is pre-installed, extending the agent's capabilities with managed configuration and the `/nemoclaw` slash command [3].

The sandbox environment is heavily restricted to prevent unauthorized actions by the agent [3]. Network egress is controlled by a baseline policy, and filesystem access is limited to specific directories [3]. The agent's home directory (`/sandbox`) is mounted read-only, with only specific data paths (`/sandbox/.openclaw-data`, `/sandbox/.nemoclaw`, `/tmp`) configured as writable [4]. This prevents the agent from modifying its own runtime environment or staging data for exfiltration [4].

### OpenShell Gateway and Policy Engine

The OpenShell gateway acts as the intermediary between the sandbox container and the outside world [3]. It intercepts all inference requests and network traffic originating from the agent [3]. The gateway is responsible for credential storage, inference proxying, and policy enforcement [3].

When the agent attempts to communicate with an external service, the request is evaluated against the defined network policies [3]. If the request is not explicitly allowed, it is blocked, and the operator is prompted for approval [2]. This deny-by-default approach ensures that the agent cannot establish unauthorized connections [5].

## Advanced Security Posture

NemoClaw implements a multi-layered security model that addresses the specific risks associated with autonomous AI agents [5]. This model encompasses network controls, filesystem restrictions, process limitations, and inference routing [5].

### Network Controls

The network policy in NemoClaw is declarative and restrictive by default [5]. All outbound connections from the sandbox are blocked unless explicitly permitted in the policy file [5]. This prevents the agent from exfiltrating data or communicating with malicious endpoints [5].

Endpoint rules can be scoped to specific binaries, ensuring that only authorized executables can access certain network resources [5]. For example, a rule might allow the `npm` binary to access the npm registry while blocking all other processes from doing so [5]. This granular control is achieved by verifying the SHA256 hash of the calling binary [5].

Additionally, endpoint rules can restrict allowed HTTP methods and URL paths, further limiting the agent's ability to interact with external services [5]. The OpenShell CONNECT proxy can also perform Layer 7 (L7) inspection of HTTP traffic, providing deeper visibility into the agent's communications [5].

### Filesystem Restrictions

NemoClaw employs stringent filesystem controls to protect system binaries, configuration files, and gateway credentials [5]. The container mounts system directories read-only, preventing the agent from modifying critical components [5].

The `/sandbox/.openclaw` directory, which contains the OpenClaw gateway configuration, is particularly well-protected [5]. It is owned by the root user with read-only permissions (`chmod 444`), and the immutable flag (`chattr +i`) is applied to the directory and its symlinks [5]. At startup, the entrypoint verifies the integrity of the configuration by checking a pinned SHA256 hash [5].

Furthermore, NemoClaw utilizes the Landlock Linux Security Module (LSM) to enforce filesystem access rules at the kernel level [5]. This provides an additional layer of defense against unauthorized file access [5].

### Process Limitations

To mitigate the risk of malicious or runaway processes, NemoClaw limits the capabilities, user privileges, and resource quotas available inside the sandbox [5].

At startup, the entrypoint drops dangerous Linux capabilities from the bounding set, restricting what child processes can acquire [5]. The OpenClaw gateway runs as a separate user (`gateway`) from the agent (`sandbox`), ensuring privilege separation [5].

The `no-new-privileges` flag is set to prevent processes from gaining additional privileges through setuid binaries or capability inheritance [5]. Additionally, a process limit (`ulimit -u 512`) caps the number of processes the sandbox user can spawn, mitigating the risk of fork-bomb attacks [5].

## Ecosystem and Integration Workflows

The NemoClaw ecosystem is designed to integrate seamlessly with existing tools and platforms, providing a comprehensive solution for deploying and managing AI agents [4].

### Comparison with OpenShell Path

While OpenShell provides the foundational sandbox capabilities, NemoClaw adds significant value through its opinionated reference stack [4]. The OpenShell community sandbox requires manual configuration of providers, channel settings, and network policies [4]. In contrast, NemoClaw automates these processes during onboarding, ensuring a secure and consistent deployment [4].

NemoClaw also introduces advanced features such as blueprint versioning and state migration [4]. The versioned blueprint ensures that the sandbox is created from a known, verified state, while state migration allows the agent's memory and configuration to be safely transferred between machines [4].

### Channel Messaging Integration

NemoClaw simplifies the integration of OpenClaw with messaging platforms such as Telegram, Discord, and Slack [4]. During onboarding, NemoClaw collects the necessary bot tokens and registers them as OpenShell providers [4]. The OpenClaw channel configuration is baked into the sandbox with placeholder tokens, which the OpenShell proxy resolves to the actual credentials at egress [4]. This eliminates the need for a separate bridge process on the host machine and ensures that the agent never has direct access to the bot tokens [4].

## Operational Workflows and Management

Managing a NemoClaw deployment involves several operational workflows, including onboarding, monitoring, and policy updates [2].

### The Onboarding Process

The onboarding process is initiated using the `nemoclaw onboard` command [2]. This guided workflow validates the operator's credentials, selects the desired inference provider, and orchestrates the creation of the sandbox [2].

During onboarding, the operator is prompted to configure the inference routing [2]. This involves selecting a provider (e.g., NVIDIA Endpoints, OpenAI, local Ollama) and specifying the model to be used [4]. NemoClaw automatically configures the OpenShell routing to ensure that all inference requests from the agent are directed to the selected provider [4].

### Monitoring and Auditing

Continuous monitoring of the sandbox is essential for maintaining a secure environment [5]. The OpenShell TUI provides real-time visibility into the agent's network activity and resource usage [5]. Operators can use the TUI to review and approve or deny network requests that are not covered by the baseline policy [5].

Additionally, NemoClaw provides tools for streaming logs from the blueprint runner and the sandbox container [3]. These logs are invaluable for troubleshooting issues and auditing the agent's behavior [3].

### Updating Network Policies

As the agent's requirements evolve, it may be necessary to update the baseline network policy [5]. NemoClaw provides preset policy files for common integrations, such as npm and PyPI [5]. These presets can be applied to grant the agent the necessary access while maintaining a secure posture [5].

When adding custom endpoint rules, operators should adhere to the principle of least privilege, restricting access to specific binaries, HTTP methods, and URL paths whenever possible [5].

## Conclusion

NVIDIA NemoClaw is a powerful reference stack that enables organizations to deploy OpenClaw AI assistants with confidence. By combining the isolation capabilities of OpenShell with a robust layer of policy enforcement, lifecycle management, and credential protection, NemoClaw provides a secure and scalable foundation for autonomous AI operations.

The comprehensive security controls, automated workflows, and deep integration with the OpenShell ecosystem make NemoClaw an essential tool for any organization looking to harness the potential of AI agents while mitigating the associated risks.

For advanced topics, including in-depth troubleshooting, scaling strategies, detailed security hardening, and edge case management, please refer to the child file: `nemoclaw-advanced.md`.

## References

[1] NVIDIA. "Overview — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/overview.html
[2] NVIDIA. "How NemoClaw Works — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/how-it-works.html
[3] NVIDIA. "Architecture — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/reference/architecture.html
[4] NVIDIA. "Ecosystem — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/about/ecosystem.html
[5] NVIDIA. "Security Best Practices — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/security/best-practices.html