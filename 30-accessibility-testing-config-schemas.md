# Accessibility Testing Configuration Schemas Guide

## Introduction

Accessibility testing is a critical aspect of software development, ensuring that applications and websites are usable by people with disabilities. This guide provides a comprehensive overview of configuration schemas used in accessibility testing tools and frameworks. We will cover the structure of configuration files, detail each configuration field, discuss default values, and offer best practices for setting up and managing accessibility testing configurations.

## Table of Contents

1. [Overview of Accessibility Testing](#overview-of-accessibility-testing)
2. [Understanding Configuration Schemas](#understanding-configuration-schemas)
3. [Common Accessibility Testing Tools](#common-accessibility-testing-tools)
4. [Configuration File Structure](#configuration-file-structure)
5. [Detailed Configuration Fields](#detailed-configuration-fields)
   - [General Settings](#general-settings)
   - [Rule Settings](#rule-settings)
   - [Output Settings](#output-settings)
   - [Target Settings](#target-settings)
6. [Default Values](#default-values)
7. [Best Practices](#best-practices)
8. [Examples](#examples)
9. [Conclusion](#conclusion)

## Overview of Accessibility Testing

Accessibility testing ensures that digital content is accessible to all users, including those with disabilities. This involves checking for compliance with standards such as WCAG (Web Content Accessibility Guidelines), Section 508, and others. Automated tools streamline this process by detecting common accessibility issues.

## Understanding Configuration Schemas

A configuration schema is a structured representation of configuration data, typically defined in formats like JSON or YAML. It specifies the settings and parameters that an accessibility testing tool can use to perform tests. This schema allows for customization and flexibility, enabling users to tailor tests to specific requirements.

## Common Accessibility Testing Tools

Several tools facilitate automated accessibility testing. Some popular ones include:

- **aXe-core**: A JavaScript library for integrating accessibility checks into existing workflows.
- **Lighthouse**: An open-source tool from Google for auditing web pages, including accessibility checks.
- **Pa11y**: A command-line tool and CI service for automated accessibility testing.
- **WAVE**: A suite of evaluation tools that help authors make their web content more accessible.

Each of these tools has its own configuration schema, which we will explore in detail.

## Configuration File Structure

Configuration files for accessibility testing tools generally follow a structured format such as JSON or YAML. Here is a typical structure:

```yaml
{
  "general": {
    "toolName": "aXe-core",
    "version": "4.3",
    "enabled": true
  },
  "rules": {
    "include": ["color-contrast", "image-alt"],
    "exclude": ["aria-hidden-focus"],
    "custom": {
      "custom-rule-id": {
        "selector": ".example",
        "enabled": true,
        "metadata": {
          "impact": "moderate",
          "description": "Example custom rule"
        }
      }
    }
  },
  "output": {
    "format": "json",
    "destination": "results/accessibility-report.json"
  },
  "target": {
    "urls": ["https://example.com"],
    "viewport": {
      "width": 1280,
      "height": 720
    },
    "browsers": ["chrome", "firefox"]
  }
}
```

### YAML Example

```yaml
general:
  toolName: "aXe-core"
  version: "4.3"
  enabled: true

rules:
  include:
    - "color-contrast"
    - "image-alt"
  exclude:
    - "aria-hidden-focus"
  custom:
    custom-rule-id:
      selector: ".example"
      enabled: true
      metadata:
        impact: "moderate"
        description: "Example custom rule"

output:
  format: "json"
  destination: "results/accessibility-report.json"

target:
  urls:
    - "https://example.com"
  viewport:
    width: 1280
    height: 720
  browsers:
    - "chrome"
    - "firefox"
```

## Detailed Configuration Fields

### General Settings

- **`toolName`**: The name of the accessibility testing tool.
  - **Type**: `string`
  - **Default**: `"aXe-core"`
  - **Description**: Specifies the tool to be used for testing.

- **`version`**: The version of the tool.
  - **Type**: `string`
  - **Default**: Latest stable version
  - **Description**: Indicates which version of the tool's rules and functionalities to apply.

- **`enabled`**: Flag to enable or disable the testing.
  - **Type**: `boolean`
  - **Default**: `true`
  - **Description**: Toggles the execution of accessibility tests.

### Rule Settings

- **`include`**: A list of rule IDs to include in the testing process.
  - **Type**: `array of strings`
  - **Default**: All rules
  - **Description**: Specifies which rules should be applied during testing.

- **`exclude`**: A list of rule IDs to exclude.
  - **Type**: `array of strings`
  - **Default**: `[]`
  - **Description**: Lists rules that should be ignored during testing.

- **`custom`**: Custom rules defined by the user.
  - **Type**: `object`
  - **Default**: `{}`
  - **Description**: Allows users to define their own rules with specific selectors and metadata.
  - **Fields**:
    - **`selector`**: CSS selector for elements the rule applies to.
      - **Type**: `string`

    - **`enabled`**: Whether the custom rule is active.
      - **Type**: `boolean`
      - **Default**: `true`

    - **`metadata`**: Additional information about the custom rule.
      - **Type**: `object`
      - **Fields**:
        - **`impact`**: Severity of the issue (e.g., "minor", "moderate", "serious", "critical").
          - **Type**: `string`

        - **`description`**: Description of what the rule checks.
          - **Type**: `string`

### Output Settings

- **`format`**: Format of the output report.
  - **Type**: `string`
  - **Default**: `"json"`
  - **Description**: Specifies the format (e.g., "json", "html", "csv") of the generated accessibility report.

- **`destination`**: Path where the report will be saved.
  - **Type**: `string`
  - **Default**: `"results/accessibility-report.json"`
  - **Description**: File path to save the output report.

### Target Settings

- **`urls`**: List of URLs to test.
  - **Type**: `array of strings`
  - **Default**: `[]`
  - **Description**: Specifies which web pages to perform accessibility testing on.

- **`viewport`**: The viewport dimensions for testing.
  - **Type**: `object`
  - **Fields**:
    - **`width`**: Width of the viewport.
      - **Type**: `integer`
      - **Default**: `1280`

    - **`height`**: Height of the viewport.
      - **Type**: `integer`
      - **Default**: `720`

- **`browsers`**: List of browsers to simulate.
  - **Type**: `array of strings`
  - **Default**: `["chrome"]`
  - **Description**: Specifies which browsers to use for testing.

## Default Values

Default values are pre-defined settings that apply when no specific configuration is provided. They ensure the tool functions out-of-the-box without requiring extensive setup. It's crucial to understand and possibly adjust these defaults to fit the testing needs better.

- **General Settings**: Default tool is `aXe-core`, and testing is enabled by default.
- **Rule Settings**: Includes all rules unless specified otherwise; custom rules are not defined.
- **Output Settings**: Outputs in JSON format to a default path.
- **Target Settings**: Default viewport is set to a standard size, and testing is performed on Chrome.

## Best Practices

1. **Start with Defaults**: Use default settings for initial testing to quickly identify major accessibility issues.

2. **Customize as Needed**: Modify rule settings to focus on specific accessibility concerns pertinent to your application or website.

3. **Define Custom Rules**: Where standard rules fall short, create custom rules to address unique accessibility requirements.

4. **Frequent Reports**: Generate reports regularly to track accessibility improvements over time.

5. **Integrate with CI/CD**: Incorporate accessibility testing into your continuous integration and deployment pipelines for ongoing compliance.

6. **Keep Configurations Versioned**: Maintain your configuration files under version control to track changes and facilitate collaboration.

## Examples

### Basic Configuration Example

```json
{
  "general": {
    "toolName": "Lighthouse",
    "enabled": true
  },
  "output": {
    "format": "html",
    "destination": "reports/lighthouse-a11y.html"
  },
  "target": {
    "urls": ["https://example.com"]
  }
}
```

### Advanced Configuration Example

```yaml
general:
  toolName: "Pa11y"
  version: "5.0"
  enabled: true

rules:
  include:
    - "label"
    - "heading-order"
  custom:
    unique-button-label:
      selector: "button"
      enabled: true
      metadata:
        impact: "serious"
        description: "Buttons should have unique labels"

output:
  format: "csv"
  destination: "reports/pa11y-results.csv"

target:
  urls:
    - "https://example.com"
  viewport:
    width: 1440
    height: 900
  browsers:
    - "chrome"
    - "firefox"
```

## Conclusion

Configuring accessibility testing tools effectively requires a thorough understanding of the available schemas and options. By leveraging the configuration settings discussed in this guide, you can tailor accessibility tests to meet your specific needs, ensuring comprehensive and efficient testing. Regular testing and adherence to best practices will help maintain and improve accessibility compliance, providing a better user experience for all users.

For further reading and updates, refer to the documentation of the specific tool you are using, as accessibility standards and testing tools frequently evolve.