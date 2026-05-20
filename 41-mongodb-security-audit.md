# MongoDB Security Audit Procedures: Comprehensive Guide for Tech Support Operations

## Introduction

In modern production environments, MongoDB often serves as the primary data store for mission-critical applications, housing sensitive customer data, financial records, and proprietary business information. As a tech support specialist, database administrator, or security engineer, conducting a rigorous security audit is not just a compliance checkbox—it is a fundamental operational requirement to prevent data breaches, unauthorized access, and catastrophic data loss. 

The landscape of database security is constantly evolving, with threats ranging from automated ransomware bots scanning for open ports to sophisticated state-sponsored actors attempting lateral movement within a network. Furthermore, regulatory frameworks such as GDPR, HIPAA, PCI-DSS, and SOC 2 mandate strict controls over data access, encryption, and auditing. Failure to comply can result in severe financial penalties and irreparable reputational damage.

This document provides an exhaustive, highly detailed, and practical guide to auditing MongoDB security. It focuses heavily on production operations, worst-case scenarios, and actionable tech support procedures. The audit covers Role-Based Access Control (RBAC), Transport Layer Security (TLS/mTLS), Field Level Encryption (FLE), auditing setup, network isolation, and the critical step of disabling server-side JavaScript execution. 

---

## 1. Role-Based Access Control (RBAC) and Authentication Audit

Role-Based Access Control (RBAC) is the first line of defense in MongoDB. A misconfigured RBAC system can lead to privilege escalation, unauthorized data modification, or complete database compromise.

### 1.1. Auditing Built-in and Custom Roles

MongoDB provides several built-in roles (e.g., `read`, `readWrite`, `dbAdmin`, `userAdmin`, `clusterAdmin`). However, relying solely on built-in roles often violates the principle of least privilege, especially in complex microservices architectures.

**Audit Procedure:**
1. **List All Users and Roles:** Execute the following command to retrieve all users and their assigned roles across all databases. This should be automated via a script that runs weekly.
   ```javascript
   use admin;
   db.system.users.find({}, { user: 1, db: 1, roles: 1 }).pretty();
   ```
2. **Identify Over-Privileged Users:** Look for users with `root`, `dbOwner`, or `userAdminAnyDatabase` roles. These roles should be strictly limited to administrative accounts and never used by application service accounts. Any service account found with these roles must be flagged as a critical security violation.
3. **Review Custom Roles:** Custom roles allow for granular permissions. Audit custom roles to ensure they do not inadvertently grant excessive privileges.
   ```javascript
   use admin;
   db.getRoles({ rolesInfo: 1, showPrivileges: true, showBuiltinRoles: false });
   ```

### 1.2. External Authentication (LDAP, Active Directory, SAML, OIDC)

Enterprise environments rarely rely on MongoDB's internal SCRAM authentication for human users. Instead, they integrate with centralized identity providers (IdPs).

**Tech Support Operation:**
- **Audit LDAP/AD Integration:** Verify that MongoDB is configured to use secure LDAP (LDAPS) over port 636. Unencrypted LDAP (port 389) transmits credentials in plaintext and must be disabled.
- **Group Mapping:** Ensure that LDAP groups are correctly mapped to MongoDB roles. A common misconfiguration is mapping a broad AD group (e.g., "All Engineering") to a highly privileged MongoDB role (e.g., `dbAdmin`).
- **Session Expiration:** For SAML/OIDC, audit the session timeout configurations. Idle sessions should be forcefully terminated after a predefined period (e.g., 60 minutes) to prevent session hijacking.

### 1.3. Enforcing the Principle of Least Privilege

Tech support operations frequently encounter scenarios where developers request `dbOwner` access for troubleshooting production issues. This must be strictly denied.

**Actionable Steps:**
- **Service Accounts:** Application service accounts must only have `readWrite` access to specific collections, not the entire database.
- **Support Accounts:** Tech support personnel should use `read` only roles for investigation. If write access is required, it must be granted temporarily via a "break-glass" procedure, requiring approval from a security manager, and heavily audited.
- **Orphaned Accounts:** Regularly audit and remove accounts belonging to former employees or deprecated applications. Implement an automated offboarding script that revokes database access immediately upon employee termination.

### 1.4. Worst-Case Scenario: Compromised Admin Credentials

If an attacker compromises an account with `userAdminAnyDatabase`, they can create hidden backdoor accounts, grant themselves access to all data, and cover their tracks by modifying audit logs (if not properly secured).
**Response:** 
1. Immediately revoke the compromised credentials.
2. Audit the `system.users` collection for any recently created accounts and delete unauthorized entries.
3. Rotate all database credentials, including application service accounts.
4. Review centralized audit logs to determine the extent of data accessed and identify any data exfiltration.

---

## 2. Transport Layer Security (TLS/mTLS)

Data in transit must be encrypted to prevent eavesdropping, man-in-the-middle (MITM) attacks, and credential sniffing. MongoDB supports TLS for client-to-server and server-to-server (intra-cluster) communication.

### 2.1. TLS Configuration Audit

Ensure that TLS is not only enabled but strictly enforced across all nodes in the cluster.

**Audit Procedure:**
1. **Check `mongod.conf`:** Verify the `net.tls` configuration block on every node (Primary, Secondaries, and Hidden nodes).
   ```yaml
   net:
     tls:
       mode: requireTLS
       certificateKeyFile: /etc/ssl/mongodb.pem
       CAFile: /etc/ssl/ca.pem
       disabledProtocols: TLS1_0,TLS1_1
   ```
2. **Verify `requireTLS`:** The `mode` must be set to `requireTLS`. Settings like `allowTLS` or `preferTLS` are unacceptable in production as they allow unencrypted fallback connections, which attackers can exploit via downgrade attacks.
3. **Cipher Suites:** Audit the allowed cipher suites. Ensure that weak or deprecated ciphers (e.g., RC4, DES, 3DES) are explicitly disabled. Only strong ciphers (e.g., AES-GCM, ChaCha20) should be permitted.

### 2.2. Mutual TLS (mTLS) for Client Authentication

For highly secure environments (e.g., financial institutions, healthcare providers), mTLS ensures that only clients with a valid cryptographic certificate signed by a trusted Certificate Authority (CA) can connect to the database. This eliminates reliance on passwords.

**Audit Procedure:**
1. **Verify mTLS Enforcement:** Check if `net.tls.CAFile` is configured and `net.tls.allowConnectionsWithoutCertificates` is set to `false` (which is the default when `CAFile` is present).
2. **Certificate Revocation:** Ensure a Certificate Revocation List (CRL) is configured (`net.tls.CRLFile`) to block compromised client certificates. Tech support must have a documented procedure for updating the CRL immediately when a client device is lost or compromised.

### 2.3. Certificate Rotation and Expiry Monitoring

Expired certificates will cause immediate, catastrophic cluster outages as nodes fail to communicate with each other, triggering election failures, and clients are rejected.

**Tech Support Operation:**
- Implement automated monitoring (e.g., via Prometheus/Grafana, Datadog, or custom Nagios scripts) to alert on certificate expiry at least 30, 15, and 7 days in advance.
- Document and practice the zero-downtime certificate rotation procedure. This involves:
  1. Updating the certificates on all secondary nodes and restarting them one by one.
  2. Stepping down the primary node (`rs.stepDown()`).
  3. Updating the certificate on the former primary and restarting it.

---

## 3. Field Level Encryption (FLE)

While TLS protects data in transit and Transparent Data Encryption (TDE) protects data at rest on the disk, Field Level Encryption (FLE) protects sensitive data *in use*. With Client-Side Field Level Encryption (CSFLE) or Queryable Encryption, the database server never sees the plaintext data. The data is encrypted by the application driver before it leaves the application server.

### 3.1. Auditing CSFLE Implementation

CSFLE requires the application to encrypt data before sending it to MongoDB. The database only stores ciphertext.

**Audit Procedure:**
1. **Identify Sensitive Fields:** Work with compliance and legal teams to identify PII, PHI, or PCI data (e.g., Social Security Numbers, credit card numbers, medical diagnoses, birth dates).
2. **Verify Schema Validation:** Ensure MongoDB schema validation is configured to reject unencrypted data for sensitive fields. This prevents developers from accidentally inserting plaintext data.
   ```javascript
   db.runCommand({
     collMod: "patients",
     validator: {
       $jsonSchema: {
         bsonType: "object",
         properties: {
           ssn: {
             bsonType: "binData",
             description: "SSN must be encrypted (binData)"
           },
           credit_card: {
             bsonType: "binData",
             description: "Credit card must be encrypted"
           }
         }
       }
     }
   });
   ```
3. **Key Management Service (KMS) Integration:** Verify that the Customer Master Key (CMK) is securely stored in a managed KMS (AWS KMS, Azure Key Vault, GCP KMS, or HashiCorp Vault). Audit the IAM policies to ensure the application has the minimum required permissions (e.g., `kms:Encrypt`, `kms:Decrypt`) and that the database servers have *no* access to the KMS.

### 3.2. Queryable Encryption vs. CSFLE

Audit the architecture to ensure the correct encryption technology is used. CSFLE supports exact match queries, while Queryable Encryption (introduced in MongoDB 6.0) supports more complex queries (e.g., equality, range, prefix) on encrypted data without decrypting it on the server. Ensure the performance overhead of Queryable Encryption has been benchmarked and approved.

### 3.3. Worst-Case Scenario: KMS Outage or Key Deletion

If the KMS is unreachable or the CMK is deleted, the application will be unable to decrypt data, resulting in a massive application-level outage and potential data loss (if the key is permanently destroyed).
**Response:** 
- Ensure KMS high availability is configured across multiple regions.
- Implement strict IAM policies preventing accidental or malicious deletion of the CMK.
- Maintain secure, offline backups of the CMK if supported by the KMS provider.
- Tech support must have a runbook for failing over to a backup KMS region.

---

## 4. Auditing Setup and Configuration

MongoDB Enterprise and MongoDB Atlas provide robust auditing capabilities to track administrative and data access operations. Without auditing, forensic analysis during a security incident is impossible.

### 4.1. Enabling the Audit Log

The audit log must be enabled to track who did what, when, and from where.

**Audit Procedure:**
1. **Check Configuration:** Verify the `auditLog` section in `mongod.conf`.
   ```yaml
   auditLog:
     destination: file
     format: JSON
     path: /var/log/mongodb/audit.json
   ```
2. **Format:** Ensure the format is set to `JSON` for easy ingestion, parsing, and indexing by SIEM tools (e.g., Splunk, ELK stack, Datadog).

### 4.2. Audit Filters and Performance Impact

Logging every single read and write operation (DML) will severely degrade database performance, increase latency, and consume massive amounts of disk space, potentially causing the database to crash due to disk exhaustion.

**Tech Support Operation:**
- Configure audit filters to capture only critical events: authentication failures, schema changes (DDL), role modifications, user creation/deletion, and access to highly sensitive collections.
- **Example Filter:**
  ```yaml
  auditLog:
    filter: '{ atype: { $in: [ "authenticate", "createCollection", "dropCollection", "grantRolesToUser", "createUser", "dropUser" ] } }'
  ```
- **Performance Monitoring:** Monitor disk I/O and CPU usage closely after enabling or modifying auditing filters. Ensure the audit log resides on a separate dedicated disk volume to prevent it from impacting the database storage engine.

### 4.3. Centralized Log Management and Immutable Storage

Audit logs stored locally on the database server are vulnerable to tampering. If an attacker gains root access to the server, they can delete or modify the `audit.json` file to hide their tracks.
**Requirement:** Forward all audit logs in real-time to a centralized, immutable SIEM system using agents like Filebeat, Fluentd, or Splunk Universal Forwarder. Set up automated alerts for:
- Multiple failed authentication attempts (brute-force detection).
- Unauthorized role changes or privilege escalation.
- Access from unexpected IP addresses.

---

## 5. Network Isolation and Firewalling

MongoDB should never be directly exposed to the public internet. Network isolation is a critical layer of defense against automated scanning, brute-force attacks, and zero-day exploits.

### 5.1. Bind IP Configuration

By default, MongoDB binds to localhost. When configuring for a network, it must only bind to internal, private IP addresses.

**Audit Procedure:**
1. **Check `net.bindIp`:** Ensure it does not contain `0.0.0.0` or any public IP addresses.
   ```yaml
   net:
     bindIp: 127.0.0.1,10.0.1.15
   ```

### 5.2. VPC Peering and PrivateLink

For cloud deployments (e.g., MongoDB Atlas), utilize VPC Peering or AWS PrivateLink / Azure Private Link / GCP Private Service Connect.

**Tech Support Operation:**
- Audit the network architecture to ensure all application-to-database traffic flows exclusively over private, internal networks.
- Verify that public IP access is completely disabled in the Atlas console or cloud provider settings.
- For tech support access, require engineers to connect via a secure VPN or a hardened Bastion Host (Jump Box) with MFA enabled. Direct SSH access to database nodes should be disabled.

### 5.3. Security Groups and Firewall Rules

Implement strict firewall rules (Security Groups in AWS, NSGs in Azure, iptables locally) at the network level.

**Audit Procedure:**
- **Inbound Rules:** Only allow inbound traffic on the MongoDB port (default 27017) from the specific IP CIDR blocks of the application servers, monitoring tools, or bastion hosts.
- **Outbound Rules:** Restrict outbound traffic from the database servers to only necessary destinations (e.g., KMS endpoints, monitoring services, backup storage). Block all other outbound internet access to prevent reverse shells or data exfiltration.

### 5.4. Worst-Case Scenario: Ransomware via Exposed Port

If a misconfiguration exposes port 27017 to the internet without authentication, automated ransomware bots will connect, wipe the database, and leave a ransom note within minutes.
**Response:** 
1. Immediately isolate the compromised node from the network by modifying Security Groups.
2. Do not pay the ransom.
3. Initiate the disaster recovery (DR) plan by restoring from the latest immutable, offsite backup.
4. Conduct a post-mortem to identify the network misconfiguration (e.g., a developer accidentally opening the port for testing) that allowed public access and implement preventative guardrails (e.g., AWS Config rules).

---

## 6. Disabling Server-Side JavaScript Execution

MongoDB supports the execution of JavaScript code on the server for certain operations (e.g., `$where` clauses, `mapReduce`, and `group` commands). This feature introduces significant security risks, including NoSQL injection and Denial of Service (DoS) attacks.

### 6.1. The Risks of Server-Side JavaScript

Allowing server-side JS execution opens the door to malicious payloads. An attacker who can inject JavaScript into a query can potentially execute arbitrary code, access the file system, or consume all CPU resources by running infinite loops, effectively taking down the database.

### 6.2. Disabling JS in `mongod.conf`

Modern MongoDB applications should use the Aggregation Framework instead of `mapReduce` or `$where`. The Aggregation Framework is faster, more efficient, and significantly more secure. Therefore, server-side JavaScript should be disabled in all production environments.

**Audit Procedure:**
1. **Check Configuration:** Verify that `security.javascriptEnabled` is set to `false` in `mongod.conf`.
   ```yaml
   security:
     javascriptEnabled: false
   ```
2. **Application Compatibility:** Before disabling this in production, tech support must work with the development team to ensure no legacy applications rely on `$where` or `mapReduce`. 
3. **Migration Assistance:** Provide guidance on rewriting legacy queries. For example, a `$where` query like `db.users.find({ $where: "this.credits > this.debits" })` should be rewritten using the aggregation pipeline: `db.users.find({ $expr: { $gt: [ "$credits", "$debits" ] } })`.

### 6.3. Worst-Case Scenario: NoSQL Injection via `$where`

If JavaScript is enabled and an application improperly sanitizes user input, an attacker could inject a payload like `db.users.find({ $where: "this.username == '" + userInput + "'" })`. If `userInput` is `"' || true || '"`, the attacker bypasses authentication and retrieves all user records.
**Response:** 
1. Identify the vulnerable application endpoint via application logs and MongoDB audit logs.
2. Temporarily disable the endpoint or implement strict input sanitization (e.g., using parameterized queries or strict type checking).
3. Permanently disable server-side JavaScript on the database to mitigate the root cause and prevent future occurrences.

---

## 7. Comprehensive Incident Response and Tech Support Workflows

A security audit is incomplete without defining the procedures for responding to security incidents. Tech support teams must be prepared for the worst-case scenarios and have documented runbooks.

### 7.1. Handling Suspected Data Exfiltration

If monitoring tools detect an unusually high volume of outbound network traffic from the database servers, or if the SIEM alerts on a massive data export operation, it may indicate data exfiltration.

**Tech Support Workflow:**
1. **Isolate:** Immediately restrict network access to the affected database node, allowing only connections from the incident response team's bastion host. Do not shut down the server, as this destroys volatile memory (RAM) needed for forensics.
2. **Investigate:** Analyze the MongoDB audit logs and network flow logs to identify the source IP, the authenticated user, the collections accessed, and the volume of data transferred.
3. **Revoke:** Terminate all active sessions for the compromised user using `db.killOp()` and immediately revoke their credentials.
4. **Report:** Escalate to the security operations center (SOC) and legal teams, providing a detailed timeline, the scope of the potential breach, and the affected data categories.

### 7.2. Routine Security Health Checks

Tech support should not wait for an annual compliance audit. Implement automated, weekly security health checks to ensure continuous compliance.

**Checklist:**
- Verify no new users have been granted `root`, `dbOwner`, or `userAdminAnyDatabase` roles.
- Confirm TLS certificates are valid for at least the next 30 days.
- Ensure the audit log is actively writing and being successfully forwarded to the SIEM.
- Run an automated network scan (e.g., using Nmap from an external perspective) to confirm port 27017 is not publicly accessible.
- Validate that server-side JavaScript remains disabled across all nodes.
- Test the restoration of immutable backups to ensure data can be recovered in the event of a ransomware attack.

---

## Conclusion

Securing a MongoDB production environment requires a multi-layered, defense-in-depth approach, encompassing strict access controls, robust encryption (in transit, at rest, and in use), comprehensive auditing, and rigorous network isolation. By meticulously applying the audit procedures outlined in this document, tech support and operations teams can significantly reduce the attack surface, ensure compliance with stringent regulatory standards, and protect the organization's most valuable asset: its data. Continuous monitoring, automated alerting, and well-rehearsed incident response plans are not optional—they are essential components of maintaining a secure, resilient, and highly available database infrastructure.
