# Wiki.js Advanced Patterns and Operations Guide

Wiki.js is a powerful, open-source modern wiki platform built on Node.js, Vue.js, and GraphQL. As organizations scale their documentation needs, moving beyond basic page creation becomes essential. This comprehensive guide delves into the advanced patterns, configurations, and operational strategies required to manage a production-grade Wiki.js instance. From bidirectional Git synchronization and rendering pipeline customization to performance optimization and custom extension building, this document serves as the definitive resource for Wiki.js administrators and advanced users.

## 1. Git Storage Synchronization

One of the most powerful features of Wiki.js is its ability to synchronize content with external storage backends. Among the 11 supported storage backends, Git stands out for its version control capabilities, enabling a docs-as-code workflow.

### Bidirectional Sync

Wiki.js supports bidirectional synchronization with Git repositories. This means that changes made within the Wiki.js web interface are automatically committed and pushed to the Git repository, while changes pushed directly to the Git repository (e.g., via a developer's local IDE) are pulled and reflected in the wiki. This bidirectional flow ensures that technical writers and developers can collaborate seamlessly using their preferred tools.

To configure bidirectional sync, administrators must define the target repository URL, branch, and authentication credentials in the Wiki.js storage settings. The synchronization interval can be customized, or webhooks can be employed to trigger immediate pulls upon repository updates. It is crucial to ensure that the Git user configured in Wiki.js has both read and write permissions to the repository.

### Conflict Resolution

While bidirectional sync is powerful, it introduces the potential for conflicts, particularly when a page is edited simultaneously in the web interface and the Git repository. Wiki.js handles conflicts by prioritizing the database state. If a conflict occurs during a pull operation, the system will attempt to merge the changes. If an automatic merge is not possible, the sync operation may fail, requiring manual intervention.

To mitigate conflicts, it is recommended to establish clear workflows. For instance, designate specific sections of the wiki for web-based editing and others for Git-based editing. Additionally, leveraging the automatic snapshot feature in Wiki.js ensures that a version history is maintained for every update, allowing administrators to revert to a previous state if a conflict corrupts a page.

### SSH vs HTTPS Authentication

When configuring Git storage, administrators must choose between SSH and HTTPS authentication. 

**HTTPS Authentication:** This method requires a username and a personal access token (PAT) or password. It is generally easier to set up and is less prone to firewall restrictions. However, managing PATs can be cumbersome, especially when dealing with token expiration and rotation policies.

**SSH Authentication:** SSH relies on public/private key pairs, offering a more secure and robust authentication mechanism. To use SSH, administrators must generate an SSH key pair on the server hosting Wiki.js and add the public key to the Git provider (e.g., GitHub, GitLab). This method eliminates the need for passwords and tokens, making it ideal for automated, long-term synchronization. Ensure that the private key is securely stored and accessible only by the Wiki.js process.

## 2. Multi-Language and Locale Support

In a globalized environment, documentation must cater to diverse audiences. Wiki.js provides robust multi-language and locale support, allowing administrators to create and manage content in multiple languages seamlessly.

### Locale Configuration

Wiki.js supports a wide array of locales. Administrators can define the default locale for the wiki and enable additional locales as needed. Each page in Wiki.js is associated with a specific locale, and the system uses the `locale` parameter in conjunction with the `path` and `privateNS` to generate a unique SHA1 hash for the page.

### Page Migration and Translation

When expanding documentation to new languages, Wiki.js facilitates page migration and translation. Users can create a new version of an existing page in a different locale. The system maintains a link between the different language versions of a page, allowing users to switch seamlessly between translations using the language selector in the user interface.

It is important to note that Wiki.js does not automatically translate content. Administrators must rely on manual translation or integrate third-party translation services. When migrating pages, ensure that the path structure remains consistent across locales to maintain a logical hierarchy and facilitate easy navigation.

## 3. Custom CSS and JS Injection

To align the wiki with corporate branding or to introduce custom interactivity, Wiki.js allows the injection of custom Cascading Style Sheets (CSS) and JavaScript (JS).

### Global vs Per-Page Injection

Administrators can inject CSS and JS globally, affecting the entire wiki, or on a per-page basis. Global injection is configured in the administration panel under the "Theme" settings. This is ideal for applying brand colors, typography, and global navigation modifications.

Per-page injection provides granular control, allowing specific styles or scripts to be applied only to individual pages. This is particularly useful for creating interactive dashboards, custom calculators, or unique layouts for specific documentation sections.

### Lifecycle Hooks and Security

When injecting custom JS, it is crucial to understand the Wiki.js lifecycle. Because Wiki.js is a single-page application (SPA) built with Vue.js, standard DOM load events may not fire as expected when navigating between pages. To ensure that custom scripts execute correctly, developers must use the provided lifecycle hooks. Specifically, custom JS must use `window.boot.register('page-ready', callback)` to execute code after a page has fully rendered.

Security is a paramount concern when allowing custom JS injection. By default, Wiki.js employs a security sanitizer that strips custom HTML and potentially malicious scripts. Administrators must carefully manage permissions, granting the `write:styles` and `write:scripts` permissions only to trusted users.

## 4. Rendering Pipeline Customization

Wiki.js features a highly customizable rendering pipeline, supporting 14 Markdown rendering extensions and 11 HTML post-processing extensions. This flexibility allows administrators to tailor the rendering process to their specific needs.

### Enabling and Disabling Extensions

Administrators can enable or disable specific extensions in the administration panel. For instance, if the wiki does not require mathematical rendering, the KaTeX and MathJax extensions can be disabled to improve rendering performance. Conversely, enabling extensions like `emoji`, `tasklists`, and `footnotes` can significantly enhance the richness of the documentation.

### Mathematical Rendering: KaTeX vs MathJax

For technical documentation requiring mathematical formulas, Wiki.js offers two primary rendering engines: KaTeX and MathJax.

**KaTeX:** Known for its exceptional rendering speed, KaTeX is the preferred choice for wikis with extensive mathematical content. It renders formulas synchronously and does not require heavy client-side processing, resulting in faster page load times.

**MathJax:** While slightly slower than KaTeX, MathJax offers broader compatibility and supports a wider range of LaTeX macros and extensions. It is highly configurable and provides excellent accessibility features. Administrators should choose the engine that best balances performance and feature requirements.

### Diagramming: Mermaid, PlantUML, and Kroki

Visualizing complex architectures and workflows is essential in technical documentation. Wiki.js supports several diagramming tools:

**Mermaid:** A JavaScript-based diagramming and charting tool that renders Markdown-inspired text definitions dynamically in the browser. It is lightweight and ideal for flowcharts, sequence diagrams, and Gantt charts.

**PlantUML:** A versatile tool that generates UML diagrams from a specialized text language. PlantUML requires a backend server to render the diagrams into images. Wiki.js can be configured to point to a public or self-hosted PlantUML server.

**Kroki:** A unified API that provides access to multiple diagramming libraries, including BlockDiag, GraphViz, and Nomnoml. By integrating Kroki, Wiki.js users can leverage a vast array of diagramming tools through a single interface.

## 5. Advanced Content Components

Beyond standard text and images, Wiki.js supports advanced content components that enhance the interactivity and organization of documentation.

### Tabsets

Tabsets allow authors to group related content into selectable tabs, reducing page clutter and improving readability. This is particularly useful for providing code examples in multiple programming languages or presenting step-by-step instructions for different operating systems. The `tabset` HTML post-processing extension must be enabled to utilize this feature.

### Pivot Tables

For data-heavy documentation, the `pivot-table` Markdown extension enables the creation of interactive pivot tables directly within the wiki. Authors can define the data source and configuration parameters, allowing users to dynamically aggregate, filter, and analyze data without leaving the page.

### Task Lists

The `tasklists` extension transforms standard Markdown lists into interactive checklists. This feature is invaluable for creating onboarding guides, deployment checklists, and project tracking pages. Users can check and uncheck items, providing a visual indication of progress.

## 6. Building Custom Extensions

While Wiki.js offers a rich set of built-in extensions, organizations may require custom functionality. Wiki.js supports the integration of external tools and the development of custom extensions.

### Pandoc Integration

Pandoc is a universal document converter that can transform files between numerous formats (e.g., Markdown, HTML, Word, PDF). By integrating Pandoc, Wiki.js can support advanced document import and export capabilities. Administrators must install Pandoc on the host server and configure the Wiki.js Pandoc extension to define the conversion parameters and supported formats.

### Puppeteer and Sharp

For advanced rendering and image processing, Wiki.js can leverage Puppeteer and Sharp.

**Puppeteer:** A Node library that provides a high-level API to control headless Chrome or Chromium. In the context of Wiki.js, Puppeteer can be used to generate high-quality PDF exports of wiki pages, ensuring that the layout and styling are preserved perfectly.

**Sharp:** A high-performance Node.js image processing library. Integrating Sharp allows Wiki.js to automatically resize, compress, and optimize uploaded images, significantly improving page load times and reducing storage consumption.

## 7. CleanCSS Minification

To optimize the delivery of CSS assets, Wiki.js utilizes CleanCSS for minification. CleanCSS is a fast and efficient CSS optimizer that removes whitespace, comments, and redundant rules, resulting in smaller file sizes and faster download times.

Administrators can configure the CleanCSS integration to define the level of optimization and compatibility settings. It is important to note that aggressive minification can sometimes break complex CSS rules. Therefore, it is recommended to test custom styles thoroughly after enabling minification. Additionally, when using external optimization services like Cloudflare, administrators must disable features like Auto Minify, Mirage, and Rocket Loader, as they can conflict with Wiki.js's internal asset management and break the rendering pipeline.

## 8. Page Tree Management and Tag System

Organizing content effectively is crucial for maintaining a usable wiki. Wiki.js provides robust tools for structuring and categorizing pages.

### Page Tree Management

Wiki.js employs a hierarchical page tree structure, allowing administrators to organize content into logical folders and subfolders. The page path dictates its position in the tree. It is important to adhere to the path rules: no dots, spaces, backslashes, or double slashes, and no starting or ending slashes.

Administrators can manage the page tree through the administration panel, moving, renaming, and deleting pages as needed. When a page is moved or renamed, Wiki.js automatically updates internal links to prevent broken references.

### Tag System

In addition to the hierarchical structure, Wiki.js features a flexible tag system. Authors can assign multiple tags to a page, enabling cross-categorization and faceted search. Tags are particularly useful for grouping related content that spans different sections of the page tree, such as "troubleshooting," "api," or "deprecated."

Administrators can manage the global tag vocabulary, merging duplicate tags and deleting obsolete ones to maintain a clean and consistent taxonomy.

## 9. Scheduled Publishing

For organizations that require strict control over content release cycles, Wiki.js supports scheduled publishing. Authors can define a `publishStartDate` and a `publishEndDate` in the page metadata.

**Publish Start Date:** The page will remain hidden from standard users until the specified date and time. This is ideal for preparing release notes, announcements, or embargoed documentation in advance.

**Publish End Date:** The page will automatically become inaccessible to standard users after the specified date and time. This is useful for time-sensitive content, such as promotional offers or temporary event information.

Administrators and users with the `manage:pages` permission can view and edit scheduled pages regardless of the current date.

## 10. Private Pages and Namespaces

Security and access control are paramount in enterprise environments. Wiki.js provides granular permissions to restrict access to sensitive information.

### Private Pages

Authors can designate individual pages as private, restricting access to specific users or groups. This is useful for drafting content, storing internal notes, or sharing confidential information with a limited audience.

### Namespaces

For broader access control, administrators can utilize namespaces. A namespace is a distinct section of the wiki with its own set of permissions and configurations. By assigning pages to a private namespace, administrators can ensure that only authorized users can view, edit, or manage the content within that namespace. The page hash generation incorporates the `privateNS` parameter, ensuring cryptographic separation between public and private content.

## 11. Asset Management

Effective asset management is essential for maintaining a performant and organized wiki. Wiki.js provides a centralized asset manager for uploading, organizing, and inserting images, documents, and other files.

Administrators can define the maximum file size for uploads and restrict the allowed file types to prevent the distribution of malicious content. The asset manager supports folder structures, allowing users to categorize assets logically.

When integrating with external storage backends like S3 or Azure Blob Storage, Wiki.js can offload asset storage, reducing the burden on the local server and leveraging the scalability and durability of cloud storage providers.

## 12. Performance Optimization

As a Wiki.js instance grows in content and user base, performance optimization becomes critical. Administrators must implement strategies to ensure fast response times and high availability.

### Connection Pooling

Wiki.js relies heavily on its database backend (PostgreSQL is recommended and required for High Availability). To optimize database interactions, administrators must configure connection pooling in the `config.yml` file. Connection pooling maintains a cache of database connections, reducing the overhead of establishing new connections for every request. Properly tuning the pool size based on the server's resources and expected traffic is essential for preventing database bottlenecks.

### Caching Strategies

Caching is a highly effective technique for improving read performance. Wiki.js supports various caching mechanisms to reduce database load and accelerate page rendering.

**In-Memory Caching:** Wiki.js utilizes in-memory caching for frequently accessed data, such as configuration settings and user permissions.

**Page Caching:** For static content, administrators can implement page caching using a reverse proxy like Nginx or Redis. This allows the server to serve pre-rendered HTML pages directly from the cache, bypassing the Node.js application entirely.

### CDN Integration

To optimize the delivery of static assets (CSS, JS, images) to a global audience, administrators should integrate a Content Delivery Network (CDN). A CDN caches assets on edge servers distributed worldwide, reducing latency and improving load times for users geographically distant from the origin server.

When configuring a CDN, ensure that the Wiki.js `bindIP` and `port` settings are correctly configured to handle forwarded requests. As mentioned earlier, disable aggressive optimization features on the CDN (e.g., Cloudflare's Auto Minify) to prevent conflicts with Wiki.js's internal asset management.

## 13. Known Limitations and Workarounds

While Wiki.js is a robust platform, it has certain limitations that administrators must be aware of. Understanding these limitations and implementing appropriate workarounds is crucial for maintaining a stable production environment.

1. **Subfolder Installation:** Wiki.js cannot be installed in a subfolder (e.g., `example.com/wiki`). It must be hosted on a dedicated subdomain or root domain (e.g., `wiki.example.com`).
2. **Real-Time Collaborative Editing:** Wiki.js does not currently support real-time collaborative editing (like Google Docs). Users must coordinate edits to avoid conflicts, or rely on Git storage synchronization for version control.
3. **High Availability (HA) Mode:** HA mode requires PostgreSQL. SQLite cannot be used in HA mode due to its file-based nature and lack of concurrent write support. To enable HA, set `ha: true` in `config.yml`.
4. **MySQL Authentication:** MySQL `caching_sha2_password` is not supported. Administrators must configure MySQL to use `mysql_native_password` for the Wiki.js database user.
5. **Editor Conversion:** Converting content between different editors (e.g., from Markdown to WYSIWYG) may result in formatting loss. It is recommended to choose an editor and stick with it for a given page.
6. **Custom HTML Stripping:** By default, the security sanitizer strips custom HTML. To use custom HTML, administrators must disable the sanitizer or carefully configure allowed tags, balancing functionality with security risks.
7. **API Rate Limiting:** Wiki.js does not have native API rate limiting. To protect the GraphQL endpoint from abuse, administrators must implement rate limiting at the reverse proxy level (e.g., using Nginx `limit_req`).
8. **Git Sync Conflicts:** As discussed, Git storage sync can conflict with direct database edits. Establish clear workflows to minimize concurrent modifications.
9. **Page Path Restrictions:** Page paths cannot contain dots, spaces, or special characters. Use hyphens for separation and adhere strictly to the path rules.
10. **Cloudflare Optimizations:** Cloudflare's Auto Minify, Mirage, and Rocket Loader break Wiki.js assets. These features must be disabled for the Wiki.js domain.
11. **Docker Updates:** When deploying via Docker, the `wiki-update-companion` container must be updated separately from the main `wiki` container to ensure smooth version upgrades.
12. **Custom JS Execution:** Custom JS must use `window.boot.register('page-ready', callback)` to execute reliably within the SPA architecture.
13. **Port Binding:** Binding Wiki.js to a port lower than 1024 (e.g., 80 or 443) requires special privileges. Use `setcap` to grant the Node.js binary permission, or use `iptables` to redirect traffic from a higher port.

## 14. Conclusion

Managing a production-grade Wiki.js instance requires a deep understanding of its architecture, configuration options, and operational nuances. By leveraging advanced patterns such as Git synchronization, rendering pipeline customization, and robust performance optimization strategies, administrators can build a highly scalable, secure, and performant documentation platform. Adhering to best practices and understanding the platform's limitations ensures a seamless experience for both content creators and consumers, solidifying Wiki.js as a premier choice for modern knowledge management.
