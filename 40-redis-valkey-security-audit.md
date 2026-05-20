# Topic 5: Security Audit Procedures for Redis/Valkey

## Introduction

Redis and Valkey, as in-memory data stores and key management solutions respectively, are critical infrastructure components in modern production environments. Their security posture must be rigorously maintained to prevent data breaches, unauthorized access, and service disruptions. This document provides a **super comprehensive** guide on security audit procedures specifically focusing on:

- Access Control Lists (ACLs)
- Transport Layer Security (TLS) and Mutual TLS (mTLS) configuration
- Disabling dangerous commands (e.g., `rename-command`)
- Network isolation strategies

The content is tailored for **production operations and technical support teams**, emphasizing practical steps, worst-case scenarios, and mitigation strategies.

---

## Table of Contents

1. [Understanding Redis/Valkey Security Landscape](#understanding-redisvalkey-security-landscape)  
2. [Access Control Lists (ACLs)](#access-control-lists-acls)  
   - Overview  
   - ACL Configuration Best Practices  
   - Auditing ACLs in Production  
   - Troubleshooting ACL Issues  
3. [TLS and mTLS Configuration](#tls-and-mtls-configuration)  
   - Importance of Encryption in Transit  
   - Configuring TLS in Redis/Valkey  
   - Mutual TLS for Enhanced Security  
   - Auditing TLS/mTLS Deployment  
   - Common Pitfalls and Remediation  
4. [Disabling Dangerous Commands](#disabling-dangerous-commands)  
   - Risks of Dangerous Commands  
   - Using `rename-command` for Command Disabling  
   - Auditing Renamed or Disabled Commands  
   - Incident Handling for Command Exploits  
5. [Network Isolation](#network-isolation)  
   - Principles of Network Segmentation  
   - Best Practices for Redis/Valkey Network Setup  
   - Firewall Rules and Security Groups  
   - Audit Steps for Network Isolation  
6. [Worst-Case Scenarios and Incident Response](#worst-case-scenarios-and-incident-response)  
   - Unauthorized Access Cases  
   - Command Injection or Misuse  
   - TLS Certificate Compromise  
   - Network Breach  
7. [Summary and Continuous Security Monitoring](#summary-and-continuous-security-monitoring)  

---

## Understanding Redis/Valkey Security Landscape

Redis, by default, is an in-memory key-value store that is designed for speed and simplicity, but this also means it lacks built-in security features unless explicitly configured. Valkey, used for cryptographic key management, shares similar security demands.

### Key Security Concerns:

- **Unauthorized Access:** Without proper ACLs, any client can connect and issue commands.
- **Data Exposure:** Redis does not encrypt data at rest or in transit by default.
- **Command Abuse:** Certain commands (`FLUSHALL`, `CONFIG`, `DEBUG`) can be exploited for destructive or information-gathering purposes.
- **Network Exposure:** Redis is often deployed within internal networks but may be accidentally exposed to public networks.
- **Certificate and Key Management:** For TLS/mTLS, proper management of certificates is critical.

---

## Access Control Lists (ACLs)

### Overview

Since Redis 6.0, ACLs provide fine-grained control over user permissions, allowing admins to specify **who can connect, what commands they can run, and on which keys**.

Valkey, depending on version and implementation, follows similar principles or integrates with external authorization.

### ACL Configuration Best Practices

- **Default User Disabled:** The default `default` user should be disabled or have no permissions.
- **Granular Permissions:** Create specific users for different roles (read-only, write-only, admin).
- **Key Pattern Restrictions:** Limit users to only access keys relevant to their role using `~` patterns.
- **Command Restrictions:** Allow only necessary commands per user.
- **Strong Passwords:** Use robust passwords, ideally managed by secrets stores.
- **Rotate Credentials Regularly:** Implement scheduled rotation policies.
- **Use ACL Categories:** Use command categories where possible, e.g., `@read`, `@write`.

#### Example ACL Configuration

```
# Disable default user
ACL SETUSER default off

# Create readonly user
ACL SETUSER readonly on >StrongReadPwd ~cache:* +@read

# Create admin user with full access
ACL SETUSER admin on >StrongAdminPwd allcommands allkeys
```

### Auditing ACLs in Production

1. **List all users and their ACLs:**

   ```
   ACL LIST
   ```

2. **Check user status and permissions:**

   ```
   ACL GETUSER <username>
   ```

3. **Verify no user has `allcommands` unless explicitly required.**

4. **Audit key patterns:** Confirm key patterns (`~`) do not grant overbroad access.

5. **Check for inactive or legacy users:** Remove or disable unused accounts.

6. **Review password complexity and expiry policies** (where enforced externally).

#### Automating ACL Audits

- Use scripts to periodically fetch ACL configurations.
- Compare with baseline security policy.
- Alert on any unauthorized changes.

### Troubleshooting ACL Issues

- **Problem:** User cannot access certain keys despite permissions.  
  **Solution:** Verify the key pattern restrictions (`~`). Remember Redis ACLs require explicit matching.

- **Problem:** Connection refused or permission denied.  
  **Solution:** Check if user is enabled (`on`), password correctness, and IP whitelist.

- **Problem:** Over-permissive users found during audit.  
  **Solution:** Immediately restrict and rotate passwords.

---

## TLS and mTLS Configuration

### Importance of Encryption in Transit

Redis by default communicates in plaintext. In production, this exposes credentials and data to interception. TLS encrypts the connection, preventing MITM and eavesdropping.

mTLS enhances security by requiring client certificates, enabling mutual authentication.

### Configuring TLS in Redis/Valkey

Redis 6+ supports native TLS. For earlier versions, a TLS proxy (e.g., stunnel) is required.

#### Redis TLS Configuration Parameters (redis.conf):

```
tls-port 6379
port 0  # disable non-TLS port

tls-cert-file /etc/redis/certs/redis.crt
tls-key-file /etc/redis/certs/redis.key
tls-ca-cert-file /etc/redis/certs/ca.crt

tls-auth-clients no  # for one-way TLS
```

For mTLS:

```
tls-auth-clients yes
```

### Mutual TLS for Enhanced Security

- Requires clients to present valid certificates.
- Validates client identity.
- Prevents unauthorized clients from connecting even if credentials are compromised.

#### Steps to Enable mTLS:

1. Configure Redis with `tls-auth-clients yes`.
2. Ensure clients have certificates signed by the trusted CA.
3. Distribute client certificates securely.
4. Test mutual authentication.

### Auditing TLS/mTLS Deployment

- **Verify TLS is enabled and non-TLS ports are disabled:**

  ```
  redis-cli -p 6379 --tls info server
  ```

- **Check certificate validity and expiration:**

  Use `openssl`:

  ```
  openssl x509 -in /etc/redis/certs/redis.crt -noout -dates
  ```

- **Confirm clients use TLS:**

  Check client logs and connection parameters.

- **Test mTLS enforcement:**

  Attempt connection without client cert; it should fail.

### Common Pitfalls and Remediation

| Issue                                         | Explanation                                        | Remediation                                          |
|-----------------------------------------------|--------------------------------------------------|----------------------------------------------------|
| TLS port enabled alongside non-TLS port       | Allows insecure connections                        | Disable non-TLS port (`port 0`)                     |
| Expired or invalid certificates                | Connections may fail or be vulnerable              | Renew certificates before expiration                |
| Clients not configured for TLS                  | Data sent in plaintext                              | Update client libraries/configurations              |
| Weak cipher suites                              | Vulnerable to cryptographic attacks                | Use strong ciphers, update Redis and TLS libraries  |
| mTLS misconfiguration (e.g., client certs missing) | Unauthorized clients able to connect               | Enforce `tls-auth-clients yes` and validate certs   |

---

## Disabling Dangerous Commands

### Risks of Dangerous Commands

Commands like `FLUSHALL`, `CONFIG`, `DEBUG`, and `EVAL` can be abused to:

- Erase data
- Change server configuration
- Execute arbitrary code
- Leak sensitive information

### Using `rename-command` for Command Disabling

Redis allows renaming or disabling commands by setting their name to empty string.

#### Example: Disable `FLUSHALL` and `CONFIG`

```
rename-command FLUSHALL ""
rename-command CONFIG ""
```

This prevents their execution even if ACLs are misconfigured.

### Auditing Renamed or Disabled Commands

1. Check `redis.conf` or equivalent configuration management system for `rename-command` entries.

2. Use the `COMMAND` command to verify:

   ```
   COMMAND INFO FLUSHALL
   ```

   If the command is disabled, it will not appear or will return an error.

3. Cross-check with ACLs to ensure no user has access to dangerous commands.

4. Confirm that any renaming does not inadvertently create security holes (e.g., renaming to a common name that others might guess).

### Incident Handling for Command Exploits

- Immediately disable the exploited command via `rename-command`.
- Rotate credentials for all users.
- Perform forensic analysis of command logs.
- Restore from backups if data was deleted or altered.
- Harden ACLs to limit command execution.

---

## Network Isolation

### Principles of Network Segmentation

- **Least Privilege:** Only allow network access to Redis/Valkey instances from necessary hosts.
- **Segmentation:** Place Redis/Valkey behind firewalls and within private subnets.
- **Zero Trust:** Do not trust any network traffic by default.

### Best Practices for Redis/Valkey Network Setup

- Bind Redis to localhost or private IPs only:

  ```
  bind 127.0.0.1 10.0.0.5
  ```

- Disable public access to Redis ports.
- Use VPN or private networking when accessing Redis remotely.
- Implement firewall rules to restrict access to specific IPs or subnets.
- Use security groups (in cloud environments) to limit traffic.

### Firewall Rules and Security Groups

Example firewall rules:

| Source IP/Subnet | Destination Port | Action  | Purpose                     |
|------------------|------------------|---------|-----------------------------|
| 10.0.1.0/24      | 6379             | Allow   | Application servers access  |
| 0.0.0.0/0        | 6379             | Deny    | Block public access         |
| Management subnet | 22 (SSH)         | Allow   | Admin access                |

### Audit Steps for Network Isolation

- **Verify Redis is not listening on public interfaces:**

  ```
  netstat -tulnp | grep redis
  ```

- **Scan ports from outside networks:**

  Use `nmap` to verify no open Redis ports are exposed.

- **Review firewall and security group rules:**

  Confirm only required IP ranges have access.

- **Test connectivity from authorized hosts and deny from unauthorized hosts.**

- **Check for accidental exposure via cloud provider configurations or VPN leaks.**

---

## Worst-Case Scenarios and Incident Response

### Unauthorized Access Cases

**Scenario:** An attacker gains access using default user or weak ACLs.

**Response:**

- Immediately disable or rotate passwords.
- Revoke all active sessions.
- Audit logs to identify compromised data or commands.
- Harden ACLs and enforce multi-factor authentication where possible.

### Command Injection or Misuse

**Scenario:** Dangerous command exploited to delete data or change config.

**Response:**

- Disable dangerous commands using `rename-command`.
- Restore data from backups.
- Investigate the vector used to run the commands.
- Review ACLs and network access policies.

### TLS Certificate Compromise

**Scenario:** TLS private key or client certificates leaked.

**Response:**

- Revoke compromised certificates immediately.
- Generate new certificates.
- Update Redis server and clients with new certificates.
- Monitor for abnormal connection attempts.

### Network Breach

**Scenario:** Unauthorized network access detected.

**Response:**

- Isolate Redis/Valkey instances by modifying firewall rules.
- Conduct full network scan and forensic investigation.
- Check for persistence mechanisms or malware.
- Review and tighten network segmentation.

---

## Summary and Continuous Security Monitoring

Maintaining a secure Redis/Valkey deployment demands:

- **Robust ACL implementation:** Least privilege principles and regular audits.
- **Encrypted communications:** Strict TLS/mTLS enforcement.
- **Command restrictions:** Disable or rename dangerous commands.
- **Strict network isolation:** Firewall rules and private networking.
- **Incident preparedness:** Clear response plans for worst-case scenarios.

### Continuous Monitoring Recommendations:

| Monitoring Aspect            | Tools/Methods                                  | Frequency          |
|-----------------------------|-----------------------------------------------|--------------------|
| ACL Configuration Changes   | Redis logs, configuration management tools    | Continuous/Weekly   |
| TLS Certificate Expiry      | Monitoring scripts, certificate management    | Daily/Weekly       |
| Network Access Logs         | Firewall logs, IDS/IPS                         | Real-time           |
| Command Execution Logs      | Redis slow log, audit plugin                   | Continuous          |
| User Authentication Failures| Redis logs, SIEM integration                   | Real-time           |

By adhering to these comprehensive audit procedures and operational best practices, technical support and operations teams can ensure Redis and Valkey remain resilient against security threats in production.

---

# Appendix

### Example Redis ACL Script for Production

```bash
redis-cli ACL SETUSER default off
redis-cli ACL SETUSER readonly on >ReadPwd_123 ~app:* +@read
redis-cli ACL SETUSER writer on >WritePwd_123 ~app:* +@write
redis-cli ACL SETUSER admin on >AdminPwd_123 allcommands allkeys
```

### Example Redis TLS Startup Command

```bash
redis-server /etc/redis/redis.conf --tls-port 6379 --port 0 \
--tls-cert-file /etc/redis/certs/redis.crt \
--tls-key-file /etc/redis/certs/redis.key \
--tls-ca-cert-file /etc/redis/certs/ca.crt \
--tls-auth-clients yes
```

### Useful Commands for Audit

| Command                      | Purpose                             |
|------------------------------|-----------------------------------|
| `ACL LIST`                   | List all ACL users                 |
| `ACL GETUSER <username>`     | Show user permissions              |
| `COMMAND INFO <cmd>`         | Check command status               |
| `redis-cli --tls`            | Connect with TLS                   |
| `netstat -tulnp`             | Check listening ports              |
| `nmap -p 6379 <host>`        | Scan Redis port                   |

---

**End of Document**