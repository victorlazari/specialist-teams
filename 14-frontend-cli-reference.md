# Frontend CLI Command Reference

## Introduction

Welcome to the comprehensive CLI command reference for `frontend-cli`, the command-line interface designed to streamline and enhance the development workflow for frontend projects. This documentation will cover every aspect of `frontend-cli`, from installation and configuration to detailed usage of each command. Whether you're initializing a new project, building for production, or running tests, this guide provides all the information you need to effectively utilize `frontend-cli`.

## Installation

To start using `frontend-cli`, you must first install it. The package is distributed via npm, so ensure you have Node.js and npm installed on your system.

### Prerequisites

- **Node.js:** Version 14 or higher is recommended.
- **npm:** Version 6 or higher.

### Installation Steps

1. Open your terminal.
2. Run the following command to install `frontend-cli` globally:

   ```bash
   npm install -g frontend-cli
   ```

3. Verify the installation by executing:

   ```bash
   frontend-cli --version
   ```

   You should see the version number of `frontend-cli` printed in the console.

## Global Flags

Global flags can be used with any command in `frontend-cli` to modify its behavior or output.

- `-h`, `--help`: Display help information for a command.
- `-v`, `--version`: Output the version number of `frontend-cli`.
- `--verbose`: Output additional information during execution.
- `--quiet`: Suppress all output except for errors.
- `--config <path>`: Specify a path to a configuration file.

## Commands

### `init`

The `init` command scaffolds a new frontend project with a predefined structure and configuration.

#### Usage

```bash
frontend-cli init [options] <project-name>
```

#### Options

- `-t`, `--template <name>`: Specify a project template (e.g., `react`, `vue`, `angular`).
- `--no-install`: Skip the installation of dependencies.
- `--git`: Initialize a git repository in the new project directory.

#### Examples

Initialize a new React project named `my-app`:

```bash
frontend-cli init -t react my-app
```

Initialize a new Vue project without installing dependencies:

```bash
frontend-cli init -t vue my-app --no-install
```

### `build`

The `build` command compiles the source code into a production-ready bundle.

#### Usage

```bash
frontend-cli build [options]
```

#### Options

- `--env <environment>`: Set the environment for the build (default is `production`).
- `--output <directory>`: Specify the output directory for the build files.
- `--source-map`: Generate source maps for the build.
- `--minify`: Minify the output (enabled by default in production).

#### Examples

Build the project for production:

```bash
frontend-cli build
```

Build the project with source maps and output to a specific directory:

```bash
frontend-cli build --source-map --output dist/
```

### `serve`

The `serve` command starts a local development server with live reloading capabilities.

#### Usage

```bash
frontend-cli serve [options]
```

#### Options

- `-p`, `--port <number>`: Specify the port on which the server will run (default is `3000`).
- `--open`: Automatically open the default web browser.
- `--proxy <url>`: Proxy API requests to the specified backend server.
  
#### Examples

Start the development server on port 4000:

```bash
frontend-cli serve --port 4000
```

Start the development server and proxy API requests:

```bash
frontend-cli serve --proxy http://localhost:5000
```

### `test`

The `test` command runs the test suite for the project.

#### Usage

```bash
frontend-cli test [options]
```

#### Options

- `--watch`: Run tests in watch mode.
- `--coverage`: Generate a code coverage report.
- `--bail`: Exit after the first test failure.

#### Examples

Run the test suite with code coverage:

```bash
frontend-cli test --coverage
```

Run tests in watch mode:

```bash
frontend-cli test --watch
```

### `lint`

The `lint` command analyzes the codebase for potential errors and style issues.

#### Usage

```bash
frontend-cli lint [options] [files...]
```

#### Options

- `--fix`: Automatically fix problems where possible.
- `--format <formatter>`: Specify an output format (e.g., `stylish`, `json`).

#### Examples

Lint all files in the `src` directory:

```bash
frontend-cli lint src/
```

Lint and automatically fix issues:

```bash
frontend-cli lint --fix
```

### `deploy`

The `deploy` command automates the deployment of the frontend application to a specified environment.

#### Usage

```bash
frontend-cli deploy [options]
```

#### Options

- `--env <environment>`: Set the deployment environment (e.g., `staging`, `production`).
- `--token <api-token>`: Provide an API token for authentication.
- `--dry-run`: Simulate the deployment process without making any changes.

#### Examples

Deploy to the production environment:

```bash
frontend-cli deploy --env production --token my-api-token
```

Simulate a deployment to staging:

```bash
frontend-cli deploy --env staging --dry-run
```

### `analyze`

The `analyze` command provides insights into the bundle size and composition.

#### Usage

```bash
frontend-cli analyze [options]
```

#### Options

- `--json`: Output the analysis in JSON format.
- `--html`: Generate an interactive HTML report.

#### Examples

Analyze the bundle and generate an HTML report:

```bash
frontend-cli analyze --html
```

Output the analysis in JSON format:

```bash
frontend-cli analyze --json
```

## Configuration

`frontend-cli` can be configured using a configuration file. By default, it looks for a `frontend.config.js` file in the project root.

### Sample Configuration File

```javascript
module.exports = {
  build: {
    output: 'dist',
    sourceMap: true,
    minify: true,
  },
  serve: {
    port: 3000,
    proxy: 'http://localhost:5000',
  },
  deploy: {
    environments: {
      production: {
        apiToken: process.env.PRODUCTION_API_TOKEN,
      },
      staging: {
        apiToken: process.env.STAGING_API_TOKEN,
      },
    },
  },
};
```

## Environment Variables

Environment variables can be used to configure `frontend-cli` dynamically. Commonly used environment variables include:

- `NODE_ENV`: Set the environment mode (`development`, `production`).
- `PORT`: Specify the port for the development server.
- `API_TOKEN`: Provide an API token for deployment.

## Best Practices

- **Use a `.env` File:** Store environment-specific variables in a `.env` file and load them using a library like `dotenv`.
- **Version Control:** Keep your `frontend.config.js` file under version control to track configuration changes.
- **Scripts Management:** Use npm scripts to encapsulate common `frontend-cli` commands for consistency and ease of use.
- **Regular Updates:** Regularly update `frontend-cli` to benefit from new features and security patches.

## Conclusion

This comprehensive guide aims to provide you with all the necessary information to effectively use `frontend-cli` in your frontend development workflow. By leveraging the commands and configurations outlined in this documentation, you can optimize your development process, ensure code quality, and streamline deployment. Remember to explore each command with its various options to fully utilize the capabilities of `frontend-cli`. Happy coding!