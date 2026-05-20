# Wiki.js Troubleshooting and Operations Guide

Welcome to the definitive troubleshooting and operations guide for Wiki.js (v2.x). Wiki.js is a powerful, open-source (AGPL-3.0) wiki platform built on Node.js (>= 20), Vue.js 2, Vuetify, and a robust GraphQL API. While it offers extensive features including 6 editors, 14 Markdown rendering extensions, 11 HTML post-processing extensions, 21 authentication modules, 11 storage backends, and 8 search engines, operating it in production can present unique challenges.

This guide covers the most common and complex issues encountered by system administrators, DevOps engineers, and technical support teams when deploying and maintaining Wiki.js.

---

## 1. Installation and Deployment Limitations

### 1.1 Cannot Install in Subfolder (Subdomain Only)
**Symptom:** Attempting to serve Wiki.js from a subfolder (e.g., `https://example.com/wiki/`) results in broken assets, routing failures, and API errors.
**Root Cause:** Wiki.js v2.x is strictly designed to run at the root of a domain or subdomain. The Vue.js frontend router and GraphQL API endpoints are hardcoded to expect the root path `/`.
**Resolution:** 
You must deploy Wiki.js on a dedicated subdomain (e.g., `https://wiki.example.com`). If you absolutely must serve it under a main domain, you can use a reverse proxy to handle the subdomain routing, but the application itself must believe it is at the root.
*Note: Subfolder support is a known limitation and is not supported in the 2.x branch.*

### 1.2 Port Binding Issues (Ports < 1024)
**Symptom:** Wiki.js fails to start when configured to run on port 80 or 443, throwing an `EACCES: permission denied` error.
**Root Cause:** By default, Linux prevents non-root users from binding to privileged ports (ports below 1024). Running Node.js as root is a severe security risk.
**Resolution:**
There are three recommended ways to resolve this:
1. **Reverse Proxy (Recommended):** Run Wiki.js on a high port (e.g., 3000) and use Nginx, Apache, Caddy, or Traefik to proxy traffic from port 80/443 to port 3000.
2. **Setcap:** Grant the Node.js binary permission to bind to privileged ports:
   ```bash
   sudo setcap cap_net_bind_service=+ep /usr/bin/node
   ```
3. **iptables Redirect:** Forward traffic from port 80 to 3000 at the firewall level:
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3000
   ```

---

## 2. Reverse Proxy and CDN Issues

### 2.1 Cannot Upload Large Files
**Symptom:** Uploading files larger than a few megabytes fails with a `413 Payload Too Large` error or simply times out.
**Root Cause:** This is almost always caused by the reverse proxy (Nginx, Apache, etc.) or the `bodyParserLimit` in Wiki.js `config.yml`.
**Resolution:**
1. **Nginx:** Increase the `client_max_body_size` in your Nginx configuration block:
   ```nginx
   server {
       client_max_body_size 100M;
       # ... other config
   }
   ```
2. **Wiki.js Config:** Ensure the `bodyParserLimit` in `config.yml` is set high enough (default is usually sufficient, but check if modified):
   ```yaml
   bodyParserLimit: 100mb
   ```
3. **Cloudflare:** If using Cloudflare, note that the Free plan limits uploads to 100MB per request.

### 2.2 Cloudflare Breaking Assets
**Symptom:** The Wiki.js interface looks broken, icons are missing, scripts fail to load, or the page remains blank.
**Root Cause:** Cloudflare's aggressive optimization features interfere with Vue.js and Vuetify's dynamic asset loading and script execution.
**Resolution:**
You must disable specific Cloudflare optimizations for your Wiki.js domain. In the Cloudflare dashboard, navigate to your domain and disable:
- **Auto Minify:** Disable for JavaScript, CSS, and HTML. Wiki.js already minifies its assets.
- **Rocket Loader:** This alters the execution order of scripts and completely breaks Vue.js initialization. **Must be OFF.**
- **Mirage:** Disable image optimization that alters DOM loading.
Create a Page Rule in Cloudflare for `*wiki.yourdomain.com/*` to explicitly disable these features.

### 2.3 SSL Redirect Loops
**Symptom:** Accessing the wiki results in an `ERR_TOO_MANY_REDIRECTS` error in the browser.
**Root Cause:** The reverse proxy is handling SSL (HTTPS) and forwarding the request to Wiki.js via HTTP, but Wiki.js is configured to force HTTPS or the Site URL in the database is mismatched.
**Resolution:**
1. Ensure your reverse proxy passes the correct headers:
   ```nginx
   proxy_set_header X-Forwarded-Proto $scheme;
   ```
2. If you cannot access the admin panel to fix the Site URL, you must fix it directly in the database. Connect to your database (e.g., PostgreSQL) and run:
   ```sql
   UPDATE settings SET value = '{"host":"https://wiki.example.com"}' WHERE key = 'host';
   ```
   Restart the Wiki.js process after modifying the database.

---

## 3. Database and High Availability (HA)

### 3.1 MySQL caching_sha2_password Error
**Symptom:** Wiki.js fails to connect to a MySQL 8+ database, logging `ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MySQL client`.
**Root Cause:** MySQL 8 defaults to the `caching_sha2_password` authentication plugin, which the underlying Node.js MySQL driver used by Wiki.js does not fully support in all environments.
**Resolution:**
You must change the authentication plugin for the Wiki.js database user to `mysql_native_password`.
Log into MySQL as root and execute:
```sql
ALTER USER 'wiki'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_strong_password';
FLUSH PRIVILEGES;
```

### 3.2 High Availability (HA) Mode Cache Issues
**Symptom:** In a multi-node deployment, users see stale content, or updates made on one node do not reflect on others.
**Root Cause:** HA mode is not properly configured, or an unsupported database is being used.
**Resolution:**
1. **Database Requirement:** HA mode **requires** PostgreSQL. SQLite, MySQL, MariaDB, and MSSQL cannot be used for HA because Wiki.js relies on PostgreSQL's `LISTEN/NOTIFY` pub/sub mechanism to invalidate caches across nodes.
2. **Configuration:** Ensure `ha: true` is set in the `config.yml` of **every** node.
   ```yaml
   ha: true
   ```
3. **Clock Sync:** Ensure all nodes have their system clocks synchronized via NTP.

---

## 4. Content and Editor Troubleshooting

### 4.1 HTML Sanitization Stripping Content
**Symptom:** Custom HTML tags, iframes, or inline styles are removed when saving a page using the WYSIWYG or Code editors.
**Root Cause:** Wiki.js includes a strict HTML security sanitizer (one of its 11 HTML post-processing extensions) enabled by default to prevent XSS attacks.
**Resolution:**
If you trust your authors and need custom HTML:
1. Go to **Administration > Rendering > HTML**.
2. Locate the **Security** tab.
3. You can either disable the sanitizer entirely (not recommended for public wikis) or configure the allowed tags and attributes.
4. To allow iframes (e.g., for YouTube embeds), add `iframe` to the allowed tags and `src`, `width`, `height`, `allowfullscreen` to the allowed attributes.

### 4.2 Custom JS Not Executing
**Symptom:** JavaScript added via the Code Injection settings or within pages does not run when navigating between pages.
**Root Cause:** Wiki.js is a Single Page Application (SPA). Standard `window.onload` or `document.addEventListener('DOMContentLoaded', ...)` events only fire once on the initial page load, not on subsequent SPA navigations.
**Resolution:**
You must use the Wiki.js specific boot registration pattern. Wrap your custom JavaScript as follows:
```javascript
window.boot.register('page-ready', function() {
    // Your custom code here
    console.log('Page has fully loaded and rendered!');
});
```

### 4.3 Editor Conversion Data Loss
**Symptom:** Switching a page from the Markdown editor to the WYSIWYG editor, or vice versa, results in lost formatting or broken content.
**Root Cause:** The 6 different editors (markdown, wysiwyg, ckeditor, code, api, asciidoc) store data differently. Converting between them is not a 1:1 lossless process. Markdown to HTML is one-way; HTML back to Markdown relies on heuristics and often fails on complex layouts.
**Resolution:**
- **Best Practice:** Choose one editor standard for your team and stick to it.
- **Recovery:** If data is lost during conversion, Wiki.js automatically creates a snapshot on every update. Go to the page's **History** tab and restore the previous version.

### 4.4 Page Creation Failures
**Symptom:** Users cannot create a new page, receiving an error about invalid paths or duplicate content.
**Root Cause:** Wiki.js enforces strict rules on page paths.
**Resolution:**
- **Path Rules:** Page paths cannot contain dots (`.`), spaces, backslashes (`\`), or double slashes (`//`). They cannot start or end with a slash.
- **Duplicate Path:** Ensure the path isn't already taken. The page hash is calculated as `SHA1(locale + path + privateNS)`.
- **Empty Content:** A page must have at least some content to be saved.

---

## 5. System and Integration Failures

### 5.1 Upgrade Button Not Working
**Symptom:** Clicking the "Upgrade" button in the Administration panel does nothing or throws an error, especially in Docker environments.
**Root Cause:** The main Wiki.js Node process cannot overwrite its own files while running, especially inside a containerized environment where the filesystem might be read-only or ephemeral.
**Resolution:**
For Docker deployments (`ghcr.io/requarks/wiki`), you must use the `wiki-update-companion` container.
1. Ensure the companion container is running and mapped to the same volume as the main wiki container.
2. The companion container handles the actual file replacement and restarts the main process.
3. Alternatively, update by pulling the latest Docker image and recreating the container:
   ```bash
   docker pull ghcr.io/requarks/wiki:2
   docker-compose up -d
   ```

### 5.2 Git Storage Sync Conflicts
**Symptom:** The Git storage backend stops syncing, or pages edited in the UI are overwritten by old versions from Git.
**Root Cause:** Git storage sync can conflict with direct database edits or simultaneous edits. If a merge conflict occurs, the Wiki.js Git extension (which uses simple git commands) may halt.
**Resolution:**
1. **Check Logs:** Look at the Wiki.js logs for Git merge conflict errors.
2. **Manual Resolution:** Access the server, navigate to the local Git repository used by Wiki.js, and manually resolve the conflict using standard Git commands (`git status`, `git merge --abort`, etc.).
3. **Force Sync:** In the Admin panel under Storage, you can force a push or pull, but this may overwrite data. Always backup the database first.

### 5.3 Search Engine Not Indexing
**Symptom:** New pages do not appear in search results.
**Root Cause:** The search index is out of sync, or the configured search engine (e.g., Elasticsearch, PostgreSQL pg_trgm) is unreachable.
**Resolution:**
1. Go to **Administration > Search Engine**.
2. Click **Rebuild Index**.
3. If using PostgreSQL (`pg_trgm`), ensure the extension is installed in the database:
   ```sql
   CREATE EXTENSION IF NOT EXISTS pg_trgm;
   ```
4. Check the GraphQL API logs for indexing errors.

### 5.4 Telegram/Email Notification Failures
**Symptom:** Users do not receive password reset emails or Telegram notifications.
**Root Cause:** Incorrect SMTP settings, blocked ports, or invalid Telegram bot tokens.
**Resolution:**
- **Email:** Verify SMTP host, port (usually 587 for TLS or 465 for SSL), and credentials. Ensure your hosting provider (e.g., DigitalOcean, AWS) does not block outbound port 25/587.
- **Telegram:** Ensure the Bot Token is correct and the bot has been started by the user. Check the Wiki.js logs for `TELEGRAM_API_ERROR`.

### 5.5 Session Problems and Slow Loading Times
**Symptom:** Users are frequently logged out, or the wiki takes a long time to load.
**Root Cause:** Session issues are often tied to reverse proxy misconfigurations (dropping cookies) or database latency. Slow loading is usually due to large assets, unoptimized databases, or lack of caching.
**Resolution:**
- **Sessions:** Ensure your reverse proxy passes the `X-Forwarded-For` and `X-Forwarded-Proto` headers. If using multiple nodes without HA mode, sessions will not be shared.
- **Performance:** 
  - Use PostgreSQL.
  - Enable Redis for caching if supported in your architecture.
  - Ensure Cloudflare is configured correctly (see section 2.2).
  - Check the `config.yml` `logLevel` (set to `info` or `warn` in production, not `debug`).

### 5.6 Menu Items Showing Keys Instead of Text
**Symptom:** The navigation menu or UI elements show translation keys like `locales.en.menu.home` instead of the actual text.
**Root Cause:** The locale files failed to load, or the configured locale is missing/corrupted.
**Resolution:**
1. Go to **Administration > Locale**.
2. Ensure the desired locale is downloaded and set as default.
3. Click the cloud icon to force an update of the locale files from the Requarks servers.
4. Restart the Wiki.js process to flush the locale cache.

---

## 6. Advanced Debugging via GraphQL

Wiki.js relies heavily on its GraphQL API endpoint at `/graphql`. When the UI fails, you can often diagnose issues by querying the API directly.

### 6.1 Authenticating API Requests
You must use a Bearer token for API requests. Generate an API token in the Administration panel under **API Access**.

```bash
curl -X POST   -H "Content-Type: application/json"   -H "Authorization: Bearer YOUR_API_TOKEN"   -d '{"query": "{ site { title } }"}'   https://wiki.example.com/graphql
```

### 6.2 Checking System Status
Query the system status to verify database connectivity and version info:

```graphql
query {
  system {
    version
    dbVersion
    nodejsVersion
  }
}
```

### 6.3 Verifying Permissions
If a user cannot access a page, verify their group permissions. Wiki.js uses granular permissions like `read:pages`, `write:pages`, `manage:pages`, `delete:pages`, `write:styles`, `write:scripts`, `read:source`, `read:history`, `manage:system`.

---

## 7. Deep Dive: Understanding Wiki.js Architecture

To effectively troubleshoot Wiki.js, it is crucial to understand its underlying architecture and how its various components interact.

### 7.1 The Node.js Backend
Wiki.js is built on Node.js (requiring version 20 or higher for the latest 2.x releases). The backend is responsible for handling API requests, managing database connections, processing authentication, and serving the initial HTML payload. It uses a modular architecture where features like authentication, storage, and search are implemented as plugins.

### 7.2 The Vue.js Frontend
The frontend is a Single Page Application (SPA) built with Vue.js 2 and the Vuetify UI framework. When a user navigates to a page, the frontend requests the page data via the GraphQL API and renders it dynamically. This SPA architecture is why custom JavaScript must be registered using `window.boot.register` rather than standard DOM events.

### 7.3 The GraphQL API
Almost all communication between the frontend and backend occurs via the GraphQL API. This API is also available for external integrations. Understanding how to query this API is essential for advanced troubleshooting, as it can reveal information not always visible in the UI.

### 7.4 Storage Backends
Wiki.js supports 11 different storage backends, including local disk, Git, S3, and various cloud providers. The storage system is designed to be pluggable, allowing administrators to choose the best option for their infrastructure. However, this flexibility can also introduce complexity, especially when dealing with synchronization issues or permission errors.

### 7.5 Search Engines
The platform supports 8 different search engines, ranging from the built-in database search to powerful external engines like Elasticsearch and Algolia. Choosing the right search engine depends on the size of the wiki and the required search capabilities. Troubleshooting search issues often involves verifying the connection to the search engine and ensuring the index is up to date.

### 7.6 Authentication Modules
With 21 supported authentication modules, Wiki.js can integrate with almost any identity provider. Configuring these modules can be complex, and troubleshooting often involves checking the configuration settings, verifying the identity provider's response, and examining the Wiki.js logs for authentication errors.

---

## 8. Best Practices for Production Deployments

To minimize issues and ensure a stable Wiki.js deployment, follow these best practices:

### 8.1 Use PostgreSQL
While Wiki.js supports multiple databases, PostgreSQL is the recommended choice for production deployments. It is required for High Availability (HA) mode and generally offers better performance and reliability than other options.

### 8.2 Implement a Robust Backup Strategy
Regularly backup both the database and the storage backend. Wiki.js's built-in snapshot feature is useful for recovering individual pages, but a comprehensive backup strategy is essential for disaster recovery.

### 8.3 Monitor System Performance
Monitor the performance of the Node.js process, the database, and the reverse proxy. Use tools like Prometheus and Grafana to track key metrics and set up alerts for potential issues.

### 8.4 Keep Wiki.js Updated
Regularly update Wiki.js to the latest stable version to benefit from bug fixes, security patches, and new features. If using Docker, ensure the `wiki-update-companion` container is configured correctly to facilitate seamless updates.

### 8.5 Secure the Deployment
Implement strong security measures, including HTTPS, firewall rules, and regular security audits. Review the HTML sanitization settings to ensure they align with your security requirements.

---

## 9. Conclusion

Operating Wiki.js in production requires a solid understanding of Node.js environments, reverse proxy configurations, and database management. By adhering to the architectural requirements (like using PostgreSQL for HA and avoiding subfolder deployments) and understanding the nuances of its SPA architecture and security sanitizers, you can maintain a stable and highly performant knowledge base. Always remember to leverage the built-in snapshot version history and keep regular database backups to prevent data loss during editor conversions or storage sync conflicts.
