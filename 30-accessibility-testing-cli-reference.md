# Accessibility Testing CLI Command Reference

This document serves as a comprehensive reference for the `a11y-test` command-line interface (CLI) tool. The `a11y-test` tool is designed to help developers and testers ensure that their web applications meet accessibility standards. This CLI supports various commands, each with a set of options and arguments to provide flexibility and control over the accessibility testing process.

## Table of Contents

1. [Installation](#installation)
2. [Basic Usage](#basic-usage)
3. [Commands Overview](#commands-overview)
4. [Command Details](#command-details)
   - [scan](#scan)
   - [report](#report)
   - [configure](#configure)
   - [validate](#validate)
   - [help](#help)
5. [Examples](#examples)

---

## Installation

To install the `a11y-test` CLI tool, you can use the following command with npm:

```bash
npm install -g a11y-test
```

## Basic Usage

The `a11y-test` tool syntax follows a simple pattern:

```bash
a11y-test <command> [options]
```

## Commands Overview

- **scan**: Scans a web page or application for accessibility issues.
- **report**: Generates a report from a previous scan.
- **configure**: Sets up or modifies configuration settings for the tool.
- **validate**: Validates the configuration settings.
- **help**: Displays help information about the CLI tool or its commands.

## Command Details

### scan

The `scan` command analyzes web pages for accessibility issues.

#### Syntax

```bash
a11y-test scan [options] <url>
```

#### Options

- `-o, --output <file>`: Specify the output file for the scan results.
- `-f, --format <format>`: Specify the output format (json, html, csv). Default is `json`.
- `-r, --ruleset <ruleset>`: Define a specific ruleset to use (e.g., WCAG2A, WCAG2AA, WCAG2AAA).
- `-d, --depth <depth>`: Set the depth for scanning links from the initial URL. Default is `0`.
- `-c, --config <file>`: Use a specific configuration file.
- `-i, --ignore <rules>`: Comma-separated list of rules to ignore.
- `-t, --timeout <milliseconds>`: Set a timeout for the scan process. Default is `30000` (30 seconds).

#### Examples

```bash
a11y-test scan https://example.com -o results.json -f json
a11y-test scan https://example.com -r WCAG2AA -d 2
a11y-test scan https://example.com -i rule1,rule2 -t 60000
```

### report

The `report` command is used to generate a detailed accessibility report from a previous scan.

#### Syntax

```bash
a11y-test report [options] <scan-file>
```

#### Options

- `-f, --format <format>`: Specify the report format (html, pdf, markdown). Default is `html`.
- `-o, --output <file>`: Specify the output file for the report.
- `-t, --template <path>`: Use a custom template for generating the report.

#### Examples

```bash
a11y-test report scan-results.json -f pdf -o detailed-report.pdf
a11y-test report scan-results.json -t custom-template.html
```

### configure

The `configure` command allows users to set or modify configuration settings for `a11y-test`.

#### Syntax

```bash
a11y-test configure [options]
```

#### Options

- `-l, --list`: List all current configuration settings.
- `-s, --set <key=value>`: Set a configuration option.
- `-r, --reset`: Reset all configurations to their default values.
- `-f, --file <path>`: Specify a configuration file to read settings from.

#### Examples

```bash
a11y-test configure --list
a11y-test configure --set ruleset=WCAG2AA
a11y-test configure --reset
a11y-test configure --file custom-config.json
```

### validate

The `validate` command checks if the current configuration is valid.

#### Syntax

```bash
a11y-test validate [options]
```

#### Options

- `-c, --config <file>`: Validate a specific configuration file.

#### Examples

```bash
a11y-test validate
a11y-test validate --config custom-config.json
```

### help

The `help` command provides help information for the `a11y-test` tool or its specific commands.

#### Syntax

```bash
a11y-test help [command]
```

#### Examples

```bash
a11y-test help
a11y-test help scan
```

## Examples

### Example 1: Basic Scan with Default Settings

To perform a basic scan of a website with default settings:

```bash
a11y-test scan https://example.com
```

### Example 2: Scan with Custom Ruleset and Output File

To scan a website using the WCAG2AA ruleset and save the results to a JSON file:

```bash
a11y-test scan https://example.com -r WCAG2AA -o results.json -f json
```

### Example 3: Generate a PDF Report from Scan Results

To generate a PDF report from previously obtained scan results:

```bash
a11y-test report results.json -f pdf -o accessibility-report.pdf
```

### Example 4: Configure Tool to Use a Custom Ruleset

To configure the `a11y-test` tool to always use the WCAG2AAA ruleset:

```bash
a11y-test configure --set ruleset=WCAG2AAA
```

### Example 5: Validate Custom Configuration File

To validate a custom configuration file before using it:

```bash
a11y-test validate --config custom-config.json
```

---

This document provides an extensive guide to using the `a11y-test` command-line tool for accessibility testing. Each command and option is designed to offer flexibility and precision in evaluating web accessibility, ensuring that your applications meet the required standards.