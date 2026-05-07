# Vitest CLI Command Reference

## Introduction

Vitest is a blazing fast unit test framework powered by Vite. It provides a comprehensive Command Line Interface (CLI) that allows developers to run tests, manage test environments, generate coverage reports, and integrate seamlessly into Continuous Integration (CI) pipelines. This document serves as an exhaustive, deep-dive reference for the Vitest CLI, covering every command, flag, argument, and providing detailed examples of usage.

Whether you are running a simple test suite, debugging complex asynchronous tests, or configuring advanced coverage reports, mastering the Vitest CLI is essential for maximizing your productivity and ensuring the reliability of your codebase.

---

## Core Commands

The Vitest CLI is invoked using the `vitest` command. By default, running `vitest` without any arguments will start Vitest in watch mode in development environments, and in run mode in CI environments.

### `vitest` (Default Command)

Starts the test runner.

**Usage:**
```bash
vitest [options] [filters...]
```

**Arguments:**
- `[filters...]`: Optional string or regular expression filters to run only specific test files. For example, `vitest math` will only run test files containing "math" in their path.

**Examples:**
```bash
# Run all tests in watch mode (default behavior in dev)
vitest

# Run only tests matching "user"
vitest user

# Run tests matching "api" or "auth"
vitest api auth
```

### `vitest run`

Runs the test suite once and exits. This is equivalent to running `vitest --run`. It is the default behavior in CI environments.

**Usage:**
```bash
vitest run [options] [filters...]
```

**Examples:**
```bash
# Run all tests once
vitest run

# Run specific tests once
vitest run components/Button.test.ts
```

### `vitest watch`

Starts the test runner in watch mode. It watches for file changes and re-runs the relevant tests automatically. This is equivalent to running `vitest --watch`.

**Usage:**
```bash
vitest watch [options] [filters...]
```

**Examples:**
```bash
# Start watch mode explicitly
vitest watch
```

### `vitest dev`

An alias for `vitest watch`.

**Usage:**
```bash
vitest dev [options] [filters...]
```

### `vitest related`

Runs tests related to a list of source files. This is particularly useful in pre-commit hooks (e.g., with lint-staged) to only run tests affected by the changed files.

**Usage:**
```bash
vitest related <files...> [options]
```

**Arguments:**
- `<files...>`: A space-separated list of source files.

**Examples:**
```bash
# Run tests related to a specific source file
vitest related src/utils/math.ts

# Run tests related to multiple files
vitest related src/components/Header.vue src/components/Footer.vue
```

### `vitest typecheck`

Runs typechecking alongside your tests. This requires TypeScript to be installed and configured.

**Usage:**
```bash
vitest typecheck [options]
```

**Examples:**
```bash
# Run typechecking
vitest typecheck

# Run typechecking in watch mode
vitest typecheck --watch
```

---

## Global Options and Flags

Vitest provides a wide array of options to customize its behavior. These options can be passed to any of the core commands.

### Execution Mode Flags

#### `--watch`, `-w`
Enable watch mode. Vitest will keep running and re-run tests when files change.
- **Default:** `true` in dev, `false` in CI.
- **Example:** `vitest --watch`

#### `--run`
Run tests once and exit. Disables watch mode.
- **Example:** `vitest --run`

#### `--ui`
Enable the Vitest UI. This starts a local web server with a beautiful graphical interface for viewing test results, coverage, and logs.
- **Example:** `vitest --ui`

#### `--api`
Start the Vitest API server. This is useful for building custom integrations or tools on top of Vitest.
- **Example:** `vitest --api`
- **Advanced:** `vitest --api.port 51204 --api.host 0.0.0.0`

### Filtering and Selection

#### `--testNamePattern <pattern>`, `-t <pattern>`
Run only tests with a name that matches the given regular expression.
- **Example:** `vitest -t "should calculate sum"`
- **Example:** `vitest --testNamePattern="^User API"`

#### `--dir <path>`
Base directory to scan for the test files.
- **Example:** `vitest --dir src/tests`

#### `--update`, `-u`
Update snapshot files. This is necessary when you intentionally change the output of a component or function that is being snapshot-tested.
- **Example:** `vitest -u`

#### `--changed`, `-c`
Run tests related to changed files based on Git (uncommitted files).
- **Example:** `vitest --changed`

#### `--changed <commit>`
Run tests related to files changed since the specified Git commit or branch.
- **Example:** `vitest --changed HEAD~1`
- **Example:** `vitest --changed main`

#### `--shard <shard>`
Test suite shard to execute in a format of `<index>/<count>`. This is extremely useful for parallelizing tests across multiple CI machines.
- **Example:** `vitest --shard 1/3` (Runs the first third of the tests)
- **Example:** `vitest --shard 2/3` (Runs the second third)

### Environment and Configuration

#### `--config <file>`, `-c <file>`
Specify a custom configuration file. By default, Vitest looks for `vitest.config.ts`, `vitest.config.js`, `vite.config.ts`, etc.
- **Example:** `vitest --config ./config/vitest.custom.ts`

#### `--environment <env>`
The test environment to use. Common options are `node`, `jsdom`, `happy-dom`, or `edge-runtime`.
- **Default:** `node`
- **Example:** `vitest --environment jsdom`

#### `--root <path>`, `-r <path>`
Root path of the project.
- **Example:** `vitest --root ./packages/core`

#### `--mode <mode>`
Override Vite mode (e.g., `development`, `production`, `test`).
- **Default:** `test`
- **Example:** `vitest --mode production`

#### `--workspace <file>`
Path to a workspace configuration file. Useful for monorepos.
- **Example:** `vitest --workspace vitest.workspace.ts`

### Output and Formatting

#### `--reporter <name>`
Specify the reporter to use for output. You can specify multiple reporters. Built-in reporters include `default`, `verbose`, `dot`, `junit`, `json`, `html`, `tap`, `hanging-process`.
- **Example:** `vitest --reporter verbose`
- **Example:** `vitest --reporter default --reporter junit`

#### `--outputFile <file>`
Write test results to a file when using reporters like `json` or `junit`. Can be a string or an object mapping reporter names to file paths.
- **Example:** `vitest --reporter json --outputFile ./results.json`

#### `--silent`
Silent console output from tests.
- **Example:** `vitest --silent`

#### `--hideSkippedTests`
Hide skipped tests from the console output.
- **Example:** `vitest --hideSkippedTests`

#### `--color`, `--no-color`
Force colorize or disable colorization of the console output.
- **Example:** `vitest --no-color`

#### `--clearScreen`
Clear terminal screen when re-running tests in watch mode.
- **Default:** `true`
- **Example:** `vitest --no-clearScreen`

### Coverage Options

Vitest supports generating coverage reports using either `v8` (default) or `istanbul`.

#### `--coverage`
Enable coverage report generation.
- **Example:** `vitest run --coverage`

#### `--coverage.provider <name>`
Select the coverage provider (`v8` or `istanbul`).
- **Default:** `v8`
- **Example:** `vitest --coverage.provider istanbul`

#### `--coverage.reporter <name>`
Specify coverage reporters (e.g., `text`, `json`, `html`, `lcov`, `clover`). Can be specified multiple times.
- **Example:** `vitest --coverage --coverage.reporter text --coverage.reporter html`

#### `--coverage.reportsDirectory <dir>`
Directory to write coverage reports to.
- **Default:** `./coverage`
- **Example:** `vitest --coverage --coverage.reportsDirectory ./reports/coverage`

#### `--coverage.exclude <pattern>`
Glob pattern to exclude files from coverage.
- **Example:** `vitest --coverage --coverage.exclude "src/**/*.test.ts"`

#### `--coverage.include <pattern>`
Glob pattern to include files in coverage.
- **Example:** `vitest --coverage --coverage.include "src/**/*.ts"`

#### `--coverage.thresholds.lines <number>`
Set the minimum threshold for line coverage. If coverage falls below this, the command will fail.
- **Example:** `vitest --coverage --coverage.thresholds.lines 80`

### Performance and Execution Control

#### `--threads`, `--no-threads`
Enable or disable multi-threading. By default, Vitest runs tests in multiple threads using worker threads. Disabling threads can be useful for debugging or if tests have global state conflicts.
- **Note:** In newer versions, this is often managed via `--pool` (e.g., `--pool threads` or `--pool forks`).
- **Example:** `vitest --no-threads`

#### `--pool <pool>`
Specify the execution pool. Options are `threads`, `forks`, or `vmThreads`.
- **Default:** `threads`
- **Example:** `vitest --pool forks`

#### `--poolMatchGlobs <globs>`
Use a specific pool for specific files.
- **Example:** `vitest --poolMatchGlobs "**/*.browser.test.ts:forks"`

#### `--isolate`, `--no-isolate`
Isolate environment for each test file. Disabling isolation can significantly speed up tests but may cause cross-test contamination.
- **Default:** `true`
- **Example:** `vitest --no-isolate`

#### `--maxConcurrency <number>`
Maximum number of concurrent tests in a suite.
- **Default:** `5`
- **Example:** `vitest --maxConcurrency 10`

#### `--maxWorkers <number|percent>`
Maximum number of workers to run tests. Can be an absolute number or a percentage of available CPUs.
- **Example:** `vitest --maxWorkers 4`
- **Example:** `vitest --maxWorkers 50%`

#### `--minWorkers <number>`
Minimum number of workers to run tests.
- **Example:** `vitest --minWorkers 2`

#### `--bail <number>`
Stop test execution after a specified number of failed tests.
- **Example:** `vitest --bail 1` (Stop on first failure)

#### `--retry <number>`
Retry failed tests a specified number of times.
- **Example:** `vitest --retry 3`

#### `--sequence.shuffle`
Run files and tests in a random order. Useful for detecting order-dependent tests.
- **Example:** `vitest --sequence.shuffle`

#### `--sequence.seed <seed>`
Set the seed for the randomizer when using `--sequence.shuffle`.
- **Example:** `vitest --sequence.shuffle --sequence.seed 12345`

### Debugging and Profiling

#### `--inspect`
Enable Node.js inspector. This allows you to attach a debugger (like Chrome DevTools or VS Code) to the Vitest process.
- **Example:** `vitest --inspect`

#### `--inspect-brk`
Enable Node.js inspector and break before the test starts.
- **Example:** `vitest --inspect-brk`

#### `--logHeapUsage`
Show the size of the heap after each test. Useful for debugging memory leaks.
- **Example:** `vitest --logHeapUsage`

#### `--dangerouslyIgnoreUnhandledErrors`
Ignore any unhandled errors that occur. Not recommended for general use, but can be useful in specific debugging scenarios.
- **Example:** `vitest --dangerouslyIgnoreUnhandledErrors`

#### `--printConsoleTrace`
Always print console traces.
- **Example:** `vitest --printConsoleTrace`

---

## Advanced Usage Scenarios

### CI/CD Integration

In a Continuous Integration environment, you typically want to run tests once, generate coverage, and output results in a format that the CI system can parse (like JUnit).

```bash
vitest run --coverage --reporter junit --outputFile ./junit.xml
```

### Monorepo Workspaces

If you are using a monorepo (e.g., with pnpm workspaces, Yarn workspaces, or Nx), Vitest has built-in workspace support. You can define a `vitest.workspace.ts` file and run tests across all packages.

```bash
# Run tests in all workspace projects
vitest --workspace vitest.workspace.ts

# Run tests only in a specific project within the workspace
vitest --project core
```

### Typechecking

Vitest can run `tsc --noEmit` or `vue-tsc --noEmit` in the background to ensure your types are correct alongside your tests.

```bash
vitest typecheck --run
```

### Browser Mode (Experimental)

Vitest has experimental support for running tests directly in the browser, providing a more accurate environment for DOM-heavy applications.

```bash
vitest --browser
```
*(Note: Browser mode requires additional configuration and dependencies like `@vitest/browser` and a browser provider like `playwright` or `webdriverio`.)*

---

## Configuration File Equivalents

Almost every CLI flag has an equivalent option in the `vitest.config.ts` file. It is generally recommended to put complex configurations in the config file and use the CLI flags for temporary overrides or CI-specific settings.

For example, running `vitest --coverage.provider istanbul --reporter verbose` is equivalent to:

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'istanbul'
    },
    reporters: ['verbose']
  }
})
```

## Conclusion

The Vitest CLI is a powerful and flexible tool that caters to a wide range of testing needs. From simple watch modes during development to complex, sharded, and coverage-instrumented runs in CI, understanding the available commands and flags allows you to tailor the testing experience to your project's exact requirements. By leveraging features like `--shard`, `--pool`, and `--ui`, teams can significantly improve their testing workflow, reduce execution times, and maintain a high standard of code quality.