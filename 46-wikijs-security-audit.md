# Wiki.js Security Audit & Hardening Guide

## 1. Introduction
Wiki.js (v2.x) is a powerful, open-source (AGPL-3.0) wiki platform built on Node.js (>= 20), Vue.js 2, Vuetify, and a robust GraphQL API. Created by Nicolas Giard (Requarks), it has become a staple for enterprise knowledge management. While it offers extensive features—including 21 authentication modules, 11 storage backends, 8 search engines, and multiple editors—securing a production deployment requires a comprehensive, defense-in-depth approach. 

This document serves as a deep-dive security audit and hardening guide for Wiki.js. It covers everything from authentication and API security to infrastructure hardening, database encryption, and container security. Whether you are deploying via Docker, Kubernetes, or bare metal, adhering to these guidelines will ensure your knowledge base remains secure against internal and external threats.

## 2. Authentication Module Selection & Best Practices
Wiki.js supports an impressive array of 21 authentication modules, including Auth0, Azure AD, CAS, Discord, Dropbox, Facebook, Firebase, GitHub, GitLab, Google, Keycloak, LDAP/AD, Local, Microsoft, OAuth2, OIDC, Okta, Rocket.chat, SAML 2.0, Slack, and Twitch. Selecting and configuring the right module is critical for enterprise security.

### LDAP / Active Directory Integration
When integrating with legacy LDAP or Active Directory environments, security must be prioritized at the network and protocol levels.
- **Secure Transport (LDAPS):** Ensure that connections are secured using LDAPS (LDAP over SSL/TLS) on port 636, rather than plain text LDAP on port 389. Plain text LDAP transmits credentials in the clear, making them susceptible to packet sniffing.
- **Service Accounts:** Use a dedicated service account with read-only permissions to bind to the directory. Never use domain admin credentials or accounts with write access to the directory.
- **Search Base Optimization:** Restrict the search base to the specific organizational unit (OU) containing the users who need access to the wiki. This minimizes the attack surface and improves query performance.
- **Group Mapping:** Map LDAP groups to Wiki.js groups to automate role-based access control (RBAC). This ensures users are automatically deprovisioned or downgraded when their roles change in the central directory.

### SAML 2.0 and OIDC (OpenID Connect)
For modern enterprise environments, SAML 2.0 or OIDC (via providers like Keycloak, Okta, or Azure AD) are the strongly recommended authentication methods.
- **Enforce MFA at the IdP:** Enforce Multi-Factor Authentication (MFA) at the Identity Provider (IdP) level. Wiki.js relies entirely on the IdP for authentication verification, so securing the IdP is paramount.
- **Token Expiration:** Configure short-lived access tokens and refresh tokens in your OIDC provider to minimize the window of opportunity for stolen tokens.
- **Attribute Mapping:** Carefully map attributes (e.g., email, name, groups) to ensure accurate user provisioning. Avoid mapping sensitive attributes that are not required by Wiki.js. Ensure that the `email` attribute is verified by the IdP before trusting it in Wiki.js.

### Local Authentication Risks
If local authentication must be used, enforce strong password policies. However, administrators must be aware that Wiki.js does not have built-in brute force protection or account lockout mechanisms for local accounts. This critical security gap must be mitigated at the reverse proxy or Web Application Firewall (WAF) level.

## 3. API Token Management and Permission Scopes
Wiki.js exposes a powerful GraphQL endpoint at `/graphql` secured via Bearer token authentication. API tokens are used for programmatic access, automation, and third-party integrations.

### Token Generation and Storage
- **Principle of Least Privilege:** Generate API tokens with the absolute minimum permissions required for the specific task. Wiki.js permissions are granular and include `read:pages`, `write:pages`, `manage:pages`, `delete:pages`, `write:styles`, `write:scripts`, `read:source`, `read:history`, and `manage:system`.
- **Secure Storage:** Treat API tokens like highly sensitive passwords. Store them securely in a secrets management system (e.g., HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) and never hardcode them in scripts, CI/CD pipelines, or source code repositories.

### High-Risk Permission Scopes
Carefully assign scopes to API tokens. For example, a backup script that only needs to export pages should only be granted `read:pages` and `read:source`.
- **The Danger of `manage:system`:** The `manage:system`, `write:styles`, and `write:scripts` scopes are extremely high-risk. Tokens with these scopes can completely compromise the wiki, alter configurations, or inject malicious code. They should be strictly controlled, rarely issued, and heavily audited.

### Example: Secure GraphQL Query
When querying the API, always enforce HTTPS to protect the Bearer token in transit. Pass the token in the Authorization header:
```bash
curl -X POST https://wiki.example.com/graphql \
  -H "Authorization: Bearer YOUR_HIGHLY_SECURE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "query { pages { list { id path title isPrivate } } }"}'
```

## 4. HTML Security Sanitizer & Custom Script Injection Risks
Wiki.js supports 6 different editors: markdown, wysiwyg (html), ckeditor (html), code (html), api (yml), and asciidoc (adoc). To protect against Cross-Site Scripting (XSS) attacks, Wiki.js employs an HTML security sanitizer (whitelist-based filtering) by default during its post-processing phase.

### The Security Sanitizer Mechanism
The built-in security sanitizer aggressively strips out potentially dangerous HTML tags and attributes, such as `<script>`, `<iframe>`, `object`, `embed`, and inline event handlers like `onload` or `onerror`. 
- **Do Not Disable:** Never disable the security sanitizer unless absolutely necessary for a specific, highly trusted internal use case. Disabling it opens the door to stored XSS attacks, allowing malicious users to execute arbitrary JavaScript in the browsers of other users.
- **Handling Custom HTML:** If users need to embed custom content (e.g., diagrams, videos, complex tables), mandate the use of built-in extensions (e.g., diagram via draw.io, mediaplayers, plantuml) rather than allowing raw HTML bypasses.

### Custom Script Injection Risks
Wiki.js allows administrators to inject custom CSS and JavaScript globally via the administration panel. This feature requires the `write:styles` and `write:scripts` permissions.
- **High Risk Vector:** These permissions are equivalent to full administrative control over the frontend application. A malicious user or compromised account with `write:scripts` can inject keyloggers, steal session tokens, redirect users to phishing sites, or deface the entire wiki.
- **Strict Access Control:** Only grant these permissions to highly trusted, senior administrators.
- **Code Review Process:** Any custom scripts should be thoroughly reviewed for security vulnerabilities and performance impacts before being injected into the production environment.
- **Implementation Note:** Custom JS must use `window.boot.register('page-ready', callback)` to execute properly within the Vue.js lifecycle. Failure to do so may result in race conditions or broken functionality.

## 5. Reverse Proxy Hardening & SSL/TLS Configuration
Wiki.js should never be exposed directly to the internet. It is designed to be deployed behind a robust reverse proxy such as Nginx, Apache, HAProxy, or Traefik. The reverse proxy is responsible for SSL/TLS termination, security header injection, and rate limiting.

### SSL/TLS Configuration Best Practices
- **Let's Encrypt vs Custom Certs:** Use Let's Encrypt for automated, free certificates, or provision custom enterprise certificates. Ensure that the reverse proxy is configured to use strong cipher suites and modern TLS versions (strictly TLS 1.2 and TLS 1.3). Disable older, vulnerable protocols like TLS 1.0 and 1.1.
- **HSTS Implementation:** Enable HTTP Strict Transport Security (HSTS) to force clients to connect via HTTPS, preventing SSL stripping attacks.

### Nginx Hardening Example
Below is an example of a hardened Nginx configuration tailored for Wiki.js:

```nginx
server {
    listen 443 ssl http2;
    server_name wiki.example.com;

    ssl_certificate /etc/letsencrypt/live/wiki.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/wiki.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Content Security Policy (CSP)
    # Note: Adjust CSP based on your specific extensions, search engines (e.g., Algolia), and integrations
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' wss:;" always;

    # Block hidden files and directories
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support for potential future features or extensions
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## 6. Database Security
Wiki.js supports multiple database engines: PostgreSQL (recommended, required for HA), MySQL 8+, MariaDB 10.2.7+, MSSQL 2012+, and SQLite 3.9+. PostgreSQL is highly recommended for production deployments due to its robustness, performance, and support for High Availability (HA) mode.

### Connection Encryption
Ensure that the connection between the Wiki.js Node.js application and the database is encrypted using SSL/TLS. This prevents man-in-the-middle (MITM) attacks on the database traffic, which could expose sensitive wiki content or user credentials.
- In `config.yml`, configure the database connection to require SSL. For PostgreSQL, this often involves setting `ssl: true` in the database configuration block.

### User Privileges and Isolation
- **Dedicated User:** Create a dedicated database user specifically for Wiki.js. Never use the `postgres`, `root`, or `sa` administrative users.
- **Least Privilege:** Grant the Wiki.js database user only the permissions necessary to operate on its specific database (e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `ALTER`, `DROP`, `INDEX` on the `wikijs` database).
- **MySQL Limitation Warning:** Note that MySQL's `caching_sha2_password` authentication plugin is not supported by the underlying Node.js driver used by Wiki.js. You must configure the Wiki.js database user to use `mysql_native_password`.

## 7. CORS, CSP Headers, and Brute Force Protection

### Cross-Origin Resource Sharing (CORS)
Wiki.js API endpoints should be protected by strict CORS policies. If you are building custom integrations that query the GraphQL API from a browser, ensure that the reverse proxy only allows CORS requests from explicitly trusted origins. Wildcard (`*`) CORS policies should be strictly prohibited in production.

### Content Security Policy (CSP) Deep Dive
A robust CSP is critical for mitigating XSS attacks. The CSP should restrict the sources from which scripts, styles, images, and other resources can be loaded.
- **The Vue.js Challenge:** Wiki.js relies heavily on inline scripts and styles for its Vue.js frontend and various rich text editors. A strict CSP without `'unsafe-inline'` and `'unsafe-eval'` will break the application's core functionality.
- **Pragmatic Mitigation:** Use a pragmatic CSP (as shown in the Nginx example above) that allows necessary inline execution but restricts external domains. If you use external services like Algolia for search or AWS S3 for storage, you must explicitly add their domains to the respective CSP directives (e.g., `connect-src`, `img-src`).

### Brute Force Protection Implementation
As noted earlier, Wiki.js lacks native rate limiting or brute force protection. This must be implemented externally.
- **Fail2Ban Integration:** Use Fail2Ban to parse reverse proxy access and error logs. Configure it to block IP addresses that exhibit brute-force behavior (e.g., multiple failed POST requests to `/login` or `/graphql` within a short timeframe).
- **Nginx Rate Limiting:** Implement strict rate limiting on authentication and API endpoints to prevent resource exhaustion and credential stuffing.

```nginx
# Define rate limit zones
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=graphql_limit:10m rate=30r/m;

server {
    # ...
    location /login {
        limit_req zone=login_limit burst=5 nodelay;
        proxy_pass http://127.0.0.1:3000;
        # ...
    }
    
    location /graphql {
        limit_req zone=graphql_limit burst=20 nodelay;
        proxy_pass http://127.0.0.1:3000;
        # ...
    }
}
```

## 8. Account Verification & Rate Limiting
- **Email Verification:** If local authentication is enabled, strictly enforce email verification for all new accounts. This prevents automated bots from creating fake accounts and spamming the wiki.
- **GraphQL Rate Limiting:** The GraphQL endpoint (`/graphql`) is particularly susceptible to resource exhaustion attacks. Complex, deeply nested queries can consume significant CPU and database resources. Rate limiting at the reverse proxy is essential to maintain availability.

## 9. File Upload Security
Wiki.js allows users to upload images and attachments. This presents a significant security risk, including malware distribution and storage exhaustion.
- **Storage Backends:** Wiki.js supports 11 storage backends (e.g., S3, Azure, DigitalOcean, Disk). Using cloud object storage (like AWS S3 or Azure Blob Storage) is strongly recommended over local disk storage. It isolates uploaded files from the application server, preventing local file inclusion (LFI) or remote code execution (RCE) vulnerabilities if a malicious file is somehow executed.
- **File Type Restrictions:** Configure Wiki.js to only allow specific, safe file extensions (e.g., `.jpg`, `.png`, `.pdf`, `.docx`). Explicitly reject dangerous extensions like `.exe`, `.sh`, `.php`, `.html`, or `.js`.
- **Malware Scanning:** In high-security environments, integrate a reverse proxy or ICAP server to scan uploaded files for malware using ClamAV or a commercial antivirus solution before they reach the storage backend.

## 10. Private Pages, Namespace Isolation, and Group-Based Access Control
Wiki.js uses a robust Group-Based Access Control (GBAC) system to manage permissions.
- **Cryptographic Page Hashes:** Page hashes are calculated as `SHA1(locale + path + privateNS)`. This ensures that private namespaces are cryptographically separated in the database, preventing direct object reference (IDOR) attacks from guessing page IDs.
- **Namespace Isolation:** Use namespaces to isolate sensitive documentation (e.g., HR, Finance, Security). Assign permissions to these namespaces strictly based on groups.
- **Default Group Auditing:** Carefully review the permissions assigned to the "Guest" and "Default" groups. Ensure that anonymous users cannot access private namespaces, view source code (`read:source`), or perform unauthorized actions.
- **Path Rules:** Remember that page paths cannot contain dots, spaces, backslashes, or double slashes. No starting or ending slashes are allowed. This strict routing prevents directory traversal attacks but requires strict naming conventions.

## 11. Audit Logging and Monitoring
Comprehensive audit logging is essential for incident response, compliance, and forensic analysis.
- **Log Level Configuration:** Set the `logLevel` in `config.yml` to `info` for production environments. Use `debug` only for temporary troubleshooting, as it may expose sensitive data.
- **Centralized Logging:** Forward Wiki.js logs (stdout/stderr) and reverse proxy logs to a centralized logging system (e.g., ELK stack, Splunk, Datadog, Graylog) for analysis, retention, and alerting.
- **Monitor High-Risk Actions:** Create automated alerts for high-risk actions, such as:
  - Changes to user permissions or group assignments.
  - Spikes in failed login attempts.
  - Modifications to custom scripts or styles.
  - Unexpected database connection errors.

## 12. Docker & Kubernetes Security
Wiki.js is commonly deployed using Docker (`ghcr.io/requarks/wiki`) or Kubernetes (via the official Helm chart). Securing the container runtime is critical.

### Docker Security Best Practices
- **Non-Root User Execution:** The official Docker image is designed to run as the `node` user (UID 1000) by default. Ensure that you do not override this to run as `root`. Running as a non-root user mitigates the impact of container breakout vulnerabilities.
- **Read-Only Filesystem:** Run the container with a read-only root filesystem to prevent attackers from modifying the application code, installing unauthorized packages, or dropping malware.
```yaml
# docker-compose.yml security-hardened example
services:
  wiki:
    image: ghcr.io/requarks/wiki:2
    user: "1000:1000"
    read_only: true
    tmpfs:
      - /tmp
      - /wiki/data/cache
    volumes:
      - ./config.yml:/wiki/config.yml:ro
      - wiki-data:/wiki/data
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
```
- **wiki-update-companion Risks:** If using the `wiki-update-companion` container for automated updates, be aware that it requires access to the Docker socket (`/var/run/docker.sock`). This is a massive security risk, as access to the Docker socket is equivalent to root access on the host. It is generally safer to manage updates via standard CI/CD pipelines rather than exposing the Docker socket.

### Kubernetes Security Best Practices
- **Network Policies:** Implement strict Kubernetes Network Policies to restrict traffic to the Wiki.js pods. Only allow ingress from the designated Ingress Controller and egress to the database and necessary external services (e.g., IdP, cloud storage backends). Deny all other traffic by default.
- **Secrets Management:** Never store the `config.yml` or database credentials in plain text ConfigMaps. Use Kubernetes Secrets, or better yet, integrate with an external secrets manager like HashiCorp Vault or AWS Secrets Manager using the CSI Secrets Store driver.
- **Pod Security Context:** Enforce strict security contexts on the pods to ensure they run with the least privilege necessary:
```yaml
# Kubernetes Pod Security Context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

## 13. Known Limitations & Security Trade-offs
When auditing and deploying Wiki.js, administrators must be aware of the following known limitations and their security implications:
1. **Subfolder Installation:** Wiki.js cannot be installed in a subfolder (e.g., `example.com/wiki`); it requires a dedicated subdomain (e.g., `wiki.example.com`). This actually improves security by simplifying cookie scoping and reducing the risk of session fixation or cross-site scripting attacks from neighboring applications on the same domain.
2. **SQLite in HA:** SQLite cannot be used in High Availability (HA) mode. For HA, PostgreSQL is strictly required. Using SQLite in a production environment presents an availability risk and should be avoided for mission-critical deployments.
3. **CloudFlare Optimization Conflicts:** CloudFlare optimizations (Auto Minify, Mirage, Rocket Loader) break Wiki.js assets and Vue.js hydration. You must disable these features (often via Page Rules) to ensure the frontend functions correctly and securely.
4. **Git Storage Sync Conflicts:** Git storage sync can conflict with direct database edits. If using Git as a storage backend, ensure strict access controls on the Git repository, as commits directly modify wiki content, bypassing the Wiki.js application layer security.
5. **Port Binding Restrictions:** Binding to ports < 1024 requires `setcap` or iptables redirects. It is significantly safer to run Wiki.js on an unprivileged port (e.g., 3000) and use a reverse proxy to bind to port 443.
6. **Editor Conversion Data Loss:** Converting between the 6 different editors may lose formatting. While not a direct security vulnerability, it can lead to data integrity issues.
7. **Real-time Collaboration:** Wiki.js v2.x does not support real-time collaborative editing. This prevents race conditions but may impact user workflows.

## Conclusion
Securing Wiki.js requires a comprehensive, multi-layered approach. By carefully selecting authentication modules, strictly managing API tokens, hardening the reverse proxy, securing the database connection, and enforcing container security best practices, organizations can safely deploy Wiki.js as a robust and secure enterprise knowledge base. Continuous monitoring, regular updates, and strict access control are essential to maintaining a strong security posture against evolving threats.
