# Playwright CLI Command Reference

Playwright is a robust framework for end-to-end testing of web applications. It provides a consistent and reliable interface to automate web browsers like Chrome, Firefox, and Webkit. The Playwright Command Line Interface (CLI) is a powerful tool to manage and execute tests, generate test results, and perform other operations related to Playwright projects. This document provides a comprehensive guide to understanding and using the Playwright CLI effectively.

## Table of Contents

1. [Introduction to Playwright CLI](#introduction-to-playwright-cli)
2. [Installation](#installation)
3. [Command Overview](#command-overview)
4. [Global Options](#global-options)
5. [Commands](#commands)
    - [install](#install)
    - [test](#test)
    - [codegen](#codegen)
    - [show-trace](#show-trace)
    - [install-deps](#install-deps)
    - [screenshot](#screenshot)
    - [pdf](#pdf)
    - [open](#open)
    - [run-server](#run-server)
6. [Examples](#examples)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

## Introduction to Playwright CLI

The Playwright CLI is a command-line tool designed to facilitate various tasks associated with Playwright, such as installing browser dependencies, running tests, recording scripts, and more. This CLI provides a straightforward interface to interact with the Playwright API without having to write extensive code, making it an invaluable tool for developers and testers.

## Installation

To install Playwright and its CLI, you need Node.js installed on your computer. The recommended way to install Playwright is via npm:

```bash
npm install playwright
```

This command will install Playwright along with its CLI. You can verify the installation by running:

```bash
npx playwright --version
```

## Command Overview

The Playwright CLI offers various commands to perform different operations. Here is a high-level overview of the commands available:

- **install**: Installs the necessary browser binaries.
- **test**: Runs Playwright tests.
- **codegen**: Generates Playwright scripts by recording interactions in the browser.
- **show-trace**: Opens the Playwright Trace Viewer.
- **install-deps**: Installs dependencies required by browsers.
- **screenshot**: Captures a screenshot of a web page.
- **pdf**: Generates a PDF from a web page.
- **open**: Opens the interactive Playwright Inspector.
- **run-server**: Runs a local server for testing purposes.

Each command comes with its own set of options and arguments, which provide fine-grained control over its behavior.

## Global Options

Before diving into specific commands, it's essential to understand the global options available for the Playwright CLI:

- `-h, --help`: Display help information about the command.
- `-v, --version`: Output the version number of the Playwright CLI.

These options can be used with any command to get detailed help or version information.

## Commands

### install

The `install` command is used to download and install the browser binaries required by Playwright. This ensures that you have the appropriate versions of Chrome, Firefox, and Webkit installed on your system.

#### Usage

```bash
npx playwright install [options] [browser...]
```

#### Options

- `--with-deps`: Installs both the browser binaries and their dependencies.
- `--force`: Forces reinstallation of the browser binaries.
- `--browser <browser>`: Specifies which browser to install. Options include `chromium`, `firefox`, and `webkit`.

#### Examples

```bash
# Install all browsers
npx playwright install

# Install only Chromium and Firefox
npx playwright install chromium firefox

# Install all browsers with dependencies
npx playwright install --with-deps
```

### test

The `test` command runs Playwright tests. It is the primary command for executing your test suites.

#### Usage

```bash
npx playwright test [options] [test-files...]
```

#### Options

- `--project <project>`: Runs tests in a specific project.
- `--config <file>`: Specifies a configuration file.
- `--retries <number>`: Sets the number of retries for failing tests.
- `--trace <on|off|retain-on-failure>`: Manages trace collection.
- `--timeout <timeout>`: Sets a timeout for test execution.
- `--grep <pattern>`: Filters tests by a regular expression.
- `--reporter <reporter>`: Specifies the reporter to use.
- `--workers <number>`: Sets the number of concurrent workers.

#### Examples

```bash
# Run all tests
npx playwright test

# Run specific test files
npx playwright test tests/example.spec.js

# Run tests with a specific configuration
npx playwright test --config=playwright.config.js

# Run tests using a specific reporter
npx playwright test --reporter=dot
```

### codegen

The `codegen` command generates Playwright scripts by recording interactions in the browser. This is particularly useful for creating initial test scripts quickly.

#### Usage

```bash
npx playwright codegen [options] [url]
```

#### Options

- `--target <language>`: Specifies the target language for code generation (e.g., `javascript`, `python`).
- `--output <file>`: Writes the generated script to a file.
- `--viewport-size <width,height>`: Sets the viewport size.
- `--device <device>`: Emulates a specific device during recording.
- `--wait-for-navigation`: Waits for navigations to complete.

#### Examples

```bash
# Generate a script by interacting with a web page
npx playwright codegen https://example.com

# Generate a Python script
npx playwright codegen --target=python https://example.com

# Output the script to a file
npx playwright codegen --output=example-script.js https://example.com
```

### show-trace

The `show-trace` command opens the Playwright Trace Viewer, allowing you to visualize traces collected during test execution.

#### Usage

```bash
npx playwright show-trace <trace-file>
```

#### Options

This command does not have additional options.

#### Examples

```bash
# Open a trace file in the Trace Viewer
npx playwright show-trace trace.zip
```

### install-deps

The `install-deps` command installs the necessary system dependencies required for browser execution. This is especially important for running browsers in environments like Docker.

#### Usage

```bash
npx playwright install-deps
```

#### Options

This command does not have additional options.

#### Examples

```bash
# Install system dependencies for all browsers
npx playwright install-deps
```

### screenshot

The `screenshot` command captures a screenshot of a webpage.

#### Usage

```bash
npx playwright screenshot [options] <url> <output-file>
```

#### Options

- `--viewport-size <width,height>`: Sets the viewport size for the screenshot.
- `--full-page`: Captures a full-page screenshot.
- `--device <device>`: Emulates a device while capturing the screenshot.

#### Examples

```bash
# Capture a screenshot of a webpage
npx playwright screenshot https://example.com example.png

# Capture a full-page screenshot
npx playwright screenshot --full-page https://example.com full-page.png
```

### pdf

The `pdf` command generates a PDF from a webpage. Note that this command is only supported in Chromium-based browsers.

#### Usage

```bash
npx playwright pdf [options] <url> <output-file>
```

#### Options

- `--viewport-size <width,height>`: Sets the viewport size for the PDF.
- `--format <format>`: Specifies the paper format (e.g., `A4`, `A3`).
- `--margin <top,right,bottom,left>`: Sets the margins for the PDF.

#### Examples

```bash
# Generate a PDF from a webpage
npx playwright pdf https://example.com example.pdf

# Generate a PDF with specific margins
npx playwright pdf --margin 10,10,10,10 https://example.com example.pdf
```

### open

The `open` command launches the interactive Playwright Inspector, allowing you to debug and explore your tests interactively.

#### Usage

```bash
npx playwright open [url]
```

#### Options

This command does not have additional options.

#### Examples

```bash
# Open the Playwright Inspector with a specific URL
npx playwright open https://example.com
```

### run-server

The `run-server` command sets up a local server for testing purposes. This is useful for serving static files or running server-side scripts during testing.

#### Usage

```bash
npx playwright run-server <directory>
```

#### Options

- `--port <number>`: Specifies the port on which the server should run.

#### Examples

```bash
# Run a local server serving files from the 'public' directory
npx playwright run-server public

# Run a local server on a specific port
npx playwright run-server public --port 8080
```

## Examples

### Running Tests Across Multiple Browsers

```bash
# Run tests using all supported browsers
npx playwright test --project=chromium --project=firefox --project=webkit
```

### Generating a Script and Saving to a File

```bash
# Record interactions on a webpage and save the script in Python
npx playwright codegen --target=python --output=script.py https://example.com
```

### Using Trace Viewer

```bash
# Run tests and collect traces
npx playwright test --trace=on

# View the collected trace
npx playwright show-trace trace.zip
```

## Best Practices

- **Use Configuration Files**: To manage settings and browser options consistently, use a Playwright configuration file (`playwright.config.js`).
- **Leverage Trace Viewer**: Use traces to debug complex test scenarios by visualizing step-by-step execution.
- **Regular Updates**: Keep Playwright and browsers updated to benefit from the latest features and security patches.
- **Parallel Execution**: Utilize multiple workers to run tests in parallel, reducing overall execution time.

## Troubleshooting

- **Browser Installation Issues**: Ensure that all dependencies are installed using `npx playwright install-deps`.
- **Test Failures**: Use the `--retries` option to retry failed tests and `--trace=retain-on-failure` to collect traces for analysis.
- **Unsupported Features**: Some commands are browser-specific, like PDF generation, which is only supported in Chromium.

This comprehensive reference aims to equip you with the knowledge to effectively use the Playwright CLI in your testing workflows. Playwright's rich feature set, combined with the powerful CLI, provides a versatile environment for automated browser testing.