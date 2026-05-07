# Bash Security Audit Checklist

The Bash shell is a powerful and widely used command-line interface and scripting language on Unix-based systems. Due to its wide usage, ensuring its security is paramount. This document provides a comprehensive checklist and detailed guide for conducting a security audit of the Bash environment. It includes validation steps, permission models, known vulnerabilities, and hardening strategies. This guide is intended for system administrators and security professionals to secure Bash environments.

## Table of Contents

1. [Introduction](#introduction)
2. [Pre-Audit Preparation](#pre-audit-preparation)
3. [Environment Validation](#environment-validation)
4. [Permission Model](#permission-model)
5. [Known Vulnerabilities](#known-vulnerabilities)
6. [Hardening Strategies](#hardening-strategies)
7. [Post-Audit Actions](#post-audit-actions)
8. [Conclusion](#conclusion)

---

## Introduction

The Bourne Again Shell (Bash) is the default shell for many Linux distributions and Unix systems. Its flexibility and power make it indispensable, but these same features can also introduce security risks. This checklist provides a structured approach to auditing and securing Bash environments, focusing on best practices and common vulnerabilities.

## Pre-Audit Preparation

Before starting a security audit, ensure the following:

- **Backup Configuration Files**: Make sure to backup all relevant configuration files such as `.bashrc`, `.bash_profile`, and system-wide settings in `/etc/profile` and `/etc/bash.bashrc`.
- **Documentation**: Collect documentation on the specific environment, including version information, specific configurations, and any custom scripts.
- **Access Control**: Establish who will perform the audit and ensure they have appropriate access rights.
- **Environment Inventory**: Prepare an inventory of all systems running Bash. Note the versions and any deviations from standard configurations.

## Environment Validation

### Step 1: Bash Version Check

Ensure you are using the latest stable version of Bash, as older versions may have unpatched vulnerabilities.

```bash
bash --version
```

- **Action**: If an outdated version is detected, plan for an upgrade to the latest stable release.

### Step 2: Configuration File Review

Review the following key configuration files for unnecessary or risky settings:

- `~/.bashrc`
- `~/.bash_profile`
- `/etc/profile`
- `/etc/bash.bashrc`

**Validation Actions:**

- Look for any unexpected or suspicious aliases or functions.
- Ensure no sensitive information is hard-coded in these files.
- Confirm the presence of security-related settings like `HISTCONTROL` and `HISTFILESIZE`.

### Step 3: Environment Variables

Inspect environment variables for sensitive data exposure.

```bash
printenv
```

**Validation Actions:**

- Ensure no secrets or sensitive information (such as passwords, tokens) are stored in environment variables.
- Review `PATH`, `LD_LIBRARY_PATH`, and other important variables for insecure paths.

## Permission Model

### User and File Permissions

Proper permission management is crucial for Bash security:

- **Home Directory**: Ensure user home directories have correct permissions.

```bash
ls -ld /home/username
```

- **Action**: Permissions should typically be set to `700` to prevent other users from accessing sensitive files.

- **Bash History**: Limit access to the Bash history file.

```bash
ls -l ~/.bash_history
```

- **Action**: Set permissions to `600` to ensure only the user can read and write to it.

### SUID and SGID Executables

Identify and review SUID (Set User ID) and SGID (Set Group ID) executables:

```bash
find / -perm /6000 -type f 2>/dev/null
```

- **Action**: Minimize the number of SUID/SGID programs. Remove the SUID/SGID bits from executables that do not require them.

## Known Vulnerabilities

### Shellshock Vulnerability

One of the most critical vulnerabilities in Bash is "Shellshock". To check if your system is vulnerable, run the following:

```bash
env x='() { :;}; echo vulnerable' bash -c "echo this is a test"
```

- **Action**: If the output includes "vulnerable", your system is at risk. Immediately update Bash to a patched version.

### Command Injection Risks

Review scripts for command injection vulnerabilities:

- **Validation**: Look for unescaped or unsanitized inputs that are used in shell commands.
- **Action**: Use `shellcheck` to detect potential command injection flaws in scripts.

```bash
shellcheck script.sh
```

## Hardening Strategies

### Restricting Bash Features

- **Limited Shell Access**: Use `rbash` (Restricted Bash) for users who require limited functionality.

```bash
ln -s /bin/bash /bin/rbash
```

- **Action**: Configure `/etc/shells` and user profiles to use `rbash` where appropriate.

### Secure Bash History

- **HISTCONTROL**: Configure `HISTCONTROL` to ignore duplicate and commands starting with spaces.

```bash
export HISTCONTROL=ignoreboth
```

- **HISTFILESIZE**: Limit the size of the history file to prevent excessive storage of commands.

```bash
export HISTFILESIZE=500
```

- **Clear History on Logout**: Optionally, clear history upon logout for highly secure environments.

```bash
unset HISTFILE
```

### Security Patches and Updates

- **Regular Updates**: Ensure that your system regularly receives and applies security patches, especially for Bash.

```bash
sudo apt update && sudo apt upgrade
```

- **Action**: Consider using automated systems like `cron` or `unattended-upgrades` on Debian-based systems to handle updates.

### Script Security

- **Script Permissions**: Ensure scripts are not writable by unauthorized users.

```bash
chmod 700 script.sh
```

- **Code Review**: Regularly review scripts for potential security issues, including logic errors and unsafe operations.

### Logging and Monitoring

- **Audit Logs**: Enable and review audit logs for Bash commands and activities.

```bash
sudo auditctl -a always,exit -F arch=b64 -S execve
```

- **Action**: Use tools like `auditd` and `rsyslog` to monitor and alert on suspicious activity.

## Post-Audit Actions

- **Document Findings**: Compile a detailed report of findings, including vulnerabilities discovered, actions taken, and recommendations for improvement.
- **Remediation**: Implement changes based on audit results and validate effectiveness.
- **Continuous Monitoring**: Establish a schedule for regular audits and monitoring to ensure ongoing security.

## Conclusion

Securing the Bash environment is an ongoing process that requires vigilance and attention to detail. By following this comprehensive checklist, you can significantly reduce the risk of security breaches and ensure that your Bash environments remain robust and secure. Regular audits, along with continuous monitoring and updates, will help maintain a secure command-line interface for all users.

---

This document provides a detailed framework for auditing Bash security. However, always remain informed about emerging threats and best practices in the security community to adapt and enhance your security posture.