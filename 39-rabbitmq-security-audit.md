# RabbitMQ Security Audit Procedures and Tech Support Operations Guide

## 1. Introduction and Scope

In modern distributed architectures, RabbitMQ serves as the central nervous system, routing millions of messages between microservices, legacy systems, and third-party integrations. As the critical data plane for asynchronous communication, its security posture is paramount. A compromised RabbitMQ cluster can lead to data exfiltration, denial of service, unauthorized message injection, and complete system compromise. 

This document provides a comprehensive, production-grade security audit procedure for RabbitMQ environments. It is specifically designed for tech support operations, site reliability engineers (SREs), and security auditors who must ensure that RabbitMQ deployments are resilient against both external attacks and internal misconfigurations. The procedures detailed herein focus on worst-case scenarios, massive message backlogs, and high-stakes production environments where downtime or data loss is unacceptable.

The scope of this audit encompasses Transport Layer Security (TLS) and mutual TLS (mTLS) configurations, Role-Based Access Control (RBAC), Lightweight Directory Access Protocol (LDAP) integration, securing the Management UI, and implementing strict resource consumption limits per user to prevent noisy neighbor problems and resource exhaustion attacks.

## 2. Transport Layer Security (TLS) and Mutual TLS (mTLS) Configuration

### 2.1. TLS Configuration Audit

Transport Layer Security is the first line of defense against eavesdropping and man-in-the-middle (MitM) attacks. In a production environment, unencrypted AMQP traffic (port 5672) must be strictly prohibited. All client-to-node and node-to-node communication must occur over TLS (port 5671).

**Audit Procedures:**

1. **Verify Listener Configuration:** Inspect the `rabbitmq.conf` file to ensure that the non-TLS listener is disabled and only the TLS listener is active.
   ```ini
   # Disable non-TLS listener
   listeners.tcp = none
   
   # Enable TLS listener on standard port
   listeners.ssl.default = 5671
   ```

2. **Cipher Suites and Protocol Versions:** Legacy protocols such as TLS 1.0 and TLS 1.1 are deprecated and vulnerable to various attacks. Ensure that only TLS 1.2 and TLS 1.3 are permitted. Furthermore, restrict the cipher suites to strong, modern algorithms.
   ```ini
   ssl_options.versions.1 = tlsv1.3
   ssl_options.versions.2 = tlsv1.2
   ssl_options.ciphers.1  = TLS_AES_256_GCM_SHA384
   ssl_options.ciphers.2  = TLS_AES_128_GCM_SHA256
   ssl_options.ciphers.3  = TLS_CHACHA20_POLY1305_SHA256
   ```

3. **Certificate Validation:** Ensure that the server certificate is valid, not expired, and issued by a trusted Certificate Authority (CA). The `ssl_options.cacertfile`, `ssl_options.certfile`, and `ssl_options.keyfile` must point to the correct paths with appropriate file permissions (read-only by the `rabbitmq` user).

### 2.2. Mutual TLS (mTLS) Enforcement

For highly sensitive environments, mTLS provides cryptographic proof of client identity, eliminating reliance on passwords that can be leaked or brute-forced.

**Audit Procedures:**

1. **Enforce Client Certificate Verification:** The `verify` option must be set to `verify_peer`, and `fail_if_no_peer_cert` must be `true`.
   ```ini
   ssl_options.verify = verify_peer
   ssl_options.fail_if_no_peer_cert = true
   ```

2. **Certificate Revocation Lists (CRL):** A compromised client certificate must be revoked immediately. Audit the CRL configuration to ensure RabbitMQ checks for revoked certificates.
   ```ini
   ssl_options.crl_check = true
   ssl_options.crl_cache_hash_dir = /etc/rabbitmq/crl
   ```

3. **Tech Support Scenario: mTLS Troubleshooting:** When clients fail to connect via mTLS, tech support must verify the certificate chain. Use `openssl s_client -connect rabbitmq.internal:5671 -cert client.crt -key client.key -CAfile ca.crt` to diagnose handshake failures. Common issues include clock skew, missing intermediate certificates, or incorrect Subject Alternative Names (SANs).

## 3. Role-Based Access Control (RBAC) and Least Privilege

RabbitMQ's internal authorization mechanism relies on virtual hosts (vhosts), users, and permissions. A flat permission structure is a significant security risk.

### 3.1. User and Vhost Isolation

**Audit Procedures:**

1. **Default Credentials:** The default `guest` user must be deleted or its password changed, and it must be restricted from connecting remotely. By default, RabbitMQ prevents `guest` from connecting via non-loopback interfaces, but this must be explicitly verified.
   ```bash
   rabbitmqctl delete_user guest
   ```

2. **Vhost Segregation:** Different applications and environments (e.g., dev, staging, prod) must operate in separate vhosts. Audit the vhost list and ensure no cross-contamination exists.
   ```bash
   rabbitmqctl list_vhosts
   ```

### 3.2. Granular Permissions

Permissions in RabbitMQ are defined by regular expressions for configure, write, and read operations.

**Audit Procedures:**

1. **Review Permission Matrices:** Export and review the permissions for all users. Ensure that applications only have access to the specific queues and exchanges they require.
   ```bash
   rabbitmqctl list_permissions -p /production_vhost
   ```

2. **Tech Support Scenario: Rogue Consumer:** In a worst-case scenario where a compromised service begins consuming messages from an unauthorized queue, tech support must immediately revoke the user's read permissions.
   ```bash
   rabbitmqctl clear_permissions -p /production_vhost compromised_user
   ```

3. **Topic Authorization:** For topic exchanges, standard permissions are insufficient. Audit topic permissions to ensure routing key restrictions are enforced.
   ```bash
   rabbitmqctl list_topic_permissions -p /production_vhost
   ```

## 4. LDAP Integration and Centralized Identity Management

Managing local RabbitMQ users does not scale and violates enterprise security policies that mandate centralized identity management. Integrating RabbitMQ with LDAP or Active Directory (AD) ensures that access is tied to corporate identities and is automatically revoked upon employee offboarding.

### 4.1. LDAP Configuration Audit

**Audit Procedures:**

1. **Authentication Backend Order:** Ensure that the LDAP backend is prioritized over the internal database, or that the internal database is disabled entirely for human users.
   ```ini
   auth_backends.1 = ldap
   auth_backends.2 = internal
   ```

2. **Secure LDAP (LDAPS):** Communication between RabbitMQ and the LDAP server must be encrypted. Verify that `auth_ldap.servers` points to the LDAPS port (typically 636) and that `auth_ldap.use_ssl` is `true`.
   ```ini
   auth_ldap.servers.1 = ldaps://ldap.internal.company.com
   auth_ldap.port = 636
   auth_ldap.use_ssl = true
   ```

3. **Query Optimization and Caching:** In environments with massive message throughput, LDAP queries can become a bottleneck, leading to connection timeouts and message backlogs. Audit the LDAP caching configuration to ensure performance stability.
   ```ini
   auth_ldap.cache.enabled = true
   auth_ldap.cache.ttl = 300000 # 5 minutes
   ```

### 4.2. Tech Support Scenario: LDAP Outage

If the LDAP server goes down, RabbitMQ clients may fail to authenticate, causing a massive backlog of unacknowledged messages and connection retries. Tech support must have a break-glass procedure. This involves maintaining a highly restricted, heavily monitored local admin account that can be used to bypass LDAP during an outage.

## 5. Securing the Management UI and API

The RabbitMQ Management UI and HTTP API provide powerful administrative capabilities. If exposed, they offer attackers a direct vector to manipulate the cluster, delete queues, or extract sensitive configuration data.

### 5.1. Network Exposure and TLS

**Audit Procedures:**

1. **Internal Network Only:** The Management UI (port 15672) must never be exposed to the public internet. It should only be accessible via a secure VPN, bastion host, or internal management network.

2. **Enforce HTTPS:** Similar to AMQP traffic, the Management UI must enforce HTTPS.
   ```ini
   management.ssl.port       = 15671
   management.ssl.cacertfile = /etc/rabbitmq/ca.crt
   management.ssl.certfile   = /etc/rabbitmq/server.crt
   management.ssl.keyfile    = /etc/rabbitmq/server.key
   ```

### 5.2. API Rate Limiting and Monitoring

**Audit Procedures:**

1. **Reverse Proxy Integration:** Place the Management UI behind a reverse proxy (e.g., Nginx, HAProxy) to enforce rate limiting, IP whitelisting, and Web Application Firewall (WAF) rules.
2. **Audit Logging:** Enable and monitor the RabbitMQ audit log plugin. Every action performed via the Management UI or API must be logged and forwarded to a centralized SIEM (Security Information and Event Management) system.
   ```bash
   rabbitmq-plugins enable rabbitmq_auth_backend_ldap rabbitmq_audit
   ```

## 6. Limiting Resource Consumption Per User

A critical aspect of RabbitMQ security is availability. A malicious or poorly written client can exhaust cluster resources (memory, disk space, file descriptors), leading to a denial of service for all other tenants.

### 6.1. Connection and Channel Limits

**Audit Procedures:**

1. **Max Connections:** Audit the maximum number of connections allowed per user. A single application should not be able to consume all available file descriptors.
   ```bash
   rabbitmqctl set_user_limits application_user '{"max-connections": 100}'
   ```

2. **Max Channels:** Channels multiplex over a single connection. Excessive channels consume memory. Limit the number of channels per connection.
   ```ini
   channel_max = 50
   ```

### 6.2. Queue Length and Memory Limits

In a worst-case scenario, a consumer crashes, but publishers continue sending messages. This creates a massive message backlog that can crash the RabbitMQ node due to memory exhaustion.

**Audit Procedures:**

1. **Queue Length Limits:** Enforce maximum queue lengths or sizes using policies. When the limit is reached, configure the queue to either drop the oldest messages or reject new publishes.
   ```bash
   rabbitmqctl set_policy MaxLength "^critical_" '{"max-length":100000, "overflow":"reject-publish"}' --apply-to queues
   ```

2. **Message TTL (Time-To-Live):** Ensure that messages do not reside in queues indefinitely. Apply TTL policies to automatically discard stale data.
   ```bash
   rabbitmqctl set_policy TTL "^transient_" '{"message-ttl":60000}' --apply-to queues
   ```

3. **Memory Alarms:** Verify the high-water mark for memory usage. When RabbitMQ hits this threshold, it blocks all publishers to prevent a crash.
   ```ini
   vm_memory_high_watermark.relative = 0.4
   ```

### 6.3. Tech Support Scenario: Massive Message Backlog

When a massive backlog occurs, tech support must act quickly to stabilize the cluster. 
1. **Identify the Offender:** Use `rabbitmqctl list_queues name messages memory` to find the bloated queue.
2. **Halt Publishers:** Temporarily revoke write permissions for the publishing application to stop the influx of messages.
3. **Purge or Shovel:** If the messages are expendable, purge the queue. If they are critical, use the Shovel plugin to move them to a secondary cluster for processing, thereby relieving pressure on the primary production cluster.

## 7. Conclusion

Securing a RabbitMQ cluster is an ongoing process that requires continuous auditing, monitoring, and strict enforcement of least privilege. By implementing robust TLS/mTLS configurations, granular RBAC, centralized LDAP authentication, secure management interfaces, and aggressive resource limits, organizations can ensure the confidentiality, integrity, and availability of their critical messaging infrastructure. Tech support and SRE teams must be intimately familiar with these configurations and the associated emergency procedures to respond effectively to worst-case scenarios and maintain operational stability.
