# Hermes Agent Security Audit

## Introduction
The Hermes Agent, developed by NousResearch, is a self-improving AI agent designed to operate across various environments, from local terminals to cloud sandboxes. Given its capability to execute code, manage files, and interact with external services, a robust security architecture is paramount. This document provides a comprehensive security audit of Hermes Agent v0.14.0, detailing its security mechanisms, isolation strategies, and vulnerability mitigation techniques. As AI agents become more autonomous and capable, the potential attack surface expands significantly. The Hermes Agent addresses these challenges through a multi-layered security approach, combining static analysis, runtime isolation, and dynamic authorization flows. This audit explores each of these layers in detail, providing operators with the knowledge necessary to deploy and manage the agent securely in production environments.

## DANGEROUS_PATTERNS System
The core of Hermes Agent's command execution security is the `DANGEROUS_PATTERNS` system. This system employs a set of regular expressions to identify potentially harmful commands before they are executed. This static analysis approach is crucial for preventing accidental or malicious damage to the host system, especially when the agent is operating with elevated privileges or within sensitive environments.

### Regex Patterns and Descriptions
The `DANGEROUS_PATTERNS` list includes, but is not limited to, the following regex patterns. Each pattern is carefully crafted to catch variations of dangerous commands while minimizing false positives.

- `rm\s+-rf\s+/`: Prevents recursive deletion of the root directory. This is perhaps the most well-known destructive command in Unix-like systems. The regex accounts for varying amounts of whitespace between the command and its arguments.
- `mkfs\.\w+\s+/dev/\w+`: Blocks attempts to format file systems. This prevents the agent from accidentally or maliciously erasing entire partitions or drives.
- `dd\s+if=.*of=/dev/\w+`: Stops low-level disk writing operations. The `dd` command is powerful and can easily overwrite critical system data if misused. This pattern specifically targets attempts to write directly to device files.
- `chmod\s+-R\s+777\s+/`: Prevents granting universal access to the entire file system. Such an action would completely compromise the security of the host, allowing any user to read, write, or execute any file.
- `chown\s+-R\s+root:root\s+/`: Blocks unauthorized ownership changes at the root level. While less immediately destructive than `chmod 777`, changing ownership can disrupt system services and lead to privilege escalation.
- `wget\s+.*\s+\|\s+sh`: Mitigates the risk of downloading and executing arbitrary scripts directly. This is a common vector for malware installation. By blocking this pattern, the agent is forced to download files first, allowing for potential inspection before execution.
- `curl\s+.*\s+\|\s+bash`: Similar to the `wget` pattern, prevents direct execution of downloaded scripts. This pattern covers the alternative tool commonly used for the same purpose.
- `>\s+/dev/sda`: Stops direct writing to primary disk devices. This prevents the agent from overwriting the boot sector or partition table, which would render the system unbootable.
- `mv\s+.*\s+/dev/null`: Prevents moving critical files to the null device. While `/dev/null` is often used to discard output, moving files to it effectively deletes them without the possibility of recovery.

These patterns act as a first line of defense, intercepting commands that could cause catastrophic system damage or compromise security. However, it is important to note that regex-based filtering is not foolproof. Sophisticated attackers may find ways to obfuscate commands to bypass these checks. Therefore, the `DANGEROUS_PATTERNS` system is just one component of a broader security strategy.

## Command Approval Flow
When a command matches a pattern in the `DANGEROUS_PATTERNS` list, the Hermes Agent initiates a command approval flow. This flow varies depending on the operational context, ensuring that security does not unduly hinder usability. The goal is to introduce a "human-in-the-loop" step for actions that carry significant risk.

### CLI Interactive
In a local CLI environment, the agent pauses execution and prompts the user for explicit approval. The user must review the command and confirm their intent before the agent proceeds. This interactive step ensures that potentially dangerous actions are not executed autonomously. The prompt typically displays the exact command that triggered the warning, along with a brief explanation of why it was flagged. This transparency allows the user to make an informed decision.

### Gateway Async
When operating through a messaging gateway (e.g., Telegram, Discord), the approval flow becomes asynchronous. The agent sends a message to the authorized user detailing the proposed command and requesting approval. The user can then respond with an approval or denial command (e.g., `/approve` or `/deny`). The agent waits for this response before continuing. This asynchronous approach is necessary because the user may not be actively monitoring the chat interface when the agent encounters a dangerous command. The agent maintains the context of the conversation and the pending command, ensuring that the workflow can resume smoothly once approval is granted.

### Smart Approval
Hermes Agent also incorporates a "smart approval" mechanism. This system learns from past user approvals and denials, gradually building a profile of acceptable actions within specific contexts. While it does not bypass the `DANGEROUS_PATTERNS` checks entirely, it can streamline the approval process for repetitive, low-risk tasks that the user has previously authorized. For example, if a user frequently approves a specific script execution that happens to trigger a warning, the smart approval system may eventually auto-approve that specific command, reducing friction while maintaining a baseline level of security. This feature requires careful tuning to balance convenience with safety.

## Container Isolation
To mitigate the risks associated with code execution, Hermes Agent heavily relies on containerization, primarily using Docker. The Docker backend is configured with stringent security hardening measures. Containerization provides a crucial layer of isolation, ensuring that even if the agent executes malicious code, the impact is confined to the container environment and does not compromise the host system.

### Docker Security Hardening
The Docker configuration for Hermes Agent employs several advanced security features to minimize the attack surface of the container.

- **cap-drop ALL**: This setting drops all Linux capabilities from the container, adhering to the principle of least privilege. Linux capabilities divide the privileges traditionally associated with superuser into distinct units. By dropping all capabilities, the containerized processes are prevented from performing privileged operations, such as modifying network configurations, loading kernel modules, or altering system time. This significantly reduces the potential impact of a container breakout.
- **no-new-privileges**: This flag ensures that processes within the container cannot gain additional privileges, even if they execute a setuid binary. This prevents privilege escalation attacks where an attacker exploits a vulnerability in a setuid program to gain root access within the container. This is a critical defense-in-depth measure.
- **pids-limit**: By limiting the number of processes that can run within the container, this setting mitigates fork bomb attacks and resource exhaustion. A fork bomb is a denial-of-service attack where a process continually replicates itself to deplete available system resources. The `pids-limit` ensures that even if a malicious script attempts a fork bomb, it will quickly hit the limit and be terminated, preventing it from affecting other containers or the host system.

These measures ensure that even if a malicious script is executed within the container, its impact is strictly confined. The combination of capability dropping, privilege restriction, and resource limits creates a robust sandbox environment for the agent's operations.

## DM Pairing Authorization Flow
For gateway integrations, Hermes Agent employs a Direct Message (DM) pairing authorization flow. This ensures that only authorized users can interact with and control the agent. In a multi-user environment like a Discord server or a Telegram group, it is essential to restrict access to the agent's capabilities to prevent unauthorized use or abuse.

1. **Initial Contact**: When a user first messages the agent, the agent generates a unique pairing code. This code is typically a short, alphanumeric string that is easy to communicate but difficult to guess.
2. **Verification**: The user must provide this pairing code to the agent administrator (or enter it via a secure channel) to verify their identity. This out-of-band verification step is crucial for establishing trust. It ensures that the person claiming the platform ID is indeed authorized to use the agent.
3. **Authorization**: Once verified, the agent adds the user's platform ID to an allowlist, granting them access to the agent's capabilities. The allowlist is persistently stored, so the user does not need to repeat the pairing process for subsequent interactions.

This flow prevents unauthorized access and ensures accountability for actions performed through the gateway. It provides a clear audit trail of who authorized which actions, which is essential for security monitoring and incident response.

## File Safety Checks
Hermes Agent implements rigorous file safety checks to prevent unauthorized access and modification of the host file system. These checks are applied whenever the agent attempts to read, write, or execute files, regardless of the tool being used.

### Path Traversal Prevention
The agent sanitizes all file paths provided in tool arguments to prevent path traversal attacks (e.g., `../../etc/passwd`). Path traversal vulnerabilities occur when an application uses user-supplied input to construct a file path without properly sanitizing it. This can allow an attacker to access files outside of the intended directory. Hermes Agent uses robust path normalization techniques to resolve all paths to their absolute canonical form and then verifies that the resulting path falls within the designated working directory or explicitly allowed paths. Any attempt to access files outside these boundaries is blocked and logged.

### Sensitive File Protection
The agent maintains a list of sensitive files and directories (e.g., `~/.ssh`, `/etc/shadow`, `/etc/passwd`, `/var/log`) that are strictly off-limits for read or write operations, regardless of the user's intent. This hardcoded list provides an additional layer of defense against accidental or malicious access to critical system resources. Even if a path traversal attack were to succeed, the sensitive file protection mechanism would still block access to these specific files.

## URL Safety and Website Policy
When interacting with the web (e.g., using `web_search` or `browser_navigate`), Hermes Agent enforces URL safety checks. It validates URLs against known malicious domains and adheres to a strict website policy that prohibits interaction with sites known to host malware or phishing content. This protects the agent from drive-by downloads and other web-based threats. The agent may integrate with external threat intelligence feeds to keep its list of malicious domains up to date. Additionally, the agent may employ heuristics to identify potentially dangerous URLs, such as those containing excessive obfuscation or pointing to known bad IP addresses.

## Credential Management
Proper credential management is crucial for maintaining the security of the agent and the services it interacts with. Hermes Agent requires access to various API keys, access tokens, and other secrets to function effectively. Mishandling these credentials can lead to severe security breaches.

### Secrets in .env
Sensitive information, such as API keys and access tokens, must be stored in the `~/.hermes/.env` file. This file should have strict permissions (e.g., `600`) to prevent unauthorized access by other users on the system. The `.env` file format is a standard way to manage environment variables in Python applications. By keeping secrets in a separate file, they are less likely to be accidentally committed to version control systems or exposed in configuration backups.

### Non-Secrets in config.yaml
Non-sensitive configuration settings, such as preferred models, terminal backends, and UI preferences, are stored in `~/.hermes/config.yaml`. This separation ensures that secrets are not accidentally exposed when sharing or versioning configuration files. The `config.yaml` file can be safely shared with other users or stored in a public repository without compromising the security of the agent.

## Environment Variable Passthrough Security
When executing commands or scripts, Hermes Agent carefully manages environment variable passthrough. It only passes explicitly required variables, preventing the accidental exposure of sensitive information (like API keys) to untrusted processes. By default, child processes inherit the environment of their parent. If the agent's environment contains sensitive API keys, any script it executes would also have access to those keys. To mitigate this risk, Hermes Agent explicitly constructs the environment for child processes, including only the variables that are strictly necessary for the task at hand. This principle of least privilege minimizes the potential impact of a compromised script.

## Skill Provenance and Trust Model
The skills system in Hermes Agent introduces a unique set of security challenges. Skills are essentially plugins that extend the agent's capabilities. Because skills can execute code and interact with the system, it is crucial to ensure that they are trustworthy. To address these challenges, the agent implements a skill provenance and trust model.

### Skills Guard
The "Skills Guard" mechanism verifies the integrity and origin of skills before they are loaded. It checks for digital signatures or cryptographic hashes to ensure that the skill has not been tampered with and originates from a trusted source (e.g., the official Skills Hub). This prevents the injection of malicious skills into the agent's procedural memory. When a user attempts to install a new skill, the Skills Guard verifies its signature against a list of trusted public keys. If the signature is invalid or missing, the installation is blocked. This ensures that only authorized and verified skills can be added to the agent's repertoire.

## OSV Vulnerability Checking
Hermes Agent integrates with the Open Source Vulnerability (OSV) database to continuously monitor its dependencies for known vulnerabilities. This proactive approach ensures that the agent is not susceptible to exploits targeting outdated or compromised libraries. The OSV database provides a comprehensive and up-to-date list of vulnerabilities across various open-source ecosystems. By regularly checking its dependencies against this database, Hermes Agent can alert administrators to potential security risks and prompt them to update vulnerable packages. This automated vulnerability scanning is a critical component of a robust security posture.

## Message Sanitization
To prevent injection attacks and ensure the stability of the agent's internal processing, all incoming messages are sanitized. This is particularly important when the agent is interacting with untrusted users through a messaging gateway.

### Surrogates and Non-ASCII
The agent handles surrogate pairs and non-ASCII characters carefully, ensuring they do not cause parsing errors or buffer overflows. Improper handling of complex character encodings can lead to vulnerabilities that attackers can exploit to crash the agent or execute arbitrary code. Hermes Agent uses robust string processing libraries to safely handle a wide range of character sets.

### Structure Sanitization
The agent sanitizes the structure of messages, removing potentially harmful HTML tags or script injections before processing the content. This prevents cross-site scripting (XSS) attacks and other forms of content injection. By stripping out executable content, the agent ensures that messages are treated purely as data, rather than executable code.

## Redaction System
Hermes Agent includes a redaction system that automatically identifies and masks sensitive information (e.g., credit card numbers, social security numbers, API keys) in its logs and outputs. This prevents the accidental leakage of personally identifiable information (PII) and other sensitive data. The redaction system uses regular expressions and pattern matching to identify sensitive data and replaces it with a placeholder (e.g., `[REDACTED]`). This ensures that even if logs are compromised, the sensitive information remains protected.

## Gateway Authorization
The gateway system employs multiple layers of authorization to control access. This is essential for managing who can interact with the agent and what actions they can perform.

### Allowlists
Administrators can configure allowlists to restrict access to specific users or groups on a given platform. This is the primary mechanism for controlling access to the agent. Only users whose platform IDs are on the allowlist are permitted to send commands to the agent.

### Global Allow-All
While not recommended for production environments, a "global allow-all" setting is available for testing purposes, granting access to anyone who interacts with the agent. This setting should be used with extreme caution, as it completely disables the authorization checks and exposes the agent to potential abuse.

## Supply Chain Security
Hermes Agent prioritizes supply chain security by using exact-pinned dependencies. This is a critical defense against attacks that target the software supply chain.

### Exact-Pinned Dependencies Rationale
By pinning dependencies to specific versions (e.g., `openai==2.24.0`), the agent ensures that it is not inadvertently updated to a compromised version of a library. This mitigates the risk of supply chain attacks, such as the hypothetical "Mini Shai-Hulud incident," where a malicious actor compromises a widely used dependency. Exact pinning provides deterministic builds and ensures that the agent is always running with known, tested versions of its dependencies. While this requires more manual effort to manage updates, the security benefits far outweigh the administrative overhead.

## Tirith Security Module
The Tirith security module is a core component of Hermes Agent's security architecture. It provides a centralized framework for enforcing security policies, managing authorizations, and auditing agent actions. Tirith acts as a gatekeeper, intercepting all critical operations and verifying that they comply with the established security policies. It provides a unified interface for managing security settings and reviewing audit logs, simplifying the task of securing the agent.

## Path Security
As mentioned earlier, path security is enforced to prevent traversal attacks. The agent uses robust path normalization and validation routines to ensure that all file operations remain within authorized boundaries. This is a fundamental security requirement for any application that interacts with the file system.

## Container Bypass Logic
In certain environments, such as Docker, Singularity, or Modal, the agent may bypass some of its internal security checks. This is because these environments already provide a strong layer of isolation. The container bypass logic intelligently determines when it is safe to rely on the underlying environment's security mechanisms, optimizing performance without compromising safety. For example, if the agent detects that it is running within a highly restricted Docker container, it may disable its internal file safety checks, relying instead on the container's file system isolation. This avoids redundant checks and improves the agent's overall efficiency.

## Conclusion
The Hermes Agent incorporates a comprehensive suite of security features designed to protect the host system, user data, and the integrity of the agent itself. From the `DANGEROUS_PATTERNS` system to strict container isolation and rigorous credential management, these mechanisms ensure that the agent can operate safely and autonomously in diverse environments. Continuous monitoring and proactive vulnerability management further enhance its security posture, making it a robust and reliable tool for a wide range of applications. By understanding and leveraging these security features, operators can confidently deploy Hermes Agent in production environments, knowing that it is equipped to handle the complex security challenges of autonomous AI operations.
