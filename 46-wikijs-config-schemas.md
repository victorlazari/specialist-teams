# Wiki.js Specialist Guide: Configuration Schemas and References

Wiki.js v2.x is a powerful, open-source (AGPL-3.0) wiki platform built on Node.js (>= 20), Vue.js 2, Vuetify, and a GraphQL API. Authored by Nicolas Giard (Requarks), it has become a staple for documentation, knowledge bases, and internal team wikis. This guide provides a comprehensive, deep-dive reference for all configuration schemas, environment variables, deployment manifests, GraphQL types, and module configurations required to operate, troubleshoot, and scale Wiki.js in production environments.

## 1. Core Configuration: `config.yml` Reference

The `config.yml` file is the primary configuration mechanism for Wiki.js when not using environment variables. It dictates how the Node.js application binds to the network, connects to the database, and handles core operational modes. Understanding every parameter in this file is crucial for system administrators.

### 1.1 Network and Server Settings

```yaml
# Port the server should listen on. Default is 3000.
# Note: Binding to ports < 1024 requires setcap or iptables redirect.
port: 3000

# Bind IP address (optional)
# Set to '0.0.0.0' to listen on all interfaces, or '127.0.0.1' for localhost only.
# In Docker, this is typically left as 0.0.0.0.
bindIP: 0.0.0.0

# Maximum request body size limit (e.g., for file uploads via the API or UI)
# Accepts string values like '5mb', '100kb', '1gb'.
bodyParserLimit: 5mb

# Data path for local storage, cache, and temporary files.
# Ensure the Node.js process has read/write permissions to this directory.
dataPath: ./data

# Offline mode
# Disables external network requests for air-gapped environments.
# This prevents Wiki.js from fetching external avatars, telemetry, or updates.
offline: false

# High Availability mode
# Enables multi-node clustering.
# CRITICAL: This strictly requires PostgreSQL as the database backend.
ha: false
```

*Note on High Availability (HA):* Setting `ha: true` enables multi-node clustering. This **strictly requires PostgreSQL** as the database backend because Wiki.js relies on PostgreSQL's `LISTEN/NOTIFY` pub/sub mechanism to synchronize state (like cache invalidation and search index updates) across multiple instances. SQLite cannot be used in HA mode.

### 1.2 Database Configuration (`db` block)

Wiki.js supports multiple database backends. PostgreSQL is highly recommended and required for HA. Other supported databases include MySQL 8+, MariaDB 10.2.7+, MSSQL 2012+, and SQLite 3.9+.

#### PostgreSQL (Recommended)
```yaml
db:
  type: postgres
  host: localhost
  port: 5432
  user: wikijs
  pass: wikijsrocks
  db: wiki
  ssl: false
```

#### MySQL / MariaDB
*Limitation:* MySQL `caching_sha2_password` is not supported by the underlying Node.js driver used by Wiki.js. You must configure your MySQL users to use `mysql_native_password`.
```yaml
db:
  type: mysql
  host: localhost
  port: 3306
  user: wikijs
  pass: wikijsrocks
  db: wiki
  ssl: false
```

#### SQLite
SQLite is suitable for small, single-node deployments or testing.
```yaml
db:
  type: sqlite
  storage: path/to/database.sqlite
```

#### Connection Pool Options (`pool` block)
Wiki.js uses `tarn.js` for connection pooling under the hood (via Knex.js). You can configure the pool behavior to optimize database connections, especially in high-traffic environments or when dealing with strict database connection limits.
```yaml
db:
  type: postgres
  host: localhost
  port: 5432
  user: wikijs
  pass: wikijsrocks
  db: wiki
  pool:
    min: 2
    max: 10
    acquireTimeoutMillis: 30000
    createTimeoutMillis: 30000
    idleTimeoutMillis: 30000
    reapIntervalMillis: 1000
    createRetryIntervalMillis: 200
```

### 1.3 SSL Configuration (`ssl` block)

Wiki.js can handle SSL termination directly, though using a reverse proxy (like Nginx, Traefik, or HAProxy) is generally recommended for production deployments to offload TLS processing.

#### Custom Certificates
```yaml
ssl:
  enabled: true
  provider: custom
  format: pem
  key: /path/to/privkey.pem
  cert: /path/to/cert.pem
  # Optional CA bundle for intermediate certificates
  ca: /path/to/chain.pem
```

#### Let's Encrypt (Automatic)
Wiki.js can automatically provision and renew Let's Encrypt certificates. This requires port 80 to be accessible from the internet for the HTTP-01 challenge.
```yaml
ssl:
  enabled: true
  provider: letsencrypt
  domain: wiki.example.com
  subscriberEmail: admin@example.com
```

### 1.4 Logging Configuration

Proper logging configuration is essential for troubleshooting and auditing.

```yaml
# Log level: error, warn, info, verbose, debug, silly
# Use 'debug' or 'silly' only for troubleshooting as they generate massive output.
logLevel: info

# Log format: default, json
# Use 'json' for structured logging, which is ideal for ingestion into ELK, Datadog, or Splunk.
logFormat: default
```

---

## 2. Environment Variables

For containerized deployments (Docker, Kubernetes), configuring Wiki.js via environment variables is the standard practice. Environment variables override the corresponding values in `config.yml`.

### Database Variables
- `DB_TYPE`: Database type (`postgres`, `mysql`, `mariadb`, `mssql`, `sqlite`).
- `DB_HOST`: Database server hostname or IP address.
- `DB_PORT`: Database server port.
- `DB_USER`: Database username.
- `DB_PASS`: Database password.
- `DB_NAME`: Database name.
- `DB_SSL`: Set to `true` to enable SSL/TLS for the database connection.

### Operational Variables
- `HA_ACTIVE`: Set to `true` to enable High Availability mode (requires PostgreSQL).
- `UPGRADE_COMPANION`: Set to `1` to enable the Wiki.js Update Companion (used in Docker deployments to handle automated updates).
- `PORT`: Overrides the listening port (default 3000). Note: Port < 1024 requires `setcap` or iptables redirect.
- `WIKI_ADMIN_EMAIL`: Used during initial setup to pre-configure the admin user.
- `WIKI_ADMIN_PASSWORD`: Used during initial setup to pre-configure the admin password.

---

## 3. Deployment Manifests

### 3.1 Docker Compose Schema

The standard Docker Compose deployment utilizes the official `ghcr.io/requarks/wiki` image alongside PostgreSQL and the `wiki-update-companion`. The companion container allows Wiki.js to update itself via the web UI by interacting with the Docker socket.

```yaml
version: "3.8"
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
      UPGRADE_COMPANION: 1
    restart: unless-stopped
    ports:
      - "80:3000"
    volumes:
      - wiki-data:/wiki/data

  wiki-update-companion:
    image: ghcr.io/requarks/wiki-update-companion:latest
    container_name: wiki-update-companion
    privileged: true
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - WIKI_CONTAINER_NAME=wiki

volumes:
  db-data:
  wiki-data:
```
*Limitation:* The `wiki-update-companion` container must be updated separately from the main Wiki.js container. It cannot update itself.

### 3.2 Kubernetes Helm `values.yaml` Schema

When deploying via the official Helm chart, the `values.yaml` file controls the deployment schema. Key configuration blocks include replica counts, database connections, ingress rules, and resource limits.

```yaml
replicaCount: 2 # Set > 1 for HA (requires PostgreSQL)

image:
  repository: ghcr.io/requarks/wiki
  tag: "2.5.300"
  pullPolicy: IfNotPresent

# Database configuration (using the bitnami/postgresql subchart)
postgresql:
  enabled: true
  postgresqlUsername: wikijs
  postgresqlPassword: wikijsrocks
  postgresqlDatabase: wiki

# Or use an external database (e.g., AWS RDS, GCP Cloud SQL)
externalDatabase:
  type: postgres
  host: external-db.example.com
  port: 5432
  user: wikijs
  password: wikijsrocks
  database: wiki

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: wiki.example.com
      paths:
        - /
  tls:
    - secretName: wiki-tls
      hosts:
        - wiki.example.com

# High Availability
ha:
  enabled: true # Automatically sets HA_ACTIVE=true in the container environment

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 256Mi
```

---

## 4. GraphQL API Schemas

Wiki.js is built entirely on a GraphQL API, accessible at `/graphql` with Bearer token authentication. Understanding the core types is essential for automation, custom integrations, and headless CMS usage.

### 4.1 `Page` Type Schema

The `Page` type represents a full wiki page, including its content, metadata, and rendering state.

```graphql
type Page {
  id: Int!
  path: String!
  hash: String! # SHA1(locale + path + privateNS)
  title: String!
  description: String
  isPublished: Boolean!
  isPrivate: Boolean!
  privateNS: String
  contentType: String! # e.g., markdown, html, adoc
  createdAt: Date!
  updatedAt: Date!
  editor: String! # markdown, wysiwyg, ckeditor, code, api, asciidoc
  locale: String!
  authorId: Int!
  authorName: String!
  creatorId: Int!
  creatorName: String!
  content: String! # Raw content (e.g., raw markdown)
  render: String # Rendered HTML output
  toc: String # Table of Contents JSON representation
  tags: [Tag]!
}
```
*Note on Page Paths:* Page paths cannot contain dots, spaces, backslashes, or double slashes. They must not start or end with a slash.

### 4.2 `PageListItem` Type Schema

Used in list queries (like search results or recent changes), this is a lightweight version of the `Page` type without the full content payload, optimizing API response times.

```graphql
type PageListItem {
  id: Int!
  path: String!
  title: String!
  description: String
  isPublished: Boolean!
  isPrivate: Boolean!
  privateNS: String
  contentType: String!
  createdAt: Date!
  updatedAt: Date!
  locale: String!
  tags: [Tag]!
}
```

### 4.3 `PageTreeItem` Type Schema

Used for rendering the navigation tree in the sidebar.

```graphql
type PageTreeItem {
  id: Int!
  path: String!
  title: String!
  isFolder: Boolean!
  hasChildren: Boolean!
  parent: Int
  locale: String!
  isPrivate: Boolean!
  privateNS: String
}
```

### 4.4 Example GraphQL Query

To fetch a page by its ID:

```graphql
query {
  pages {
    single(id: 123) {
      id
      title
      path
      content
      render
    }
  }
}
```

---

## 5. Internal Data Formats

### 5.1 Session JSONL Format

Wiki.js stores session data in the database, but when exported or logged, it often follows a JSONL (JSON Lines) format. Each line represents a distinct session state, containing cookie configurations, passport authentication state, and JWT tokens.

```json
{"sid":"sess:1a2b3c4d","sess":{"cookie":{"originalMaxAge":2592000000,"expires":"2023-12-01T12:00:00.000Z","secure":true,"httpOnly":true,"path":"/"},"passport":{"user":1},"jwt":"eyJhbG..."},"expire":"2023-12-01T12:00:00.000Z"}
{"sid":"sess:5e6f7g8h","sess":{"cookie":{"originalMaxAge":2592000000,"expires":"2023-12-02T15:30:00.000Z","secure":true,"httpOnly":true,"path":"/"},"passport":{"user":42},"jwt":"eyJhbG..."},"expire":"2023-12-02T15:30:00.000Z"}
```

---

## 6. Module Configuration Schemas

Wiki.js is highly modular. Configurations for rendering, search, and storage are stored in the database but managed via the UI or API. Below are the underlying configuration schemas for these modules.

### 6.1 Rendering Pipeline Configuration

Wiki.js uses a multi-stage rendering pipeline. It supports 14 Markdown rendering extensions (core, abbr, emoji, expandtabs, footnotes, imsize, katex, kroki, mathjax, multi-table, pivot-table, plantuml, supsub, tasklists) and 11 HTML post-processing extensions (core, asciinema, blockquotes, codehighlighter, diagram/draw.io, image-prefetch, mediaplayers, mermaid, security, tabset, twemoji).

#### Markdown-Core Options
The `markdown-core` extension is the foundation of the rendering pipeline. Its configuration schema includes:

```json
{
  "allowHTML": false,      // Allow raw HTML in markdown (often stripped by security sanitizer)
  "breaks": true,          // Convert \n to <br>
  "linkify": true,         // Autoconvert URL-like text to links
  "typographer": true,     // Enable smartquotes and other typographic replacements
  "quotes": "“”‘’",        // Characters to use for quotes if typographer is true
  "underline": false       // Enable underline syntax
}
```
*Limitation:* Custom HTML is stripped by default by the `security` HTML post-processing extension. If `allowHTML` is true, you must also configure the security sanitizer to allow specific tags, or disable the security sanitizer entirely (not recommended for public wikis).

### 6.2 Search Engine Configuration

Wiki.js supports 8 search engines: Algolia, AWS, Azure, Database (built-in), Elasticsearch, Manticore, PostgreSQL (pg_trgm), Solr, and Sphinx.

#### PostgreSQL (pg_trgm) Schema
When using PostgreSQL, the native `pg_trgm` extension provides excellent search capabilities without external dependencies.
```json
{
  "dict": "english",       // Text search dictionary
  "threshold": 0.3         // Similarity threshold for trigram matching
}
```

#### Elasticsearch Schema
```json
{
  "node": "http://elasticsearch:9200",
  "index": "wikijs",
  "auth": {
    "username": "elastic",
    "password": "changeme"
  },
  "sniffOnStart": false
}
```

#### Algolia Schema
```json
{
  "appId": "YOUR_APP_ID",
  "apiKey": "YOUR_ADMIN_API_KEY",
  "indexName": "wikijs"
}
```

### 6.3 Storage Module Configuration

Wiki.js supports 11 storage backends for syncing content and assets: Azure, Box, DigitalOcean, Disk, Dropbox, Google Drive, Git, OneDrive, S3, S3 Generic, and SFTP.

#### Git Storage Schema
The Git storage module allows two-way synchronization between Wiki.js and a Git repository, enabling a docs-as-code workflow.

```json
{
  "localRepoPath": "./data/repo",
  "branch": "main",
  "remote": "git@github.com:organization/wiki-content.git",
  "authType": "ssh", // Options: basic, ssh, none
  "sshKey": "-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW\n... \n-----END OPENSSH PRIVATE KEY-----",
  "sshPublicKey": "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@host",
  "username": "",    // Used for basic auth
  "password": "",    // Used for basic auth or SSH key passphrase
  "signatureName": "Wiki.js Sync",
  "signatureEmail": "wiki@example.com",
  "syncInterval": "*/5 * * * *" // Cron expression for sync frequency
}
```
*Limitation:* Git storage sync can conflict with direct DB edits if changes occur simultaneously. Version history in Wiki.js automatically creates a snapshot on every update, but Git merge conflicts require manual resolution via the server's filesystem.

#### S3 Storage Schema
```json
{
  "region": "us-east-1",
  "bucket": "my-wiki-assets",
  "accessKeyId": "AKIAIOSFODNN7EXAMPLE",
  "secretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
  "endpoint": "", // Used for S3 Generic (e.g., MinIO, DigitalOcean Spaces)
  "forcePathStyle": false
}
```

### 6.4 Authentication Modules

Wiki.js supports 21 Authentication modules: Auth0, Azure AD, CAS, Discord, Dropbox, Facebook, Firebase, GitHub, GitLab, Google, Keycloak, LDAP/AD, Local, Microsoft, OAuth2, OIDC, Okta, Rocket.chat, SAML 2.0, Slack, and Twitch.

Configuration for these modules typically involves setting a Client ID, Client Secret, and Authorization/Token endpoints. For enterprise environments, SAML 2.0 and LDAP/AD are commonly used.

### 6.5 Comment Systems

Wiki.js supports 4 Comment systems: Artalk, Commento, Default (built-in), and Disqus. The built-in system stores comments directly in the database, while the others require external service configuration (e.g., Disqus shortname).

---

## 7. Known Limitations and Operational Gotchas

When operating Wiki.js in production, be aware of the following architectural constraints and known issues:

1. **Subfolder Installation:** Wiki.js cannot be installed in a subfolder (e.g., `example.com/wiki`). It strictly requires a dedicated subdomain or root domain (e.g., `wiki.example.com`).
2. **Real-time Collaboration:** There is no real-time collaborative editing (like Google Docs). Concurrent edits may result in the last writer winning, though version history is preserved.
3. **HA Constraints:** SQLite cannot be used in High Availability mode. HA requires PostgreSQL.
4. **MySQL Authentication:** MySQL `caching_sha2_password` is not supported. You must configure MySQL users with `mysql_native_password`.
5. **Editor Conversions:** Wiki.js supports 6 editors (markdown, wysiwyg, ckeditor, code, api, asciidoc). Converting a page between editors (e.g., Markdown to WYSIWYG) may result in formatting loss.
6. **Custom HTML:** Custom HTML is stripped by default by the security sanitizer.
7. **Rate Limiting:** There is no native API rate limiting. You must implement rate limiting at the reverse proxy layer (e.g., Nginx `limit_req`).
8. **Git Sync Conflicts:** Git storage sync can conflict with direct DB edits.
9. **Path Restrictions:** Page paths cannot contain dots, spaces, or special characters.
10. **Cloudflare Optimizations:** Cloudflare optimization features (Auto Minify, Mirage, Rocket Loader) break Wiki.js assets. These must be disabled via Page Rules for the Wiki.js domain.
11. **Companion Updates:** The `wiki-update-companion` container must be updated separately from the main application container.
12. **Custom JavaScript:** Custom JS injected via the admin panel must use the specific bootloader hook: `window.boot.register('page-ready', callback)`. Standard `DOMContentLoaded` events may not fire reliably due to the Vue.js SPA architecture.
13. **Privileged Ports:** Binding to a port < 1024 (like 80 or 443) directly requires `setcap` on the Node binary or an iptables redirect, as Wiki.js drops root privileges.

---

## 8. Permissions and Access Control

Wiki.js implements a granular permission system. When configuring roles via the API or database, the following core permission flags are utilized:

- `read:pages`: View published pages.
- `write:pages`: Create and edit pages.
- `manage:pages`: Move, rename, and manage page metadata.
- `delete:pages`: Delete pages.
- `write:styles`: Inject custom CSS.
- `write:scripts`: Inject custom JavaScript.
- `read:source`: View the raw source code of a page.
- `read:history`: View page version history.
- `manage:system`: Access the administration area.

These permissions are mapped to user groups and can be scoped to specific namespaces or page paths using rules.

---

## 9. Advanced Troubleshooting

### 9.1 Database Connection Issues
If Wiki.js fails to start with database connection errors, verify the `pool` configuration in `config.yml`. In environments with aggressive connection culling (like some serverless databases or strict firewalls), you may need to decrease `idleTimeoutMillis` and enable `createRetryIntervalMillis`.

### 9.2 Search Index Corruption
If the built-in database search or an external search engine (like Elasticsearch) stops returning accurate results, you can trigger a full reindex via the GraphQL API:

```graphql
mutation {
  searchRebuildIndex {
    success
    message
  }
}
```

### 9.3 Asset Loading Failures
If the UI loads but assets (CSS/JS) return 404s or fail to execute:
1. Verify the `Site URL` in the administration panel exactly matches the URL used to access the site (including `http/https`).
2. If behind Cloudflare, ensure Rocket Loader and Auto Minify are disabled.
3. Check the `dataPath` in `config.yml` to ensure the application has write permissions to the cache directory.

By understanding and properly managing these configuration schemas, you can ensure a stable, secure, and performant Wiki.js deployment tailored to your organization's needs.
