# Vitest Configuration Schemas Guide

Vitest is a fast and lightweight test runner designed for modern web development. Its configuration system is designed to be both flexible and powerful, allowing developers to tailor it to their specific needs. This guide provides an exhaustive look at the configuration options available in Vitest. We'll cover every field, its purpose, default values, and recommended best practices to help you optimize your testing setup.

## Configuration File

Vitest configuration can be defined in several ways, typically through a configuration file. Common file names include `vitest.config.js`, `vitest.config.ts`, or a `vite.config.js` with a `test` field. The configuration is defined as an exported object containing various fields.

### Example Configuration

```javascript
// vitest.config.js
export default {
  test: {
    // Configuration fields go here
  }
}
```

## Configuration Fields

### 1. `include`

- **Type**: `string[]`
- **Default**: `['**/*.test.{js,ts}']`
- **Description**: A list of glob patterns specifying which files to include as test files.
- **Best Practices**: 
  - Use specific patterns to target only your test files and avoid unnecessary file processing.
  - Consider including patterns like `['**/*.spec.{js,ts}']` if you follow a spec naming convention.

### 2. `exclude`

- **Type**: `string[]`
- **Default**: `['node_modules', '**/dist/**']`
- **Description**: A list of glob patterns for files and directories to exclude from testing.
- **Best Practices**: 
  - Keep the default exclusions to avoid processing compiled files or dependencies.
  - Add additional exclusions for generated files or build artifacts specific to your project.

### 3. `globals`

- **Type**: `boolean`
- **Default**: `false`
- **Description**: If `true`, makes Jest-style global test functions like `describe` and `it` available.
- **Best Practices**: 
  - Enable for projects migrating from Jest or if you prefer having global test functions without importing them in each file.

### 4. `testTimeout`

- **Type**: `number`
- **Default**: `5000` (milliseconds)
- **Description**: Specifies the default timeout for each test in milliseconds.
- **Best Practices**: 
  - Adjust the timeout based on the nature of your tests; longer timeouts might be necessary for integration tests.

### 5. `hookTimeout`

- **Type**: `number`
- **Default**: `10000` (milliseconds)
- **Description**: Specifies the timeout for setup and teardown hooks.
- **Best Practices**: 
  - Increase if your setup or teardown processes are complex or involve external services.

### 6. `reporters`

- **Type**: `string[] | Reporter[]`
- **Default**: `['default']`
- **Description**: Specifies the reporters to use, determining how test results are displayed.
- **Best Practices**: 
  - Use additional reporters like `['json', 'verbose']` for detailed output or CI integration.
  - Custom reporters can be defined to tailor output to specific requirements.

### 7. `coverage`

- **Type**: `CoverageOptions`
- **Default**: `{ enabled: false }`
- **Description**: Configuration options for code coverage reporting.
- **Fields**:
  - `enabled`: `boolean` - Enable or disable coverage collection.
  - `reportsDirectory`: `string` - Directory where coverage reports are stored.
  - `reporters`: `string[]` - Coverage reporters to use, e.g., `['text', 'html']`.
  - `include`: `string[]` - Files to include for coverage.
  - `exclude`: `string[]` - Files to exclude from coverage.
- **Best Practices**: 
  - Enable only if you need coverage reports as it impacts performance.
  - Customize the `include` and `exclude` fields to get accurate coverage metrics.

### 8. `snapshot`

- **Type**: `SnapshotOptions`
- **Default**: `{ dir: '__snapshots__', update: false }`
- **Description**: Configuration related to snapshot testing.
- **Fields**:
  - `dir`: `string` - Directory where snapshots are stored.
  - `update`: `boolean` - Whether to update snapshots automatically.
- **Best Practices**: 
  - Keep snapshots in a separate directory for better organization.
  - Use the `update` option cautiously to avoid unintentional snapshot overwrites.

### 9. `maxConcurrency`

- **Type**: `number`
- **Default**: Number of logical CPUs
- **Description**: Maximum number of test files to run concurrently.
- **Best Practices**: 
  - Adjust based on your machine's capabilities to optimize test execution speed without overloading the system.

### 10. `isolation`

- **Type**: `boolean`
- **Default**: `true`
- **Description**: Determines if each test file should run in its own process.
- **Best Practices**: 
  - Enable isolation (default) for consistent test results and to prevent state leakage between tests.

### 11. `setupFiles`

- **Type**: `string[]`
- **Default**: `[]`
- **Description**: List of files to be loaded before the test suite is executed.
- **Best Practices**: 
  - Use for global setup needed across multiple test files, like setting up a test database or mocking global objects.

### 12. `deps`

- **Type**: `DependenciesOptions`
- **Default**: `{ inline: false, external: [] }`
- **Description**: Options for handling dependencies.
- **Fields**:
  - `inline`: `boolean` - Whether to bundle dependencies inline.
  - `external`: `string[]` - List of dependencies to exclude from bundling.
- **Best Practices**: 
  - Use `external` to prevent bundling large dependencies that are better loaded separately.
  - Inline only small or frequently used dependencies to reduce load times.

### 13. `environment`

- **Type**: `string`
- **Default**: `'node'`
- **Description**: The environment in which tests are executed. Options include `'node'`, `'jsdom'`, etc.
- **Best Practices**: 
  - Use `'node'` for server-side tests and `'jsdom'` for tests that require a browser-like environment.

### 14. `transform`

- **Type**: `Record<string, TransformOptions>`
- **Default**: `{}`
- **Description**: Define custom transformations for files before running tests.
- **Best Practices**: 
  - Use for transforming non-standard file types or applying custom preprocessing like transpiling.

### 15. `silent`

- **Type**: `boolean`
- **Default**: `false`
- **Description**: If `true`, suppresses console output during test execution.
- **Best Practices**: 
  - Enable for cleaner output in CI/CD pipelines where logs are captured separately.

### 16. `clearMocks`

- **Type**: `boolean`
- **Default**: `true`
- **Description**: Automatically clear mock calls and instances between every test.
- **Best Practices**: 
  - Keep enabled for ensuring test isolation and avoiding state bleed between tests.

### 17. `restoreMocks`

- **Type**: `boolean`
- **Default**: `false`
- **Description**: Automatically restore mocked functions to their original implementations.
- **Best Practices**: 
  - Enable if you need to ensure mocks are cleared between tests but want to retain original implementations.

### 18. `resetModules`

- **Type**: `boolean`
- **Default**: `false`
- **Description**: If `true`, resets the module registry before running each test.
- **Best Practices**: 
  - Useful when tests modify module state and complete isolation is required.

### 19. `logHeapUsage`

- **Type**: `boolean`
- **Default**: `false`
- **Description**: Logs memory usage after each test.
- **Best Practices**: 
  - Enable for debugging memory leaks or performance bottlenecks during test execution.

## Conclusion

The configuration of Vitest is versatile and allows you to tailor the test runner to your project's needs. By understanding and leveraging each configuration option, you can ensure efficient and effective testing workflows. Remember to adjust configurations based on your specific requirements and testing environment to achieve optimal results.