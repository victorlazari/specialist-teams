# Frontend Menu Design CLI Command Reference

## 1. Introduction

The `frontend-menu-design` Command Line Interface (CLI) is a powerful, comprehensive, and highly extensible toolset designed specifically for modern web developers, UI/UX engineers, and frontend architects. It provides an extensive suite of commands to scaffold, generate, preview, analyze, test, and export highly interactive, accessible, and responsive menu systems for web applications. Whether you are building a simple navigation bar for a personal blog, a complex mega-menu for an e-commerce platform, or a context-sensitive radial menu for a web-based application, this CLI streamlines the entire lifecycle of menu development.

This document serves as the definitive, deep-dive reference for the `frontend-menu-design` CLI. It covers every command, flag, argument, configuration schema, and provides detailed, real-world examples of usage to ensure you can leverage the full potential of the toolset in your enterprise or personal projects. By mastering this CLI, teams can enforce design consistency, guarantee WCAG accessibility compliance, and significantly reduce the boilerplate code typically associated with complex navigation structures.

## 2. Installation and Environment Setup

Before utilizing the CLI, ensure that your development environment meets the necessary prerequisites. The CLI is built on Node.js and requires a modern runtime environment.

### Prerequisites

- **Node.js**: Version 18.x (LTS) or higher is strictly required.
- **Package Manager**: npm (v8+), yarn (v1.22+ or v3+), or pnpm (v7+).
- **Operating System**: Windows 10/11, macOS (Intel or Apple Silicon), or Linux.

### Global Installation

To install the CLI globally on your system, allowing you to run it from any directory, execute the following command:

```bash
npm install -g @frontend-menu-design/cli
```

Alternatively, using yarn:

```bash
yarn global add @frontend-menu-design/cli
```

### Local Installation (Recommended)

For project-specific installations, which is the recommended approach for maintaining consistent versions across development teams and CI/CD pipelines, install it as a development dependency:

```bash
npm install --save-dev @frontend-menu-design/cli
```

When installed locally, you can invoke the CLI using `npx` or by adding scripts to your `package.json`:

```bash
npx frontend-menu-design <command>
```

```json
{
  "scripts": {
    "menu:generate": "frontend-menu-design generate",
    "menu:preview": "frontend-menu-design preview"
  }
}
```

## 3. Global Flags and Options

The CLI supports several global flags that can be applied to any command to modify its behavior, output formatting, or execution context. These flags provide granular control over how the CLI interacts with your system.

- `--help`, `-h`: Displays detailed help information for the CLI or a specific command. Includes argument descriptions and examples.
- `--version`, `-v`: Outputs the current version of the CLI installed on your system.
- `--verbose`, `-V`: Enables verbose logging. This outputs detailed trace information of the internal processes, file system operations, and abstract syntax tree (AST) transformations, which is invaluable for debugging custom plugins or templates.
- `--quiet`, `-q`: Suppresses all non-error output. This is highly recommended when running the CLI in automated CI/CD pipelines to prevent log pollution.
- `--config <path>`, `-c <path>`: Specifies a custom path to the configuration file. By default, the CLI looks for `menu.config.json`, `menu.config.js`, or `menu.config.ts` in the current working directory.
- `--dry-run`: Simulates the execution of the command without making any actual changes to the file system. It outputs a list of files that *would* be created, modified, or deleted.
- `--cwd <directory>`: Changes the current working directory for the CLI execution. Useful for monorepo setups.
- `--no-color`: Disables ANSI color output in the terminal.

## 4. Comprehensive Command Reference

### 4.1 `init`

The `init` command initializes a new frontend menu design project or injects menu configuration into an existing project. It creates the necessary configuration files, directory structures, and installs required peer dependencies based on the selected framework.

#### Usage

```bash
frontend-menu-design init [options]
```

#### Options

- `--template <name>`, `-t <name>`: Specifies the starter template to use. Available templates include:
  - `basic`: A standard horizontal navigation bar.
  - `mega-menu`: A complex, multi-column dropdown menu suitable for e-commerce.
  - `sidebar`: A vertical collapsible sidebar menu.
  - `mobile-drawer`: an off-canvas slide-out menu optimized for touch devices.
  - `radial`: A circular, context-sensitive menu for web apps.
- `--framework <name>`, `-f <name>`: Targets a specific frontend framework for the generated code. Supported frameworks are `react`, `vue`, `angular`, `svelte`, `solid`, and `vanilla`. Default is `vanilla`.
- `--typescript`, `--ts`: Initializes the project with TypeScript support, generating `.ts` and `.tsx` files, and includes necessary interface definitions for menu items.
- `--force`: Overwrites existing configuration files if they are present in the directory without prompting for confirmation.
- `--skip-install`: Skips the automatic installation of npm dependencies after initialization.

#### Detailed Examples

Initialize a new React-based mega-menu project with TypeScript support, skipping dependency installation:

```bash
frontend-menu-design init --template mega-menu --framework react --typescript --skip-install
```

Initialize a basic vanilla JavaScript menu in a specific directory, overwriting any existing configuration:

```bash
frontend-menu-design init --template basic --framework vanilla --force --cwd ./packages/ui-library
```

### 4.2 `generate` (Alias: `g`)

The `generate` command is the workhorse of the CLI. It scaffolds specific menu components, styles, state management logic, and accessibility wrappers based on the provided arguments and configuration. It uses AST manipulation to safely inject imports and component declarations into existing files.

#### Usage

```bash
frontend-menu-design generate <type> <name> [options]
```

#### Arguments

- `<type>`: The type of menu component to generate. Valid types are:
  - `item`: A standard clickable link or button.
  - `dropdown`: A container that reveals child items on hover or click.
  - `group`: A logical grouping of items, often with a header.
  - `divider`: A visual separator between items.
  - `icon`: An SVG icon wrapper optimized for menu usage.
  - `trigger`: The button or element that opens a mobile or dropdown menu.
- `<name>`: The identifier or name of the component to be generated (e.g., `UserProfile`, `SettingsDropdown`).

#### Options

- `--path <directory>`, `-p <directory>`: Specifies the output directory for the generated files. Defaults to the `componentsDir` specified in the config file, or `src/components/menu`.
- `--style <format>`, `-s <format>`: Defines the styling format. Options include `css`, `scss`, `less`, `styled-components`, `emotion`, and `tailwind`.
- `--accessible`, `-a`: Automatically generates ARIA attributes (e.g., `aria-expanded`, `aria-haspopup`, `role="menuitem"`), keyboard navigation logic (Arrow keys, Home, End, Escape), and focus management for the component, ensuring strict WCAG 2.1 AA compliance.
- `--animated`: Includes default CSS transitions or JavaScript animation library integrations (e.g., Framer Motion for React, Vue Transition component) for state changes.
- `--store <type>`: Scaffolds state management integration. Options include `redux`, `zustand`, `pinia`, or `context`.

#### Detailed Examples

Generate a new dropdown component named `UserMenu` with Tailwind CSS styling, accessibility features, and Framer Motion animations:

```bash
frontend-menu-design generate dropdown UserMenu --style tailwind --accessible --animated
```

Generate a menu item named `DashboardLink` in a specific directory with SCSS styling, and simulate the output using dry-run:

```bash
frontend-menu-design generate item DashboardLink --path src/layout/sidebar --style scss --dry-run
```

### 4.3 `preview`

The `preview` command launches a high-performance local development server (powered by Vite) to visualize and interact with the designed menu in complete isolation. It supports hot module replacement (HMR), allowing you to see changes instantly without reloading the page.

#### Usage

```bash
frontend-menu-design preview [options]
```

#### Options

- `--port <number>`, `-p <number>`: Specifies the port on which the preview server will run. Default is `3000`.
- `--host <address>`: Specifies the host address to bind the server to. Default is `localhost`. Use `0.0.0.0` to expose the server to your local network.
- `--open`, `-o`: Automatically opens the preview in the default system web browser upon server start.
- `--theme <name>`: Applies a specific theme to the preview environment (e.g., `light`, `dark`, `high-contrast`).
- `--viewport <size>`: Starts the preview in a specific viewport size to test responsiveness. Options include `mobile`, `tablet`, `desktop`, or custom dimensions like `800x600`.

#### Detailed Examples

Start the preview server on port 8080, expose it to the network, and open it in the browser:

```bash
frontend-menu-design preview --port 8080 --host 0.0.0.0 --open
```

Preview the menu using the dark theme in a mobile viewport:

```bash
frontend-menu-design preview --theme dark --viewport mobile
```

### 4.4 `build`

The `build` command compiles, minifies, and bundles the menu components, styles, and assets for production deployment. It utilizes Rollup under the hood to optimize the output for performance, tree-shaking, and minimal bundle size.

#### Usage

```bash
frontend-menu-design build [options]
```

#### Options

- `--out-dir <directory>`, `-d <directory>`: Specifies the output directory for the compiled assets. Default is `dist`.
- `--target <environment>`: Defines the target module format for the build. Options include `esmodules` (ESM), `commonjs` (CJS), and `umd`. Default is `esmodules`.
- `--minify`: Forces the minification of HTML, CSS, and JavaScript files using Terser and cssnano. This is enabled by default in production builds.
- `--sourcemap`: Generates source maps for the compiled files, aiding in production debugging and error tracking.
- `--analyze`: Generates a visual bundle size analysis report (HTML file) after the build completes, helping identify large dependencies.
- `--extract-css`: Extracts all CSS into a separate `.css` file rather than injecting it via JavaScript.

#### Detailed Examples

Build the menu for CommonJS environments with source maps enabled and CSS extracted:

```bash
frontend-menu-design build --target commonjs --sourcemap --extract-css
```

Build the project into a custom directory and generate a bundle analysis report:

```bash
frontend-menu-design build --out-dir build/production --analyze
```

### 4.5 `analyze`

The `analyze` command performs a comprehensive static analysis of the menu configuration, generated code, and structural hierarchy. It checks for accessibility violations, performance bottlenecks, structural inconsistencies, and dead code.

#### Usage

```bash
frontend-menu-design analyze [options]
```

#### Options

- `--ruleset <name>`: Specifies the ruleset to use for analysis. Options include:
  - `strict`: Enforces all best practices, performance limits, and strict accessibility rules.
  - `recommended`: A balanced ruleset suitable for most projects.
  - `accessibility-only`: Focuses solely on WCAG compliance and ARIA attribute validation.
  - `performance-only`: Focuses on DOM depth, CSS complexity, and bundle size estimates.
- `--format <type>`: Defines the output format of the analysis report. Options include `text`, `json`, `html`, and `markdown`. Default is `text`.
- `--output <file>`, `-o <file>`: Writes the analysis report to the specified file instead of standard output.
- `--fail-on-warning`: Causes the command to exit with a non-zero status code (e.g., `exit 1`) if any warnings are detected. Essential for strict CI/CD pipelines.

#### Detailed Examples

Run a strict analysis and output the results as a detailed JSON file for further processing:

```bash
frontend-menu-design analyze --ruleset strict --format json --output reports/analysis-report.json
```

Run an accessibility-focused analysis and fail the CI build if any warnings are found:

```bash
frontend-menu-design analyze --ruleset accessibility-only --fail-on-warning
```

### 4.6 `export`

The `export` command allows you to export the menu configuration, routing structure, and metadata into various data formats. This is highly useful for integration with Content Management Systems (CMS), dynamic routing libraries (like React Router or Vue Router), or generating external documentation.

#### Usage

```bash
frontend-menu-design export <format> [options]
```

#### Arguments

- `<format>`: The target format for the export. Supported formats are `json`, `yaml`, `xml`, `csv`, and `ts-types` (generates TypeScript interfaces based on the menu structure).

#### Options

- `--output <file>`, `-o <file>`: Specifies the destination file for the exported data. If omitted, the output is printed to standard output.
- `--include-meta`: Includes metadata such as creation dates, author information, versioning, and custom attributes in the export.
- `--flatten`: Flattens nested menu structures (trees) into a single-level list, which is required for certain CSV exports or flat-file databases.

#### Detailed Examples

Export the menu structure to a YAML file including all metadata:

```bash
frontend-menu-design export yaml --output config/menu-structure.yml --include-meta
```

Export a flattened version of the menu to a CSV file for a marketing team to review:

```bash
frontend-menu-design export csv --output flat-menu.csv --flatten
```

Generate TypeScript types based on the current menu configuration:

```bash
frontend-menu-design export ts-types --output src/types/menu.d.ts
```

### 4.7 `theme`

The `theme` command manages the visual themes associated with the menu. It allows you to create, update, extract, and apply color palettes, typography settings, spacing variables, and animation curves.

#### Usage

```bash
frontend-menu-design theme <action> [options]
```

#### Arguments

- `<action>`: The theme action to perform. Valid actions are:
  - `create`: Scaffolds a new theme configuration file.
  - `extract`: Analyzes an existing CSS/SCSS file and extracts colors and fonts into a theme config.
  - `apply`: Applies a specific theme to the current project, updating CSS variables or styled-components themes.
  - `list`: Lists all available themes in the project.

#### Options

- `--name <string>`, `-n <string>`: The name of the theme to create or apply.
- `--source <file>`: The source file to extract theme variables from (used exclusively with the `extract` action).
- `--css-vars`: Generates native CSS custom properties (variables) for the theme instead of preprocessor variables (like SCSS `$variables`).
- `--base <theme>`: Inherits values from a base theme when creating a new one (e.g., `--base light`).

#### Detailed Examples

Create a new theme named `ocean-breeze` using CSS variables, inheriting from the default light theme:

```bash
frontend-menu-design theme create --name ocean-breeze --css-vars --base light
```

Extract theme variables from a legacy CSS file to modernize the styling approach:

```bash
frontend-menu-design theme extract --source src/styles/legacy-nav.css --name legacy-theme
```

### 4.8 `lint`

The `lint` command enforces coding standards, structural integrity, and best practices specifically tailored for menu components. It integrates with ESLint, Stylelint, and custom AST validators under the hood but applies rules specific to navigation structures (e.g., ensuring every dropdown has a valid trigger, checking for duplicate routing paths).

#### Usage

```bash
frontend-menu-design lint [options]
```

#### Options

- `--fix`: Automatically fixes fixable linting errors and formatting issues (e.g., indentation, missing semicolons, sorting CSS properties).
- `--ignore-pattern <pattern>`: Specifies file patterns or directories to ignore during the linting process.
- `--max-warnings <number>`: Sets a threshold for the maximum number of allowed warnings before the command fails and returns a non-zero exit code.

#### Detailed Examples

Lint the project and automatically fix formatting and minor structural issues:

```bash
frontend-menu-design lint --fix
```

Lint the project, ignoring the `vendor` and `legacy` directories, and fail if there are more than 10 warnings:

```bash
frontend-menu-design lint --ignore-pattern "vendor/**/*" --ignore-pattern "legacy/**/*" --max-warnings 10
```

### 4.9 `plugin`

The `plugin` command manages CLI extensions. The `frontend-menu-design` CLI is highly extensible, allowing developers to write custom generators, analyzers, or export formats.

#### Usage

```bash
frontend-menu-design plugin <action> <plugin-name>
```

#### Arguments

- `<action>`: Valid actions are `install`, `remove`, `list`, and `create` (scaffolds a new plugin boilerplate).
- `<plugin-name>`: The npm package name of the plugin, or the local path.

#### Detailed Examples

Install a community plugin for generating GraphQL schemas from the menu structure:

```bash
frontend-menu-design plugin install @frontend-menu-design/plugin-graphql
```

Scaffold a new custom plugin for internal company use:

```bash
frontend-menu-design plugin create my-company-menu-validator
```

## 5. Configuration Schema (`menu.config.js`)

While the CLI can be used with zero configuration, its true power is unlocked through the configuration file. By default, the CLI looks for `menu.config.js`, `menu.config.ts`, or `menu.config.json` in the root of your project.

### Comprehensive Configuration Example

```javascript
/** @type {import('@frontend-menu-design/cli').UserConfig} */
module.exports = {
  // 1. Project Metadata
  project: {
    name: 'Enterprise Portal Navigation',
    framework: 'react', // 'react' | 'vue' | 'angular' | 'svelte' | 'vanilla'
    typescript: true,
    routing: 'react-router-dom', // Integration with routing libraries
  },
  
  // 2. Generation Settings
  generate: {
    defaultStyle: 'styled-components',
    componentsDir: 'src/components/navigation',
    enforceAccessibility: true,
    // Custom templates directory
    templatesDir: './.menu-templates',
    // Naming conventions
    namingConvention: 'PascalCase', // 'PascalCase' | 'camelCase' | 'kebab-case'
  },
  
  // 3. Build & Compilation Settings
  build: {
    target: 'esmodules',
    minify: process.env.NODE_ENV === 'production',
    extractCss: true,
    cssModules: false,
    // Rollup specific configurations
    rollupOptions: {
      external: ['react', 'react-dom'],
    }
  },
  
  // 4. Theme Definitions
  themes: {
    defaultTheme: 'light',
    light: {
      colors: {
        primary: '#0056b3',
        background: '#ffffff',
        text: '#333333',
        hover: '#f8f9fa',
        border: '#e9ecef'
      },
      typography: {
        fontFamily: '"Inter", sans-serif',
        fontSize: '16px',
        fontWeight: '500'
      },
      spacing: {
        paddingX: '1rem',
        paddingY: '0.5rem',
        gap: '0.25rem'
      },
      animation: {
        duration: '200ms',
        easing: 'ease-in-out'
      }
    },
    dark: {
      // Inherits structure, overrides values
      colors: {
        primary: '#4dabf7',
        background: '#1a1a1a',
        text: '#f8f9fa',
        hover: '#2c2c2c',
        border: '#333333'
      }
    }
  },

  // 5. Analysis & Linting Rules
  analyze: {
    rules: {
      'max-depth': ['error', 3], // Prevent menus deeper than 3 levels
      'require-aria-label': 'error',
      'no-duplicate-links': 'warning'
    }
  },

  // 6. Plugin Configuration
  plugins: [
    // Array of installed plugins and their options
    ['@frontend-menu-design/plugin-search', { indexFields: ['title', 'description'] }]
  ]
};
```

## 6. Advanced Architecture and Hooks

For enterprise integrations, the CLI exposes a lifecycle hook system. You can hook into the generation or build process to execute custom scripts.

### Lifecycle Hooks

In your `menu.config.js`, you can define hooks:

```javascript
module.exports = {
  // ... other config
  hooks: {
    beforeGenerate: async (context) => {
      console.log(`Preparing to generate ${context.type}: ${context.name}`);
      // Fetch dynamic data from a CMS before generation
    },
    afterGenerate: async (context, files) => {
      // Run a custom formatter like Prettier on the generated files
      const { execSync } = require('child_process');
      execSync(`npx prettier --write ${files.join(' ')}`);
    },
    beforeBuild: () => {
      // Clean up old build directories
    },
    afterBuild: (stats) => {
      // Upload build stats to a monitoring service
    }
  }
};
```

## 7. Best Practices and Enterprise Patterns

### CI/CD Pipeline Integration

When integrating the `frontend-menu-design` CLI into Continuous Integration and Continuous Deployment (CI/CD) pipelines (e.g., GitHub Actions, GitLab CI, Jenkins), strict validation is crucial.

Example GitHub Actions workflow step for validating menu integrity:

```yaml
name: Menu Integrity Check
on: [push, pull_request]

jobs:
  validate-menu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - name: Lint Menu Components
        run: npx frontend-menu-design lint --max-warnings 0
      - name: Analyze Accessibility and Structure
        run: npx frontend-menu-design analyze --ruleset strict --fail-on-warning --quiet
      - name: Test Build
        run: npx frontend-menu-design build --target esmodules
```

### Monorepo Support

In monorepo architectures (using Turborepo, Nx, or Lerna), it is recommended to maintain the menu design system as a separate package (e.g., `@my-org/ui-navigation`). Use the `--cwd` flag to execute CLI commands from the root targeting the specific package:

```bash
npx frontend-menu-design generate dropdown ProfileMenu --cwd packages/ui-navigation
```

### Accessibility (a11y) First Approach

Always use the `--accessible` flag during generation. The CLI implements the WAI-ARIA Authoring Practices Guide (APG) for menu patterns. It automatically handles:
- `roving tabindex` for keyboard navigation within dropdowns.
- `aria-expanded` state toggling.
- `aria-activedescendant` for complex combobox-style menus.
- Focus trapping within modal mobile menus.

## 8. Troubleshooting and Diagnostics

If you encounter issues while using the CLI, follow these diagnostic steps:

1. **Clear Internal Cache**: The CLI caches AST parsing results for performance. Run `frontend-menu-design clean` to remove the `.menu-cache` directory.
2. **Enable Verbose Logging**: Re-run your failing command with the `--verbose` flag. Look for stack traces related to file system permissions or AST parsing errors.
3. **Configuration Validation**: Run `frontend-menu-design analyze --ruleset config-only`. This will validate your `menu.config.js` against the internal JSON schema and highlight any deprecated options or type mismatches.
4. **Peer Dependency Conflicts**: If generating components for React or Vue, ensure your project's framework version matches the CLI's expected peer dependencies. Run `npm ls react` or `npm ls vue` to check for multiple versions.

## 9. Conclusion

The `frontend-menu-design` CLI is an indispensable, enterprise-grade toolset that abstracts the immense complexities of building accessible, responsive, and performant navigation systems. By mastering the commands, configurations, and lifecycle hooks detailed in this comprehensive reference, frontend architects and development teams can significantly accelerate their workflow, enforce strict design system consistency, and maintain the highest standards of UI/UX engineering across all web properties.