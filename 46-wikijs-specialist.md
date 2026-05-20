# Wiki.js Comprehensive Specialist Guide (Specialist #46)

## 1. Introduction and Architectural Overview

Wiki.js is a powerful, modern, and open-source wiki platform built on a robust technology stack. As the current stable version (v2.x), it has been designed to offer a highly extensible and performant environment for documentation, knowledge management, and collaborative content creation. Authored by Nicolas Giard under the Requarks organization, Wiki.js is distributed under the AGPL-3.0 license, ensuring that it remains free and open for the community.

### 1.1 Core Technology Stack

The architecture of Wiki.js is built upon several key technologies that work in tandem to deliver a seamless user experience and a highly scalable backend:

- **Backend Framework**: The core of Wiki.js is powered by **Node.js** (requiring version 20 or higher). Node.js provides an asynchronous, event-driven runtime that is highly efficient for handling concurrent requests, making it ideal for a web-based wiki platform.
- **Frontend Framework**: The user interface is constructed using **Vue.js 2**, a progressive JavaScript framework. Vue.js allows for reactive data binding and composable view components. The UI components are styled and structured using **Vuetify**, a Material Design component framework for Vue.js, which ensures a responsive and aesthetically pleasing design across all devices.
- **API Layer**: Communication between the frontend and backend is facilitated entirely through a **GraphQL API**. This endpoint, located at `/graphql`, allows the client to request exactly the data it needs, reducing over-fetching and improving performance. It utilizes Bearer token authentication for secure access.
- **Database ORM**: To interact with various database systems, Wiki.js employs **Objection.js**, an ORM (Object-Relational Mapping) tool built on top of Knex.js. Objection.js provides a powerful and flexible way to define models and relationships, abstracting the underlying SQL queries and enabling support for multiple database engines.

### 1.2 Architectural Principles

The architecture is designed with modularity and extensibility in mind. Features such as authentication, storage, search, and rendering are implemented as modular plugins or extensions. This allows administrators to tailor the platform to their specific needs without modifying the core codebase. The use of GraphQL further decouples the frontend from the backend, enabling third-party integrations and custom client applications to interact with the wiki data seamlessly.

---

## 2. Supported Database Systems

Wiki.js is highly versatile when it comes to data persistence, supporting a wide range of relational database management systems (RDBMS). The choice of database can significantly impact the performance, scalability, and high availability (HA) capabilities of the platform.

### 2.1 PostgreSQL (Recommended)

**PostgreSQL** is the officially recommended database for Wiki.js. It offers the best performance, most robust feature set, and is the **only database supported for High Availability (HA) mode**. When deploying Wiki.js in an enterprise environment or a multi-node cluster, PostgreSQL is mandatory. Furthermore, PostgreSQL provides built-in search capabilities via the `pg_trgm` extension, which can be utilized as a search engine backend.

### 2.2 MySQL 8+ and MariaDB 10.2.7+

Wiki.js supports **MySQL version 8.0 and above**, as well as **MariaDB version 10.2.7 and above**. These are popular choices for many web applications and provide excellent performance for single-node deployments.

**Important Operational Note for MySQL 8+**: Wiki.js does not currently support the `caching_sha2_password` authentication plugin, which is the default in MySQL 8. You must configure the database user to use the `mysql_native_password` plugin. This can be done with the following SQL command:

```sql
ALTER USER 'wikijs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_secure_password';
FLUSH PRIVILEGES;
```

### 2.3 Microsoft SQL Server (MSSQL) 2012+

For organizations heavily invested in the Microsoft ecosystem, Wiki.js supports **MSSQL 2012 and newer**. This allows seamless integration into existing Windows Server environments and utilizes existing database administration expertise.

### 2.4 SQLite 3.9+

**SQLite** (version 3.9+) is supported for lightweight, single-node deployments, testing, or development environments. It requires no separate database server process, as the database is stored in a single file on disk.

**Critical Limitation**: SQLite **cannot** be used in High Availability (HA) mode. It is strictly for single-instance deployments. Attempting to use SQLite with multiple Wiki.js nodes will result in database corruption and inconsistent state.

---

## 3. Editors and Content Creation

Wiki.js provides a diverse set of editors to cater to different user preferences and content types. Each editor is optimized for a specific workflow.

### 3.1 The 6 Available Editors

1.  **Markdown (contentType: markdown)**: The default and most popular editor. It provides a split-pane view with raw Markdown on the left and a live preview on the right. It supports a wide array of rendering extensions.
2.  **Visual Editor / WYSIWYG (contentType: html)**: A rich text editor designed for users who prefer a Microsoft Word-like experience. It generates HTML directly.
3.  **CKEditor (contentType: html)**: An alternative, highly customizable rich text editor that also outputs HTML. It is often preferred for its advanced formatting options.
4.  **Code (contentType: html)**: A raw code editor with syntax highlighting, ideal for writing raw HTML, CSS, or JavaScript directly into a page.
5.  **API (contentType: yml)**: A specialized editor for documenting APIs, typically using YAML format to define endpoints, parameters, and responses.
6.  **AsciiDoc (contentType: adoc)**: An editor for AsciiDoc, a lightweight markup language similar to Markdown but with more advanced features for technical documentation.

### 3.2 Page Path Rules and Hashing

When creating pages, the URL path is subject to strict validation rules to ensure consistency and prevent routing conflicts:
-   **No dots (`.`)**: File extensions or dot-notation are not allowed in paths.
-   **No spaces**: Spaces must be replaced with hyphens (`-`) or underscores (`_`).
-   **No backslashes (`\`)**: Only forward slashes (`/`) are permitted for defining hierarchy.
-   **No double slashes (`//`)**: Paths must be normalized.
-   **No starting or ending slash**: Paths must be relative to the root, e.g., `engineering/architecture`, not `/engineering/architecture/`.
-   **No special characters**: Paths should generally stick to alphanumeric characters, hyphens, and underscores.

Internally, Wiki.js identifies pages using a unique hash. The **Page Hash** is calculated as the SHA1 checksum of the locale, path, and private namespace:
`Page Hash = SHA1(locale + path + privateNS)`

### 3.3 Editor Limitations

**Warning**: Converting a page between different editors (e.g., from Markdown to WYSIWYG and back) is highly discouraged. Because different editors use different underlying content types (Markdown vs. HTML), converting between them **will likely result in a loss of formatting**, broken layouts, or corrupted content. It is best practice to choose an editor upon page creation and stick with it.

---

## 4. Rendering and Post-Processing Extensions

Wiki.js boasts a powerful rendering pipeline that transforms raw input (like Markdown) into the final HTML presented to the user. This pipeline is highly extensible.

### 4.1 Markdown Rendering Extensions (14 Modules)

For the Markdown editor, Wiki.js supports 14 rendering extensions that add specialized syntax and features:
1.  **core**: The standard Markdown parser.
2.  **abbr**: Support for abbreviations.
3.  **emoji**: Renders standard emoji shortcodes (e.g., `:smile:`).
4.  **expandtabs**: Converts tabs to spaces for consistent alignment.
5.  **footnotes**: Support for Markdown footnotes.
6.  **imsize**: Allows specifying image dimensions in Markdown syntax.
7.  **katex**: Fast math typesetting using KaTeX.
8.  **kroki**: Renders diagrams from textual descriptions using the Kroki API.
9.  **mathjax**: Alternative math typesetting using MathJax.
10. **multi-table**: Advanced table formatting.
11. **pivot-table**: Creates interactive pivot tables from data.
12. **plantuml**: Renders UML diagrams using PlantUML.
13. **supsub**: Support for superscript and subscript.
14. **tasklists**: Renders GitHub-style task lists (checkboxes).

### 4.2 HTML Post-Processing Extensions (11 Modules)

After the initial rendering (or directly from HTML editors), the content passes through 11 HTML post-processors:
1.  **core**: Basic HTML cleanup and structuring.
2.  **asciinema**: Embeds Asciinema terminal recordings.
3.  **blockquotes**: Enhances blockquote styling.
4.  **codehighlighter**: Applies syntax highlighting to code blocks using Prism.js.
5.  **diagram (draw.io)**: Embeds interactive draw.io diagrams.
6.  **image-prefetch**: Optimizes image loading.
7.  **mediaplayers**: Embeds audio and video players.
8.  **mermaid**: Renders Mermaid.js diagrams directly in the browser.
9.  **security**: **Crucial module**. Sanitizes HTML to prevent XSS attacks.
10. **tabset**: Creates interactive tabbed content areas.
11. **twemoji**: Replaces native emojis with Twitter's Twemoji for cross-platform consistency.

**Security Note**: The `security` post-processor is active by default and will strip out custom HTML tags, inline styles, and scripts that are not explicitly whitelisted. If users complain that their custom HTML or iframe embeds are disappearing upon saving, this sanitizer is the cause. It can be configured in the administration panel, but disabling it entirely poses a significant security risk.

---

## 5. Authentication Modules

Enterprise environments require flexible authentication. Wiki.js supports an impressive array of **21 authentication modules**, allowing integration with virtually any identity provider (IdP).

The supported modules include:
1. Auth0
2. Azure AD
3. CAS (Central Authentication Service)
4. Discord
5. Dropbox
6. Facebook
7. Firebase
8. GitHub
9. GitLab
10. Google
11. Keycloak
12. LDAP / Active Directory
13. Local (Built-in username/password)
14. Microsoft
15. OAuth2 (Generic)
16. OIDC (OpenID Connect)
17. Okta
18. Rocket.chat
19. SAML 2.0
20. Slack
21. Twitch

This extensive list ensures that whether an organization uses a legacy LDAP server, a modern cloud IdP like Okta or Azure AD, or social logins for a public wiki, Wiki.js can accommodate the requirement.

---

## 6. Storage Backends

Data backup and synchronization are critical for disaster recovery. Wiki.js supports **11 storage backends** to automatically sync your content to external services.

The supported storage targets are:
1. Azure Blob Storage
2. Box
3. DigitalOcean Spaces
4. Disk (Local file system)
5. Dropbox
6. Google Drive
7. Git (Bidirectional sync)
8. OneDrive
9. S3 (Amazon Web Services)
10. S3 Generic (MinIO, Wasabi, etc.)
11. SFTP

### 6.1 The Git Storage Conflict Issue

The **Git** storage backend is particularly popular as it allows bidirectional synchronization. You can edit a page in Wiki.js, and it commits to Git. Conversely, you can push a Markdown file to the Git repository, and Wiki.js will ingest it.

**Operational Hazard**: Git storage sync can conflict with direct database edits. If a user edits a page via the Wiki.js UI at the exact same time a commit is pushed to the Git repository modifying the same page, a race condition occurs. Wiki.js attempts to handle this gracefully, but in worst-case scenarios, one set of changes may overwrite the other. It is recommended to establish clear workflows: either treat the Wiki UI as the single source of truth and Git as a read-only backup, or treat Git as the source of truth and disable UI editing for those specific paths.

---

## 7. Search Engines

A wiki is only as good as its search functionality. Wiki.js provides **8 search engine integrations** to scale from small personal wikis to massive enterprise knowledge bases.

1.  **Algolia**: A powerful, hosted search API.
2.  **AWS CloudSearch**: Amazon's managed search service.
3.  **Azure Search**: Microsoft's cloud search offering.
4.  **Database (built-in)**: Basic ILIKE/LIKE queries. Good for small wikis but scales poorly.
5.  **Elasticsearch**: The industry standard for enterprise search. Highly recommended for large deployments.
6.  **Manticore Search**: A fast, open-source search engine.
7.  **PostgreSQL (pg_trgm)**: Utilizes PostgreSQL's trigram matching. An excellent middle-ground that requires no extra infrastructure if you are already using PostgreSQL.
8.  **Solr**: Apache Solr, another robust enterprise search platform.
9.  **Sphinx**: A classic open-source full-text search server.

---

## 8. Page CRUD Operations via GraphQL API

Wiki.js is built API-first. Every action available in the UI can be performed programmatically via the GraphQL API located at `/graphql`.

### 8.1 Authentication

API requests require a Bearer token. You must generate an API key in the Administration -> API Access section and include it in the HTTP headers:
`Authorization: Bearer YOUR_API_KEY`

### 8.2 API Rate Limiting Limitation

**Critical Limitation**: Wiki.js has **no native API rate limiting**. If you expose the GraphQL API to the public internet or untrusted networks, a malicious actor could easily overwhelm the Node.js process with complex queries or brute-force attacks.
**Mitigation**: You **must** implement rate limiting at the reverse proxy layer (e.g., Nginx, HAProxy, Traefik, or Cloudflare) before traffic reaches the Wiki.js container.

### 8.3 GraphQL Examples

Here are practical examples of performing CRUD operations on pages.

**Create a Page (Mutation)**
```graphql
mutation {
  pages {
    create(
      content: "# Hello World\nThis is a test page."
      description: "A test page created via API"
      editor: "markdown"
      isPublished: true
      isPrivate: false
      locale: "en"
      path: "api-test/hello-world"
      tags: ["api", "test"]
      title: "Hello World API"
    ) {
      responseResult {
        succeeded
        errorCode
        slug
        message
      }
      page {
        id
        path
      }
    }
  }
}
```

**Read a Page (Query)**
```graphql
query {
  pages {
    single(id: 123) {
      id
      title
      path
      content
      createdAt
      updatedAt
      authorName
    }
  }
}
```

**Update a Page (Mutation)**
```graphql
mutation {
  pages {
    update(
      id: 123
      content: "# Updated Content\nThis page was updated."
      title: "Updated Title"
    ) {
      responseResult {
        succeeded
        message
      }
    }
  }
}
```

**Delete a Page (Mutation)**
```graphql
mutation {
  pages {
    delete(id: 123) {
      responseResult {
        succeeded
        message
      }
    }
  }
}
```

---

## 9. Version History System

Wiki.js features a robust version history system. It takes an **automatic snapshot on every update**. Whenever a user saves a page, the previous state is preserved in the database. Users can view the history of a page, compare differences between versions, and restore previous versions if necessary. This provides a safety net against accidental deletions or malicious edits.

---

## 10. Permissions Model

Security and access control are managed through a granular permissions model based on user groups. Permissions are divided into specific scopes.

Key permissions include:
-   **read:pages**: Allows viewing published pages.
-   **write:pages**: Allows creating new pages and editing existing ones.
-   **manage:pages**: Allows moving, renaming, and altering metadata of pages.
-   **delete:pages**: Allows permanently deleting pages.
-   **write:styles**: Allows injecting custom CSS into the wiki. (High risk, grant only to trusted admins).
-   **write:scripts**: Allows injecting custom JavaScript. (Highest risk, potential for XSS, grant only to trusted admins).
-   **read:source**: Allows viewing the raw source code (e.g., raw Markdown) of a page.
-   **read:history**: Allows viewing the version history of a page.
-   **manage:system**: Grants full access to the administration panel.

Administrators can assign these permissions globally or restrict them to specific page paths using Page Rules, allowing for complex organizational structures (e.g., the HR group can only write to `/hr/*`).

---

## 11. Deployment Options and Infrastructure

Wiki.js is designed to be deployed in modern containerized environments, though manual installation is possible.

### 11.1 Docker Deployment

The most common and supported method is via Docker. The official image is hosted on the GitHub Container Registry: `ghcr.io/requarks/wiki`.

A standard `docker-compose.yml` for a PostgreSQL-backed instance looks like this:

```yaml
version: "3"
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: ghcr.io/requarks/wiki:2
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    ports:
      - "3000:3000"
    depends_on:
      - db
    restart: unless-stopped

  wiki-update-companion:
    image: ghcr.io/requarks/wiki-update-companion:latest
    environment:
      - WIKI_CONTAINER_NAME=wiki
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped

volumes:
  db-data:
```

**Crucial Note on Updates**: The `wiki-update-companion` container is required if you want to use the built-in updater in the Wiki.js admin panel. It communicates with the Docker socket to pull the new image and restart the main container. However, **the `wiki-update-companion` container itself must be updated separately** (e.g., via Watchtower or manual `docker pull`).

### 11.2 Kubernetes Deployment

For enterprise scale, Wiki.js provides an **official Helm chart**. This allows for easy deployment, scaling, and management within a Kubernetes cluster. The Helm chart supports configuring ingress, persistent volumes, and connecting to external database clusters.

### 11.3 Deployment Limitations and Gotchas

1.  **Subfolder Installation**: Wiki.js **cannot be installed in a subfolder** (e.g., `https://example.com/wiki/`). It strictly requires its own subdomain or root domain (e.g., `https://wiki.example.com/`). This is a hard limitation of the Vue.js router configuration.
2.  **Privileged Ports**: By default, Node.js cannot bind to ports below 1024 (like 80 or 443) without root privileges. Running the container as root is a security risk. If you must bind directly to these ports without a reverse proxy, you must use `setcap` on the Node binary or configure `iptables` redirects. The best practice is to run Wiki.js on port 3000 and use a reverse proxy (Nginx/Traefik) to handle ports 80/443 and SSL termination.
3.  **Cloudflare Optimization Conflicts**: If you place Wiki.js behind Cloudflare, you **must disable** Auto Minify (JS/CSS/HTML), Mirage, and Rocket Loader. These Cloudflare optimizations aggressively modify the DOM and script loading order, which breaks the Vue.js hydration process, resulting in a blank page or broken assets.

---

## 12. Configuration and High Availability (HA)

The core configuration of Wiki.js is managed via a `config.yml` file. When using Docker, these values are typically overridden by environment variables.

### 12.1 The config.yml File

Key parameters include:
-   `port`: The port the Node.js server listens on (default 3000).
-   `db`: Database connection details (type, host, port, user, pass, db).
-   `ssl`: Configuration for built-in HTTPS (usually handled by a reverse proxy instead).
-   `pool`: Database connection pool settings (min/max connections).
-   `bindIP`: The IP address to bind to (default `0.0.0.0`).
-   `logLevel`: Logging verbosity (info, warn, error, debug).
-   `ha`: High Availability toggle.
-   `dataPath`: Location for local file storage.
-   `bodyParserLimit`: Maximum size for incoming requests (useful for large uploads).

### 12.2 High Availability (HA) Mode

To run multiple instances of Wiki.js behind a load balancer for redundancy and scale, you must enable HA mode.

**Requirements for HA:**
1.  **PostgreSQL is strictly required**. MySQL, MariaDB, MSSQL, and SQLite are not supported for HA.
2.  You must set `ha: true` in the `config.yml` (or `HA_ACTIVE=true` via environment variable) on all nodes.
3.  All nodes must connect to the same PostgreSQL database cluster.
4.  You must use an external search engine (like Elasticsearch or PostgreSQL pg_trgm). The built-in database search may behave inconsistently across nodes.
5.  Storage must be centralized (e.g., S3, Azure Blob) rather than local disk, so all nodes have access to the same assets.

When `ha: true` is set, Wiki.js utilizes PostgreSQL's LISTEN/NOTIFY pub/sub system to broadcast cache invalidations and configuration changes across all connected nodes instantly.

---

## 13. Advanced Troubleshooting and Customization

### 13.1 Custom JavaScript Execution

Administrators can inject custom JavaScript via the admin panel. However, because Wiki.js is a Single Page Application (SPA) built with Vue.js, standard `document.addEventListener('DOMContentLoaded', ...)` will not work as expected when navigating between pages.

**Rule**: Custom JS must hook into the Wiki.js lifecycle using the global boot object:
```javascript
window.boot.register('page-ready', function() {
    // Your custom code here. This runs every time a page finishes rendering.
    console.log("Page loaded and rendered!");
});
```

### 13.2 No Real-Time Collaborative Editing

**Known Limitation**: Wiki.js does **not** support real-time collaborative editing (like Google Docs or Etherpad). If two users edit the same page simultaneously, the last one to save will overwrite the other's changes. The version history will capture both, but manual merging is required.

---

## 14. Relation to Other Specialist Files

This file (Specialist #46) serves as the comprehensive guide to the Wiki.js platform. It relates to the other 6 specialist files in the repository in the following ways:

1.  **Specialist #40 (PostgreSQL Architecture)**: As PostgreSQL is the recommended database and strictly required for HA mode in Wiki.js, Specialist #40 provides the necessary deep dive into tuning, scaling, and managing the database backend that powers this wiki.
2.  **Specialist #41 (Node.js Performance Tuning)**: Wiki.js is a Node.js application. The principles of event loop monitoring, memory management, and garbage collection detailed in Specialist #41 are directly applicable to optimizing the Wiki.js backend process.
3.  **Specialist #42 (GraphQL API Security)**: Wiki.js relies entirely on a GraphQL API. Specialist #42 covers advanced techniques for securing GraphQL endpoints, which is critical for Wiki.js given its lack of native rate limiting and the potential for complex query abuse.
4.  **Specialist #43 (Vue.js SPA Deployment)**: The frontend of Wiki.js is a Vue.js Single Page Application. Specialist #43 provides context on how SPAs are built, hydrated, and served, explaining why Cloudflare optimizations break Wiki.js and why subfolder deployments are unsupported.
5.  **Specialist #44 (Docker & Helm Orchestration)**: This file outlines the deployment of Wiki.js via Docker and Kubernetes. Specialist #44 provides the foundational knowledge required to manage these containerized deployments, handle persistent volumes, and configure ingress controllers.
6.  **Specialist #45 (Enterprise Identity Providers)**: Wiki.js supports 21 authentication modules. Specialist #45 details the protocols (SAML, OIDC, OAuth2) and configurations required to integrate platforms like Wiki.js with enterprise IdPs such as Azure AD, Okta, and Keycloak.

By understanding the interplay between these technologies, an administrator can deploy, secure, and scale Wiki.js to meet the demands of any enterprise environment.
