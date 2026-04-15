# NVIDIA NemoClaw Advanced Guide: Troubleshooting, Scaling, and Edge Cases

## Introduction

This guide is intended as a supplementary resource to the main NVIDIA NemoClaw Specialist Guide. It dives into advanced topics, including complex troubleshooting scenarios, scaling strategies for enterprise deployments, detailed security hardening, and handling edge cases within the NemoClaw and OpenShell ecosystem [1].

## Advanced Security Hardening

While NemoClaw provides a robust default security posture, enterprise environments often require additional hardening to meet strict compliance and risk management frameworks [1].

### Container-Level Hardening

The NemoClaw sandbox image is designed to minimize the attack surface [2]. Build toolchains such as `gcc`, `g++`, and `make`, along with network probes like `netcat`, are explicitly removed from the runtime image [2]. If compilation is required during the build process, operators should utilize a multi-stage build approach, performing compilation in a separate stage and copying only the necessary artifacts to the runtime stage [2].

### Process and Capability Management

To prevent fork-bomb attacks and limit resource exhaustion, the container's `ENTRYPOINT` sets a strict process limit using `ulimit -u 512` [2]. This limit is also enforced by the startup script (`nemoclaw-start.sh`) [2]. When launching the sandbox directly with `docker run`, this value can be adjusted via the `--ulimit nproc=512:512` flag [2].

A critical aspect of container security is the management of Linux capabilities [2]. When running the sandbox container, operators must explicitly drop all capabilities and re-add only those strictly required [2]. This is not something the `Dockerfile` can enforce; it must be configured at runtime [2].

| Orchestrator | Configuration Example |
| --- | --- |
| Docker CLI | `docker run --rm --cap-drop=ALL --ulimit nproc=512:512 nemoclaw-sandbox` |
| Docker Compose | `cap_drop: [ALL]`, `cap_add: [NET_BIND_SERVICE]`, `security_opt: [no-new-privileges:true]` |

### Filesystem Restrictions via Landlock

NemoClaw utilizes the Landlock Linux Security Module (LSM) to enforce filesystem access rules at the kernel level [1]. The sandbox Landlock policy restricts the agent's home directory (`/sandbox`) to read-only access [2]. Only explicitly declared directories, such as `/sandbox/.openclaw-data` and `/tmp`, are writable [2].

This read-only home directory prevents the agent from writing and executing scripts, modifying its runtime environment, creating persistent hidden files, or staging data for exfiltration [2].

> **Important:** Landlock LSM requires Linux kernel 5.13 or later with `CONFIG_SECURITY_LANDLOCK=y` [2]. On kernels that do not support Landlock, protection falls back to Discretionary Access Control (DAC) only, which may allow the agent to write to files it owns [2]. For production deployments, verifying Landlock availability via `ls /sys/kernel/security/landlock` is strongly recommended [2].

## Complex Troubleshooting Scenarios

Troubleshooting a NemoClaw deployment often involves analyzing the interactions between the agent, the OpenShell gateway, and the host environment [1].

### Network Egress Failures

If the agent is failing to reach an external service, the first step is to review the OpenShell Terminal User Interface (TUI) [1]. The TUI surfaces blocked requests, allowing the operator to approve or deny them [1].

If a request is blocked and not appearing in the TUI, it may be due to a binary-scoped endpoint rule [1]. OpenShell identifies the calling binary by reading `/proc/<pid>/exe` and computing a SHA256 hash [1]. If the binary has been modified or replaced, the hash mismatch will trigger an immediate denial [1]. Operators should verify the integrity of the binary and update the policy if necessary [1].

### Inference Routing Issues

Inference routing failures typically manifest as the agent being unable to communicate with `inference.local` [1]. This can occur if the provider credentials on the host are invalid or if the OpenShell gateway is misconfigured [1].

To diagnose this, operators should check the blueprint runner logs and the sandbox container logs using the `nemoclaw logs` command [3]. Ensure that the `nemoclaw onboard` process completed successfully and that the selected provider (e.g., NVIDIA Endpoints, OpenAI) is properly configured [3].

### State Migration Errors

NemoClaw supports the migration of agent state across machines [1]. This process involves creating a snapshot, stripping credentials, and verifying integrity [1]. If a migration fails, it is often due to corrupted state files or mismatched blueprint versions [3].

Operators should ensure that the source and destination machines are running compatible versions of OpenShell and OpenClaw, as defined in the `blueprint.yaml` [3]. The migration logs will indicate which specific file or integrity check failed [3].

## Scaling and Enterprise Deployment

Scaling NemoClaw deployments requires careful consideration of resource allocation, policy management, and monitoring [1].

### Managing Multiple Sandboxes

In an enterprise environment, it is common to run multiple OpenClaw agents, each in its own sandbox [1]. NemoClaw facilitates this by allowing the `nemoclaw onboard` command to be executed multiple times, creating distinct sandboxes based on the versioned blueprint [1].

To manage these sandboxes effectively, operators should implement centralized logging and monitoring [1]. While the `nemoclaw logs` command is useful for individual sandboxes, enterprise deployments should aggregate logs from the OpenShell gateway and the sandbox containers into a centralized SIEM (Security Information and Event Management) system [1].

### Dynamic Policy Management

As agents take on new tasks, their network access requirements will change [1]. NemoClaw's declarative policy management allows operators to update the baseline policy dynamically [1].

For development environments, operators might apply presets for package registries like PyPI and npm [1]. In production, these presets should be removed, and specific endpoint rules should be defined based on the principle of least privilege [1].

| Posture Profile | Recommended Configuration |
| --- | --- |
| Locked-Down (Default) | Keep all defaults. Use operator approval for any endpoint. Use local Ollama or NVIDIA Endpoints. |
| Development | Apply PyPI/npm presets. Keep binary restrictions. Use operator approval for unknown endpoints. |
| Integration Testing | Add custom endpoint entries with tight path/method restrictions. Use `protocol: rest` for HTTP APIs. |

### Edge Cases and Known Limitations

Operators must be aware of certain edge cases and limitations when deploying NemoClaw [1].

For example, the Memory Secret Scanner in the NemoClaw plugin is designed to block the agent from writing likely secrets (API keys, tokens) to persistent memory [1]. However, this is a heuristic-based scanner and may produce false positives or false negatives [1]. Operators should not rely solely on this scanner and must ensure that credentials are not inadvertently passed to the agent [1].

Another edge case involves the `allowInsecureAuth` setting in the OpenClaw gateway [1]. This setting controls whether the gateway permits non-HTTPS authentication [1]. In a production environment, this should always be disabled, and all communication with the Control UI should be secured via HTTPS [1].

## Conclusion

By understanding the advanced security features, troubleshooting techniques, and scaling strategies outlined in this guide, operators can deploy and manage NVIDIA NemoClaw in complex enterprise environments with confidence. The combination of container-level hardening, granular network policies, and robust lifecycle management makes NemoClaw a powerful platform for running autonomous AI agents safely.

## References

[1] NVIDIA. "Security Best Practices — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/security/best-practices.html
[2] NVIDIA. "Sandbox Image Hardening — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/deployment/sandbox-hardening.html
[3] NVIDIA. "Architecture — NVIDIA NemoClaw Developer Guide." https://docs.nvidia.com/nemoclaw/latest/reference/architecture.html