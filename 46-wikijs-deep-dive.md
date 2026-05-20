# Wiki.js Architecture Deep Dive: A Comprehensive Guide

## 1. Introduction to Wiki.js Architecture

Wiki.js is a modern, open-source wiki platform built on Node.js, designed for performance, extensibility, and ease of use. Currently in its stable v2.x release, Wiki.js leverages a robust technology stack to deliver a seamless experience for both administrators and end-users. The core architecture is built upon Node.js (requiring version 20 or higher), utilizing the Express.js framework for HTTP server capabilities, and exposing a comprehensive GraphQL API for all frontend and external interactions.

The frontend is a Single Page Application (SPA) powered by Vue.js 2 and the Vuetify material design framework. This separation of concerns between the backend API and the frontend client allows for high flexibility and performance. The backend interacts with various relational databases through the Objection.js Object-Relational Mapping (ORM) library, which itself is built on top of the Knex.js query builder. Supported databases include PostgreSQL (which is highly recommended and strictly required for High Availability setups), MySQL 8+, MariaDB 10.2.7+, MSSQL 2012+, and SQLite 3.9+.

Wiki.js is licensed under the AGPL-3.0 license and is primarily authored by Nicolas Giard (Requarks). Its architecture is highly modular, supporting a wide array of editors, rendering extensions, authentication strategies, storage backends, and search engines. This deep dive will explore the intricate details of the Wiki.js request lifecycle, rendering pipeline, page models, version history, and more, providing a thorough understanding of its internal workings.

## 2. Request Lifecycle: From Client to Database

The request lifecycle in Wiki.js is a well-defined path that ensures security, efficiency, and consistency. When a client makes a request, it typically follows this trajectory: Express > GraphQL > Objection.js > Knex > Database.

### Express.js Middleware
At the outermost layer, Express.js handles incoming HTTP requests. It manages routing, static file serving, and initial middleware processing. Security headers, CORS configurations, and rate limiting (though native rate limiting is not present and requires a reverse proxy) are handled here. The Express server listens on the configured port (defined in `config.yml`) and routes API requests to the GraphQL endpoint at `/graphql`.

### GraphQL API Layer
Wiki.js relies heavily on GraphQL for its API. Instead of traditional REST endpoints, clients send GraphQL queries and mutations to the `/graphql` endpoint. This endpoint is protected by Bearer token authentication. The GraphQL layer defines the schema, types, and resolvers. When a query is received, the corresponding resolver is invoked. Resolvers are responsible for validating permissions (e.g., `read:pages`, `write:pages`, `manage:system`) and orchestrating the business logic.

```graphql
# Example GraphQL Query to fetch a page
query {
  pages {
    single(id: 123) {
      id
      path
      title
      description
      content
      contentType
      createdAt
      updatedAt
    }
  }
}
```

### Objection.js and Knex.js
Once the resolver has validated the request, it interacts with the database using Objection.js. Objection.js is an ORM that provides a structured way to define models and relationships. It translates JavaScript objects and methods into SQL queries. Under the hood, Objection.js uses Knex.js, a powerful SQL query builder. Knex.js handles the actual generation of SQL dialects specific to the configured database (PostgreSQL, MySQL, etc.) and manages connection pooling.

### Database Execution
Finally, the generated SQL query is executed against the database. The results are returned through Knex.js to Objection.js, which instantiates the corresponding model objects. These objects are then formatted by the GraphQL resolver and sent back to the client as a JSON response.

## 3. Rendering Pipeline Internals

The rendering pipeline in Wiki.js is a sophisticated process that transforms raw content from various editors into safe, optimized HTML for display. Wiki.js supports 6 different editors: `markdown`, `wysiwyg` (HTML), `ckeditor` (HTML), `code` (HTML), `api` (YML), and `asciidoc` (ADOC).

### Markdown Rendering (markdown-it)
For Markdown content, Wiki.js uses `markdown-it` as its core rendering engine. The rendering process is highly extensible, utilizing a chain of 14 Markdown rendering extensions. These include:
- `core`: Basic Markdown syntax.
- `abbr`: Abbreviations.
- `emoji`: Emoji shortcodes.
- `expandtabs`: Tab expansion.
- `footnotes`: Footnote support.
- `imsize`: Image sizing.
- `katex` / `mathjax`: Mathematical formulas.
- `kroki` / `plantuml`: Diagram generation.
- `multi-table` / `pivot-table`: Advanced tables.
- `supsub`: Superscript and subscript.
- `tasklists`: Task lists.

### HTML Post-Processing
Once the raw content is converted to HTML (or if the content was already HTML from a WYSIWYG editor), it passes through an HTML post-processing phase. This phase uses 11 extensions to enhance the HTML:
- `core`: Basic HTML adjustments.
- `asciinema`: Asciinema player integration.
- `blockquotes`: Enhanced blockquotes.
- `codehighlighter`: Syntax highlighting for code blocks.
- `diagram` (draw.io): Draw.io diagram rendering.
- `image-prefetch`: Optimizing image loading.
- `mediaplayers`: Audio and video player embedding.
- `mermaid`: Mermaid.js diagram rendering.
- `security`: Crucial security sanitization.
- `tabset`: Tabbed content.
- `twemoji`: Twitter emoji integration.

### Security Sanitization
The `security` post-processing extension is critical. It uses libraries like DOMPurify to strip out potentially malicious HTML tags and attributes, preventing Cross-Site Scripting (XSS) attacks. By default, custom HTML is aggressively stripped. If administrators need to allow specific custom HTML, they must carefully configure the sanitization rules, balancing functionality with security.

### Caching the Rendered Output
To ensure high performance, the final rendered HTML is cached. This page render cache prevents the system from having to re-parse and re-render complex Markdown or diagrams on every page load. The cache is invalidated when the page content is updated.

## 4. Page Model Internals

The Page model is central to Wiki.js. It encapsulates all metadata and content associated with a wiki page.

### Hash Generation
Every page in Wiki.js is uniquely identified by a hash. This hash is generated using the SHA1 algorithm based on three components: the locale, the path, and the private namespace (privateNS).
`Page Hash = SHA1(locale + path + privateNS)`
This ensures that pages with the same path but in different languages or namespaces are treated as distinct entities. Page paths have strict rules: they cannot contain dots, spaces, backslashes, or double slashes, and they cannot start or end with a slash.

### Content Types
The `contentType` field in the Page model indicates the format of the raw content. This determines which editor was used and which rendering pipeline should be applied. Supported content types include `markdown`, `html`, `yml`, and `adoc`.

### Table of Contents (TOC) Generation
During the rendering pipeline, Wiki.js automatically generates a Table of Contents (TOC) by extracting heading tags (H1, H2, H3, etc.) from the rendered HTML. This TOC is stored alongside the page metadata and is used by the frontend to display a navigation sidebar.

## 5. Version History Implementation

Wiki.js maintains a comprehensive version history for all pages, ensuring that no data is ever permanently lost due to accidental edits or malicious changes.

### Automatic Snapshots
The version history system operates on an automatic snapshot mechanism. Every time a page is updated, a new snapshot of the page's content and metadata is created and stored in the database. This happens transparently during the Objection.js model lifecycle hooks (e.g., `beforeUpdate`).

### Restore Mechanism
Administrators and users with the `read:history` and `write:pages` permissions can view the version history of a page. The frontend provides an interface to compare different versions and restore a previous version. When a restore action is triggered, the system creates a *new* version that contains the content of the restored version, preserving the linear history of changes.

## 6. Search Engine Abstraction Layer

Wiki.js features a powerful search engine abstraction layer that allows administrators to choose the best search backend for their needs. It supports 8 different search engines:
- Algolia
- AWS CloudSearch
- Azure Search
- Database (built-in, basic LIKE queries)
- Elasticsearch
- Manticore
- PostgreSQL (pg_trgm, highly recommended for PostgreSQL users)
- Solr
- Sphinx

The abstraction layer defines a common interface for indexing documents, updating indexes, and performing search queries. When a page is created, updated, or deleted, the event system triggers an update to the active search engine index. This ensures that search results are always up-to-date.

## 7. Storage Sync Architecture

To facilitate backups, offline access, and version control, Wiki.js includes a robust storage sync architecture. It supports 11 storage backends, including Azure, Box, DigitalOcean, Disk, Dropbox, Google Drive, Git, OneDrive, S3, S3 Generic, and SFTP.

### Bidirectional Git Sync
The Git storage backend is particularly powerful, offering bidirectional synchronization. When a page is modified in Wiki.js, the changes are committed and pushed to the configured Git repository. Conversely, if changes are pushed to the Git repository externally, Wiki.js can pull those changes and update the database. However, this can lead to conflicts if direct database edits and Git edits occur simultaneously.

### Disk Export and Cloud Storage Push
Other storage backends typically operate in a push-only mode (export). Wiki.js can be configured to periodically export the entire wiki content as Markdown files (or other raw formats) to a local disk directory or push them to cloud storage providers like AWS S3 or Google Drive. This provides a reliable backup mechanism.

## 8. Authentication Flow

Security and access control are paramount in Wiki.js. The authentication flow is built on top of Passport.js, a popular authentication middleware for Node.js.

### Passport.js Strategies
Wiki.js supports an impressive 21 authentication modules, covering almost every conceivable identity provider. These include Auth0, Azure AD, CAS, Discord, Dropbox, Facebook, Firebase, GitHub, GitLab, Google, Keycloak, LDAP/AD, Local (built-in username/password), Microsoft, OAuth2, OIDC, Okta, Rocket.chat, SAML 2.0, Slack, and Twitch.

When a user attempts to log in, the request is routed to the appropriate Passport.js strategy. The strategy handles the interaction with the external identity provider (e.g., redirecting to Google for OAuth2 consent).

### JWT Tokens and Session Management
Upon successful authentication, Wiki.js generates a JSON Web Token (JWT). This token contains the user's identity and permissions. The JWT is sent back to the client and stored (typically in local storage or a cookie). For all subsequent requests to the GraphQL API, the client includes this JWT in the `Authorization: Bearer <token>` header. The backend verifies the JWT signature and extracts the user information, establishing the session context for the request.

## 9. Caching Layer

To deliver fast response times, Wiki.js employs a multi-tiered caching layer.

### Page Render Cache
As mentioned earlier, the final rendered HTML of a page is cached. This is the most critical cache for read performance, as it bypasses the entire rendering pipeline for frequently accessed pages.

### Tree Cache
Wiki.js caches the hierarchical structure of the wiki (the navigation tree). Generating this tree requires complex database queries to resolve parent-child relationships and permissions. Caching the tree significantly speeds up the initial load of the frontend SPA.

### Search Index Cache
Some search engines (like the built-in database search) utilize caching to store recent search results, reducing the load on the database for common queries.

## 10. Event System (WIKI.events)

Wiki.js utilizes an internal event emitter system, accessible via `WIKI.events`, to facilitate inter-module communication. This decoupled architecture allows different parts of the system to react to changes without tight coupling.

For example, when a page is updated, the page module emits a `page-updated` event. Other modules listen for this event:
- The search module listens to update the search index.
- The storage module listens to trigger a Git commit or cloud storage push.
- The cache module listens to invalidate the page render cache.

This event-driven approach makes the codebase highly extensible and maintainable.

## 11. Database Migration System

As Wiki.js evolves, the database schema must be updated to support new features. Wiki.js uses a robust database migration system based on versioned JavaScript files.

When Wiki.js starts, it checks the current database schema version against the available migration files. If the database is outdated, it sequentially executes the necessary migration scripts. These scripts use Knex.js schema building methods to create tables, add columns, and modify indexes. This ensures that upgrades are smooth and data integrity is maintained across versions.

## 12. Module Loading System

Wiki.js features a dynamic module loading system. During startup, the system scans specific directories (e.g., `/server/modules`) to discover available modules, editors, rendering extensions, and authentication strategies.

This dynamic discovery mechanism allows developers to easily add new features by simply dropping a new module folder into the appropriate directory. The module loading system reads the module's configuration (usually a `module.yml` or `package.json` file) and registers its capabilities with the core system.

## 13. Vue.js Frontend Architecture

The frontend of Wiki.js is a modern Single Page Application (SPA) built with Vue.js 2.

### Vuetify Material Design
The user interface is constructed using Vuetify, a comprehensive UI component framework based on Google's Material Design specification. Vuetify provides a consistent, responsive, and accessible design language across the entire application.

### Apollo GraphQL Client
To interact with the backend GraphQL API, the frontend uses the Apollo Client. Apollo manages data fetching, caching, and state management. It seamlessly integrates with Vue.js, allowing components to declaratively request the data they need.

### Editor Integration
The frontend integrates powerful third-party editors. For Markdown, it uses CodeMirror, providing syntax highlighting, line numbers, and a smooth typing experience. For WYSIWYG editing, it integrates CKEditor, offering a rich text editing interface familiar to users of traditional word processors.

## 14. Known Limitations and Workarounds

While Wiki.js is a powerful platform, it has several known limitations that administrators must be aware of:

1.  **Subfolder Installation:** Wiki.js cannot be installed in a subfolder (e.g., `example.com/wiki`). It must be hosted on a dedicated subdomain or root domain (e.g., `wiki.example.com`).
2.  **Real-time Collaboration:** There is no native support for real-time collaborative editing (like Google Docs). If two users edit the same page simultaneously, the last save wins.
3.  **SQLite in HA Mode:** SQLite cannot be used in High Availability (HA) mode due to its file-based nature and lack of concurrent write support. PostgreSQL is strictly required for HA.
4.  **MySQL Authentication:** MySQL 8's default `caching_sha2_password` authentication plugin is not supported. Users must configure their MySQL server to use `mysql_native_password`.
5.  **Editor Conversion:** Converting content between different editors (e.g., from Markdown to WYSIWYG) may result in formatting loss, as the underlying syntax differs significantly.
6.  **Custom HTML Stripping:** As a security measure, custom HTML is stripped by default by the security sanitizer. Administrators must explicitly configure the sanitizer to allow specific tags.
7.  **Rate Limiting:** There is no native API rate limiting. To protect against brute-force attacks or API abuse, administrators must implement rate limiting at the reverse proxy level (e.g., using Nginx or HAProxy).
8.  **Git Sync Conflicts:** The bidirectional Git storage sync can encounter conflicts if direct database edits and external Git commits modify the same page simultaneously. Manual intervention may be required to resolve these conflicts.
9.  **Page Path Restrictions:** Page paths cannot contain dots, spaces, or special characters. This can be restrictive for users migrating from other wiki platforms with more lenient naming conventions.
10. **CloudFlare Optimizations:** CloudFlare's optimization features (Auto Minify, Mirage, Rocket Loader) can break Wiki.js assets. These features must be disabled for the Wiki.js domain.
11. **Docker Updates:** When running via Docker, the `wiki-update-companion` container must be updated separately from the main `wiki` container.
12. **Custom JS Execution:** Custom JavaScript injected via the administration panel must be wrapped in `window.boot.register('page-ready', callback)` to ensure it executes after the SPA has fully loaded the page content.
13. **Privileged Ports:** Binding Wiki.js to ports below 1024 (like 80 or 443) requires special privileges. Administrators must use `setcap` on the Node.js binary or configure `iptables` port redirection.

## 15. Production Operations and Worst-Case Scenarios

Operating Wiki.js in a production environment requires careful planning and monitoring.

### High Availability (HA) Configuration
For mission-critical deployments, Wiki.js should be configured in HA mode. This requires setting `ha: true` in the `config.yml` file and using PostgreSQL as the database. In HA mode, multiple Wiki.js instances can run concurrently behind a load balancer. The instances use PostgreSQL's listen/notify features to synchronize cache invalidations and events across the cluster.

```yaml
# config.yml snippet for HA
port: 3000
db:
  type: postgres
  host: pg-cluster.internal
  port: 5432
  user: wiki
  pass: securepassword
  db: wikijs
ha: true
```

### Docker and Kubernetes Deployment
The recommended deployment method is using Docker. The official image is available at `ghcr.io/requarks/wiki`. For Kubernetes environments, an official Helm chart is available, simplifying the deployment and management of HA clusters.

```bash
# Example Docker run command
docker run -d -p 8080:3000 --name wiki --restart unless-stopped -e DB_TYPE=postgres -e DB_HOST=db -e DB_PORT=5432 -e DB_USER=wiki -e DB_PASS=wiki -e DB_NAME=wiki ghcr.io/requarks/wiki:2
```

### Worst-Case Scenarios and Disaster Recovery

**Scenario 1: Database Corruption**
If the primary database becomes corrupted, the system will fail.
*Recovery:* Restore the database from the latest automated backup. If Git storage sync is enabled, the raw Markdown content is safe in the Git repository, but user accounts, permissions, and history metadata will be lost if the database backup is unavailable.

**Scenario 2: Compromised Administrator Account**
If an admin account is compromised, the attacker could delete pages or modify system settings.
*Recovery:* Immediately revoke the compromised user's access via the database or identity provider. Use the version history feature to restore any modified or deleted pages. Review the audit logs (if configured via a reverse proxy) to determine the extent of the breach.

**Scenario 3: Storage Sync Failure**
If the Git repository becomes unreachable, Wiki.js will queue the sync events. However, if the queue overflows or the server restarts, sync events may be lost.
*Recovery:* Manually trigger a full sync from the administration panel once the Git repository is available again. Monitor the Wiki.js logs for sync errors.

**Scenario 4: Out of Memory (OOM) Crashes**
Large wikis with complex rendering tasks can consume significant memory, leading to Node.js OOM crashes.
*Recovery:* Increase the memory limit for the Node.js process using the `--max-old-space-size` flag. Optimize the rendering pipeline by disabling unnecessary extensions. Ensure the page render cache is functioning correctly.

By understanding the intricate architecture of Wiki.js, administrators can effectively deploy, manage, and troubleshoot this powerful wiki platform, ensuring a reliable and performant experience for all users.
