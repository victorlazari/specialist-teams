# PostgreSQL 15+ Security Audit Procedures: Comprehensive Guide

## 1. Introduction to PostgreSQL Security Architecture

PostgreSQL 15 introduces and refines several security features that are critical for enterprise-grade deployments. A comprehensive security audit must cover multiple layers of the database architecture, from network-level access controls to granular row-level security policies. This document outlines the standard operating procedures (SOPs) for conducting a thorough security audit of a PostgreSQL 15+ environment, focusing on Role-Based Access Control (RBAC), Row-Level Security (RLS), SSL/TLS configuration, `pg_hba.conf` management, data encryption (at rest and in transit), and comprehensive auditing using the `pgaudit` extension.

This guide is designed for database administrators (DBAs), security engineers, and tech support operations teams managing large-scale, mission-critical PostgreSQL deployments. It emphasizes production operations, worst-case scenarios, huge datasets, and database migrations.

---

## 2. Role-Based Access Control (RBAC) Audit

Role-Based Access Control in PostgreSQL is managed through roles, which can act as both users and groups. A robust RBAC implementation ensures the principle of least privilege.

### 2.1 Auditing Role Attributes

PostgreSQL roles can have various attributes such as `SUPERUSER`, `CREATEDB`, `CREATEROLE`, `REPLICATION`, and `BYPASSRLS`. 

**Audit Procedure:**
1.  **Identify Superusers:** Superusers bypass all permission checks. Their use must be strictly limited.
    ```sql
    SELECT rolname FROM pg_roles WHERE rolsuper = true;
    ```
    *Action:* Verify that only authorized administrative accounts possess the `SUPERUSER` attribute. Application users must **never** be superusers.

2.  **Review Role Creation Privileges:** Roles with `CREATEROLE` can create new roles and grant privileges, potentially escalating their own privileges.
    ```sql
    SELECT rolname FROM pg_roles WHERE rolcreaterole = true;
    ```
    *Action:* Restrict `CREATEROLE` to specific security administration roles.

3.  **Check for BYPASSRLS:** Roles with this attribute bypass Row-Level Security policies.
    ```sql
    SELECT rolname FROM pg_roles WHERE rolbypassrls = true;
    ```
    *Action:* Ensure only backup/restore or specific administrative roles have this attribute.

### 2.2 Auditing Object Privileges

Privileges on database objects (tables, views, functions) must be explicitly granted.

**Audit Procedure:**
1.  **Review Default Privileges:** Default privileges dictate what permissions are automatically granted when new objects are created.
    ```sql
    SELECT pg_get_userbyid(d.defaclrole) AS role,
           n.nspname AS schema,
           d.defaclobjtype AS object_type,
           d.defaclacl AS default_privileges
    FROM pg_default_acl d
    LEFT JOIN pg_namespace n ON n.oid = d.defaclnamespace;
    ```
    *Action:* Ensure default privileges do not grant excessive access (e.g., `PUBLIC` should not have `CREATE` on the `public` schema, a change introduced in PG 15).

2.  **Analyze Table-Level Grants:**
    ```sql
    SELECT grantee, table_schema, table_name, privilege_type
    FROM information_schema.role_table_grants
    WHERE grantee != 'postgres';
    ```
    *Action:* Verify that application roles only have `SELECT`, `INSERT`, `UPDATE`, or `DELETE` as required. Revoke `TRUNCATE` or `REFERENCES` if not explicitly needed.

---

## 3. Row-Level Security (RLS) Configuration and Audit

Row-Level Security allows database administrators to restrict which rows a user can access or modify based on their role or current session context. This is crucial for multi-tenant applications.

### 3.1 Enabling and Verifying RLS

**Audit Procedure:**
1.  **Identify Tables with RLS Enabled:**
    ```sql
    SELECT relname, relrowsecurity, relforcerowsecurity
    FROM pg_class
    WHERE relkind = 'r' AND relnamespace = 'public'::regnamespace;
    ```
    *Action:* Ensure `relrowsecurity` is `true` for all sensitive tables. If `relforcerowsecurity` is `false`, table owners bypass RLS. Consider setting it to `true` if owners should also be restricted.

2.  **Review RLS Policies:**
    ```sql
    SELECT schemaname, tablename, policyname, permissive, roles, cmd, qual, with_check
    FROM pg_policies;
    ```
    *Action:* Analyze the `qual` (USING clause) and `with_check` (WITH CHECK clause) expressions. Ensure they correctly filter data based on the session context (e.g., `current_user` or `current_setting('app.tenant_id')`).

### 3.2 Worst-Case Scenario: RLS Bypass

A common vulnerability is poorly written functions running with `SECURITY DEFINER` that inadvertently bypass RLS.

**Audit Procedure:**
1.  **Identify SECURITY DEFINER Functions:**
    ```sql
    SELECT proname, proowner::regrole
    FROM pg_proc
    WHERE prosecdef = true;
    ```
    *Action:* Review the source code of these functions. Ensure they do not expose data that should be protected by RLS. Always set `search_path` explicitly within `SECURITY DEFINER` functions to prevent search path hijacking.

---

## 4. Network Security: pg_hba.conf and SSL/TLS

Network-level security is the first line of defense. PostgreSQL uses `pg_hba.conf` for client authentication and supports SSL/TLS for encrypted connections.

### 4.1 Auditing pg_hba.conf

The `pg_hba.conf` file controls which hosts are allowed to connect, how clients are authenticated, which PostgreSQL user names they can use, and which databases they can access.

**Audit Procedure:**
1.  **Review Connection Rules:**
    Examine the `pg_hba.conf` file (typically located in the data directory).
    ```text
    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    host    all             all             0.0.0.0/0               scram-sha-256
    ```
    *Action:* 
    *   **Eliminate `trust` authentication:** Never use `trust` over network connections.
    *   **Enforce `scram-sha-256`:** Ensure all password-based authentication uses `scram-sha-256` (the default in PG 14+). Avoid `md5`.
    *   **Restrict IP Addresses:** Replace `0.0.0.0/0` with specific application server IP ranges or subnets.
    *   **Database and User Restrictions:** Specify exact databases and users instead of using `all` where possible.

### 4.2 SSL/TLS Configuration

Data in transit must be encrypted to prevent eavesdropping and man-in-the-middle (MITM) attacks.

**Audit Procedure:**
1.  **Verify SSL is Enabled:**
    ```sql
    SHOW ssl;
    ```
    *Action:* Must return `on`.

2.  **Check SSL Parameters:**
    ```sql
    SHOW ssl_ciphers;
    SHOW ssl_min_protocol_version;
    ```
    *Action:* Ensure `ssl_min_protocol_version` is at least `TLSv1.2` (preferably `TLSv1.3`). Restrict `ssl_ciphers` to strong cipher suites (e.g., `HIGH:!aNULL:!MD5`).

3.  **Enforce SSL Connections:**
    In `pg_hba.conf`, use `hostssl` instead of `host` to mandate encrypted connections.
    ```text
    hostssl all             all             10.0.0.0/8              scram-sha-256
    ```

4.  **Client Certificate Authentication:** For high-security environments, configure `cert` authentication in `pg_hba.conf` to require clients to present a valid SSL certificate.

---

## 5. Data Encryption at Rest

While PostgreSQL does not have native Transparent Data Encryption (TDE) in the community edition (as of PG 15), encryption at rest must be implemented at the filesystem or block storage level.

### 5.1 Filesystem/Block Level Encryption

**Audit Procedure:**
1.  **Verify Storage Encryption:** Check the underlying infrastructure (e.g., AWS EBS encryption, Azure Disk Encryption, Linux LUKS).
    *Action:* Ensure the volume hosting the PostgreSQL data directory (`$PGDATA`) is encrypted using strong algorithms (e.g., AES-256) and that encryption keys are managed securely via a Key Management Service (KMS).

### 5.2 Column-Level Encryption (pgcrypto)

For highly sensitive data (e.g., PII, financial records), column-level encryption can be used via the `pgcrypto` extension.

**Audit Procedure:**
1.  **Identify Encrypted Columns:** Review schema definitions for columns using `pgp_sym_encrypt` or similar functions.
2.  **Key Management:** Ensure encryption keys are not hardcoded in application code or stored in plain text within the database. Keys should be injected via environment variables or retrieved from a secure vault at runtime.

---

## 6. Comprehensive Auditing with pgaudit

The `pgaudit` extension provides detailed session and object audit logging via the standard PostgreSQL logging facility. It is essential for compliance (e.g., HIPAA, PCI-DSS, SOC 2).

### 6.1 Installation and Configuration

**Audit Procedure:**
1.  **Verify Extension Installation:**
    ```sql
    SELECT extname, extversion FROM pg_extension WHERE extname = 'pgaudit';
    ```
    *Action:* Ensure `pgaudit` is installed and loaded via `shared_preload_libraries` in `postgresql.conf`.

2.  **Review pgaudit Settings:**
    ```sql
    SHOW pgaudit.log;
    SHOW pgaudit.role;
    ```
    *Action:* 
    *   `pgaudit.log` should be configured to capture relevant statement classes (e.g., `read, write, ddl, role`). Avoid setting it to `all` in high-transaction environments due to performance overhead and log volume.
    *   `pgaudit.role` can be used to audit specific roles (e.g., an `auditor` role).

### 6.2 Log Management and Retention

Audit logs are useless if they are tampered with or deleted.

**Audit Procedure:**
1.  **Log Destination:** Ensure logs are shipped to a centralized, immutable logging system (e.g., Splunk, ELK stack, AWS CloudWatch) immediately.
2.  **Log Format:** Configure `log_destination = 'csvlog'` or `jsonlog` (PG 15+) for easier parsing by SIEM tools.
3.  **Retention Policy:** Verify that logs are retained according to organizational compliance requirements (e.g., 1 year, 7 years).

---

## 7. Worst-Case Scenarios and Incident Response

Tech support operations must be prepared for security incidents.

### 7.1 Scenario: Compromised Application Credentials

**Symptoms:** Unusual data access patterns, unexpected data modifications, connections from unknown IP addresses.

**Response Procedure:**
1.  **Identify the Compromised Role:** Use `pgaudit` logs to trace the malicious activity.
2.  **Revoke Access:** Immediately change the password for the compromised role or lock the account.
    ```sql
    ALTER ROLE app_user NOLOGIN;
    ```
3.  **Terminate Active Sessions:**
    ```sql
    SELECT pg_terminate_backend(pid)
    FROM pg_stat_activity
    WHERE usename = 'app_user';
    ```
4.  **Investigate Impact:** Analyze audit logs to determine the extent of data exfiltration or modification.
5.  **Restore Data:** If data was maliciously altered, initiate a point-in-time recovery (PITR) using WAL archives.

### 7.2 Scenario: Privilege Escalation via SECURITY DEFINER

**Symptoms:** A low-privileged user executing administrative commands or accessing restricted data.

**Response Procedure:**
1.  **Identify the Vulnerable Function:** Review `pgaudit` logs for unexpected function executions.
2.  **Revoke Execute Privilege:**
    ```sql
    REVOKE EXECUTE ON FUNCTION vulnerable_func() FROM PUBLIC;
    ```
3.  **Patch the Function:** Rewrite the function to explicitly set `search_path` and validate inputs, or remove the `SECURITY DEFINER` attribute if not strictly necessary.

---

## 8. Database Migrations and Security

Security must be maintained during database migrations (e.g., upgrading from PG 13 to PG 15, or migrating to a new cloud provider).

### 8.1 Pre-Migration Audit

1.  **Export Roles and Privileges:** Use `pg_dumpall --roles-only` to capture the current RBAC state.
2.  **Review Deprecated Features:** Ensure no deprecated security features are in use (e.g., `exclusive backup` mode, older authentication methods).

### 8.2 Post-Migration Verification

1.  **Verify pg_hba.conf:** Ensure the new environment has a restrictive `pg_hba.conf`. Cloud providers often have permissive defaults.
2.  **Test RLS Policies:** Run automated tests to verify that RLS policies function correctly in the new environment.
3.  **Confirm SSL/TLS:** Verify that the new endpoints enforce SSL/TLS with the correct certificates.
4.  **Validate pgaudit:** Ensure `pgaudit` is loaded and actively logging to the new centralized logging system.

---

## 9. Conclusion

Securing a PostgreSQL 15+ environment requires a multi-layered approach. By rigorously auditing RBAC, enforcing RLS, securing network connections with `pg_hba.conf` and SSL/TLS, ensuring data encryption, and maintaining comprehensive audit logs with `pgaudit`, organizations can protect their critical data against both internal and external threats. Continuous monitoring and regular security audits are essential to adapt to evolving security landscapes and maintain compliance.
