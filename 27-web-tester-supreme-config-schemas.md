# Configuration Schemas: web-tester-supreme

## 1. Introduction

The `web-tester-supreme` framework is an enterprise-grade, highly scalable web testing automation tool designed for complex, distributed web applications. A critical aspect of leveraging its full potential lies in understanding and properly configuring its extensive schema definitions. This comprehensive guide provides a deep dive into every configuration file, field, default value, and best practice associated with `web-tester-supreme`. 

Proper configuration ensures optimal performance, reliable test execution, and seamless integration into Continuous Integration/Continuous Deployment (CI/CD) pipelines. The configuration schemas are primarily defined in YAML and JSON formats, offering a balance between human readability and machine parsability. In modern software development lifecycles, the testing framework must adapt to various environments—from local developer machines to ephemeral CI containers and large-scale cloud execution grids. The configuration schemas detailed in this document provide the necessary flexibility and control to achieve this adaptability.

This document is intended for QA engineers, SDETs (Software Development Engineers in Test), and DevOps professionals who are responsible for setting up, maintaining, and optimizing the testing infrastructure. By the end of this guide, you will have a thorough understanding of how to manipulate `web-tester-supreme` to suit any testing scenario, no matter how complex.

## 2. Core Configuration File (`web-tester.yaml`)

The `web-tester.yaml` file serves as the central nervous system for the `web-tester-supreme` execution environment. It dictates global behaviors, resource allocation, and default fallback mechanisms. This file is typically located at the root of your testing repository and is automatically detected by the CLI runner.

### 2.1. General Settings

The general settings block defines the fundamental operational parameters of the testing framework. These settings govern the high-level behavior of the test runner and provide metadata for reporting.

The `project_name` field is a string that represents the unique identifier for the testing project. This value is extensively used in reporting, logging, and when integrating with external test management systems. It helps distinguish results when multiple projects are executed on the same grid. The default value is `"default-project"`. An example usage would be `project_name: "e-commerce-frontend-v2"`.

The `version` field is a string indicating the schema version being utilized. This is crucial for backward compatibility. As `web-tester-supreme` evolves, new configuration options are added. Specifying the version ensures that the parser interprets the file correctly and applies the appropriate defaults. The default value is `"1.0"`. An example usage would be `version: "2.1"`.

The `execution_mode` field is an enum that determines how test suites are executed. The available options are `sequential`, `parallel`, and `grid`. The `sequential` mode runs tests one after another, which is best for debugging. The `parallel` mode runs tests concurrently on the local machine, utilizing multiple CPU cores. The `grid` mode distributes tests across a remote cluster of machines. The default value is `"sequential"`. An example usage would be `execution_mode: "parallel"`.

The `max_retries` field is an integer that sets the global maximum number of times a failed test will be retried before being marked as a definitive failure. This is a critical setting for handling flaky tests caused by transient network issues or unpredictable UI rendering delays. The default value is `0`. An example usage would be `max_retries: 3`.

The `fail_fast` field is a boolean that, if set to true, will cause the test runner to immediately halt execution upon the first test failure. This is highly useful in CI environments to save compute resources when a critical regression is detected early in the suite. The default value is `false`. An example usage would be `fail_fast: true`.

### 2.2. Browser Configuration

The `browser` block specifies the target environments for web testing. `web-tester-supreme` supports multi-browser execution natively, allowing you to verify cross-browser compatibility with ease.

The `type` field is an enum that specifies the primary browser engine to use. The available options are `chrome`, `firefox`, `safari`, `edge`, and `webkit`. You can also define multiple projects within the YAML to run tests against multiple browsers simultaneously. The default value is `"chrome"`.

The `headless` field is a boolean that determines whether to run the browser in headless mode (without a graphical user interface). Headless mode is highly recommended for CI environments as it consumes significantly fewer system resources and executes faster. The default value is `true`.

The `executable_path` field is a string that provides an optional path to a specific browser binary. This is useful if you need to test against a specific, non-standard version of a browser (e.g., a nightly build or a custom-compiled Chromium). The default value is `null`, which uses the bundled browser. An example usage would be `executable_path: "/usr/bin/google-chrome-stable"`.

The `viewport` field is an object that defines the simulated screen dimensions. This is essential for responsive design testing. It contains two integer fields: `width` and `height`. The default `width` is `1280` pixels, and the default `height` is `720` pixels.

The `device_scale_factor` field is a float that simulates high-DPI (Retina) displays. The default value is `1.0`. An example usage would be `device_scale_factor: 2.0`.

The `is_mobile` field is a boolean that simulates mobile browser behavior, including touch events and mobile user agents. The default value is `false`.

The `args` field is an array of strings that represents command-line arguments passed directly to the browser executable. This provides low-level control over the browser instance. The default value is an empty array `[]`. An example usage would be `args: ["--disable-gpu", "--no-sandbox", "--disable-dev-shm-usage"]`.

### 2.3. Timeout Settings

Timeouts are crucial for preventing hanging tests and ensuring efficient resource utilization. In a distributed environment, a single hanging test can tie up a worker node indefinitely. All timeout values are specified in milliseconds.

The `global_timeout` field is an integer that sets the maximum duration allowed for the entire test suite execution. If the suite exceeds this time, the runner will forcefully terminate all workers and exit with an error code. The default value is `3600000` (1 hour).

The `test_timeout` field is an integer that sets the maximum duration allowed for a single test case. If a test exceeds this time, it is marked as failed (or retried, depending on `max_retries`). The default value is `30000` (30 seconds).

The `navigation_timeout` field is an integer that sets the maximum time to wait for a page navigation (e.g., `page.goto()`) to complete. The default value is `15000` (15 seconds).

The `element_timeout` field is an integer that sets the maximum time to wait for an element to become visible, interactive, or present in the DOM during actions like `click()` or `fill()`. The default value is `5000` (5 seconds).

The `expect_timeout` field is an integer that sets the maximum time to wait for an assertion (e.g., `expect(locator).toBeVisible()`) to pass. The default value is `5000` (5 seconds).

## 3. Environment Variables

While `web-tester.yaml` provides static configuration, environment variables offer dynamic overrides. This is essential for managing secrets, credentials, and environment-specific settings (e.g., pointing tests to a staging environment versus a production environment) without modifying the source code.

The `WTS_BASE_URL` environment variable overrides the base URL defined in the configuration files. This is the most commonly used environment variable, allowing the same test suite to run against different deployment environments.

The `WTS_API_KEY` environment variable injects the API key required for interacting with external services, reporting dashboards, or the `web-tester-supreme` cloud grid.

The `WTS_EXECUTION_MODE` environment variable dynamically changes the execution mode. For example, a developer might set `WTS_EXECUTION_MODE=sequential` locally for debugging, while the CI server uses the default `parallel` mode.

The `WTS_LOG_LEVEL` environment variable sets the verbosity of the console output. Valid values are `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, and `FATAL`. Setting this to `DEBUG` is invaluable when troubleshooting framework issues.

The `WTS_WORKERS` environment variable overrides the number of concurrent workers defined in the YAML file.

The `WTS_PROJECT` environment variable forces the runner to execute only a specific project defined in the configuration file.

## 4. Test Suite Configuration (`suite.json`)

The `suite.json` file is used to group, filter, and organize individual test files into logical suites. This allows for targeted execution, which is vital for optimizing CI/CD pipelines. Instead of running thousands of tests for every minor commit, you can define a "smoke" suite that runs quickly and provides immediate feedback.

### 4.1. Suite Definition Schema

The schema for `suite.json` is designed to be highly expressive, supporting glob patterns and tag-based filtering.

```json
{
  "suites": {
    "smoke": {
      "description": "Critical path tests for deployment verification. Must run in under 5 minutes.",
      "files": [
        "tests/login/*.spec.js",
        "tests/checkout/basic.spec.js",
        "tests/navigation/header.spec.js"
      ],
      "tags": ["@critical", "@smoke"]
    },
    "regression": {
      "description": "Full regression suite covering all features and edge cases.",
      "files": [
        "tests/**/*.spec.js"
      ],
      "exclude": [
        "tests/experimental/**/*.spec.js",
        "tests/flaky/**/*.spec.js"
      ]
    },
    "api-integration": {
      "description": "Tests that verify frontend integration with backend APIs.",
      "files": [
        "tests/api/**/*.spec.js"
      ],
      "retries": 2
    }
  }
}
```

The `suites` object is the root object containing named suite definitions. The keys (e.g., `smoke`, `regression`) are the names used when invoking the CLI (e.g., `web-tester run --suite=smoke`).

The `description` field within a suite definition is a string that provides a human-readable description of the suite's purpose. This is displayed in the CLI help and in generated reports.

The `files` field within a suite definition is an array of strings containing glob patterns specifying which test files to include in this suite. `web-tester-supreme` uses standard globbing rules (e.g., `**` for recursive directory matching).

The `exclude` field within a suite definition is an array of strings containing glob patterns specifying files to explicitly exclude, even if they are matched by the `files` array. This is useful for temporarily disabling flaky tests or excluding experimental features.

The `tags` field within a suite definition is an array of strings containing tags used for further filtering during execution. If tags are specified, only tests within the matched files that *also* contain these tags will be executed.

The `retries` field within a suite definition is an integer that sets suite-specific retry logic. This overrides the global `max_retries` setting for this specific suite.

## 5. Network & Proxy Settings

For enterprise environments, configuring network routing and proxy settings is often mandatory. Corporate firewalls and secure networks require traffic to be routed through specific proxies. Furthermore, advanced testing scenarios often require intercepting and mocking network requests to simulate backend failures or isolate the frontend. `web-tester-supreme` provides robust network interception and proxy configuration schemas.

### 5.1. Proxy Configuration

Located within the `web-tester.yaml` under the `network` block.

The `proxy` object contains settings for the proxy server. The `server` field is a string representing the proxy server address (e.g., `http://proxy.corporate.internal:8080`). It supports HTTP, HTTPS, and SOCKS5 protocols. The `bypass` field is an array of strings containing a list of hostnames or IP addresses that should bypass the proxy. This is typically used for `localhost` or internal staging servers that are accessible directly. An example usage would be `bypass: ["localhost", "127.0.0.1", "*.internal.company.com"]`. The `username` field is a string for proxy authentication. The `password` field is a string for proxy authentication. **Security Warning:** It is highly recommended to use environment variables for passwords rather than hardcoding them in the YAML file (e.g., `password: "${WTS_PROXY_PASSWORD}"`).

### 5.2. Network Interception

Network interception allows you to manipulate HTTP traffic on the fly. This is configured under the `network.intercept` block.

The `intercept` field is an array of objects that defines rules for mocking or blocking network requests. Each object contains a `url_pattern` field, which is a string representing a regex or glob pattern matching the request URL. The `action` field is an enum (`block`, `mock`, `allow`, `modify`) that specifies the action to take when a request matches the pattern. The `mock_response` object is required if `action` is `mock`. It contains a `status` field (integer) for the HTTP status code to return, a `body` field (string/object) for the mock response body, and a `headers` field (object) for custom headers to include in the mock response. The `modify_request` object is required if `action` is `modify`. It contains a `headers` field (object) for headers to add or overwrite in the outgoing request, and a `post_data` field (string) to overwrite the payload of POST/PUT requests.

## 6. Authentication Schemas

Handling authentication state efficiently is critical for test performance. Logging in via the UI for every single test case is a massive anti-pattern that drastically increases execution time and introduces flakiness. `web-tester-supreme` supports state preservation, allowing you to log in once, save the state, and reuse it across multiple tests.

### 6.1. State Storage (`auth.json`)

The framework can serialize the browser's storage (Cookies, LocalStorage, SessionStorage) into an `auth.json` file. This file acts as a snapshot of an authenticated session.

The `cookies` field is an array of objects representing standard cookie definitions. Each object contains fields for `name` (string), `value` (string), `domain` (string), `path` (string), `expires` (number, Unix timestamp), `httpOnly` (boolean), and `secure` (boolean).

The `origins` field is an array of objects that captures storage for specific domains. Each object contains an `origin` field (string) representing the base URL (e.g., `https://example.com`), a `localStorage` field (array of objects) representing key-value pairs for the LocalStorage state, and a `sessionStorage` field (array of objects) representing key-value pairs for the SessionStorage state.

### 6.2. Authentication Configuration in `web-tester.yaml`

To utilize the `auth.json` file, you must configure the `auth` block in your main configuration file.

The `auth` object contains settings for authentication state management. The `state_path` field is a string representing the path to the `auth.json` file. The default value is `.wts/auth.json`. The `auto_save` field is a boolean that determines whether to automatically save the state after a designated setup test. If true, the framework will monitor storage changes during the setup phase and write them to `state_path`. The default value is `false`. The `setup_project` field is a string representing the name of the project (defined in the YAML) that is responsible for performing the login and generating the state file. This project is guaranteed to run before any other projects that depend on the state.

## 7. Reporting & Output Configuration

Comprehensive reporting is essential for analyzing test results, identifying trends, and communicating quality metrics to stakeholders. The `reporters` block in `web-tester.yaml` configures the output formats. `web-tester-supreme` supports multiple reporters simultaneously.

The `reporters` field is an array of objects or strings that specifies the desired output formats. The `"list"` option provides standard console output, printing a real-time list of tests as they execute. The `"dot"` option provides minimalist console output, printing a dot for each passing test and an 'F' for failures, which is ideal for CI environments where console logs should be kept concise. The `"json"` option outputs detailed results to a JSON file, useful for custom integrations or feeding data into analytics platforms. It accepts options like `outputFile: "results/report.json"`. The `"html"` option generates a rich, interactive HTML report that includes trace viewers, screenshots, and video recordings of failed tests. It accepts options like `outputFolder: "results/html-report"` and `open: "never"` (controls whether the report opens automatically after execution). The `"junit"` option generates JUnit XML format, which is the industry standard for CI integrations (Jenkins, GitLab CI, CircleCI). CI servers parse this XML to display test trends and failure summaries in their native UIs. It accepts options like `outputFile: "results/junit.xml"`. The `"github"` option is a specialized reporter that utilizes GitHub Actions annotations to highlight test failures directly in Pull Request diffs.

## 8. Advanced Tuning & Performance

For large-scale test suites containing thousands of tests, fine-tuning the execution engine is necessary to minimize execution time and resource consumption. The schemas in this section provide granular control over concurrency and distribution.

### 8.1. Worker Configuration

Workers are independent OS processes that execute tests. Increasing the number of workers allows for parallel execution.

The `workers` field is an integer or string that defines the number of concurrent worker processes. The default value is `1` (Sequential execution). An integer example would be `workers: 4` (Spawns exactly 4 worker processes). A percentage example would be `workers: "50%"` (Calculates the number of logical CPU cores available and uses 50% of them. This is highly recommended for CI environments where the underlying hardware may vary).

### 8.2. Sharding

While workers provide parallelization on a single machine, sharding allows splitting a test suite across multiple distinct machines (nodes) in a CI environment. This is the key to achieving massive scale.

The `shard` object contains settings for sharding. The `total` field is an integer representing the total number of shards (machines) participating in the execution. The `current` field is an integer representing the index of the current shard (1-based). Sharding is almost always configured via CLI arguments or environment variables in the CI pipeline, rather than hardcoded in the YAML.

### 8.3. Resource Limits

To prevent the testing framework from consuming all available system memory (which can lead to OOM kills in containerized environments), you can define resource limits.

The `memory_limit_mb` field is an integer that sets the maximum amount of memory (in Megabytes) a single worker process is allowed to consume. If a worker exceeds this limit, it is gracefully restarted. The default value is `2048` (2 GB).

## 9. Default Values & Overrides Hierarchy

Understanding the order of precedence for configuration values is crucial for debugging unexpected behavior. When a configuration value is defined in multiple places, `web-tester-supreme` resolves it using a strict hierarchy. The framework resolves configuration in the following order (from highest to lowest priority):

First, CLI Arguments have the absolute highest priority. Flags passed directly to the execution command will override any other settings (e.g., `web-tester run --timeout=10000`).

Second, Environment Variables prefixed with `WTS_` take precedence over configuration files (e.g., `WTS_TIMEOUT=10000`).

Third, Project-Specific Config defined within a specific `projects` array block in `web-tester.yaml` allows different projects to have different settings (e.g., Project A uses Chrome, Project B uses Firefox).

Fourth, Global Config at the root level in `web-tester.yaml` applies to all projects unless overridden.

Fifth, Framework Defaults are the hardcoded default values within the `web-tester-supreme` source code.

For example, if `timeout` is set to `30000` in `web-tester.yaml`, but the CI script runs `web-tester run --timeout=60000`, the effective timeout will be `60000`.

## 10. Best Practices

To maximize the effectiveness, stability, and maintainability of `web-tester-supreme`, adhere to the following configuration best practices:

Keep `web-tester.yaml` Version Controlled. Ensure your configuration file is committed to your repository. This maintains consistency across all environments (local, staging, CI) and ensures that all team members are using the same baseline settings.

Use Environment Variables for Secrets. Never hardcode passwords, API keys, database connection strings, or sensitive tokens in your configuration files. Always use environment variables and reference them in your setup scripts or CI configuration.

Implement State Preservation. Utilize the `auth.json` schema to save and reuse authentication states. This drastically reduces test execution time by bypassing the login UI for every test and significantly reduces the load on your authentication servers during test runs.

Optimize Timeouts. Do not rely on massive global timeouts to fix flaky tests. A test that takes 2 minutes to fail because of a global timeout wastes CI resources. Instead, use specific, targeted timeouts (like `element_timeout`) and implement proper wait strategies (e.g., waiting for network idle, waiting for specific element states, or polling APIs).

Leverage Suites for CI/CD. Define specific suites in `suite.json` (e.g., `smoke`, `sanity`, `regression`) and map them to different stages of your CI/CD pipeline. Run the `smoke` suite on every PR commit, and run the full `regression` suite nightly. This optimizes feedback loops and resource usage.

Regularly Review Configurations. As your application and test suite grow, periodically review your `workers`, `sharding`, and `timeout` settings. What worked for 100 tests might not be optimal for 10,000 tests. Adjust concurrency settings based on the capabilities of your CI infrastructure.

Use Percentage-Based Workers in CI. Instead of hardcoding `workers: 4`, use `workers: "50%"` or `"80%"`. This allows your configuration to scale automatically if you upgrade your CI runners to machines with more CPU cores, without requiring code changes.

Enable HTML Reporting in CI. Always configure the HTML reporter and archive the output folder as a CI artifact. When a test fails in CI, the HTML report (complete with traces and screenshots) is the most valuable tool for debugging the issue without having to reproduce it locally.

By mastering these configuration schemas and adhering to these best practices, teams can build robust, scalable, and highly performant automated testing pipelines with `web-tester-supreme`, ensuring high software quality and rapid delivery cycles.
