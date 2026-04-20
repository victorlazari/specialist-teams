# 28 - Playwright E2E Advanced

## Introduction

This comprehensive guide explores advanced Playwright End-to-End (E2E) testing techniques tailored for modern web applications developed using Next.js 16.2.2 (App Router), React 19.2.4, Tailwind CSS v4, and shadcn/ui. The documentation assumes familiarity with core Playwright concepts and expands upon sophisticated features such as network interception and mocking, visual regression testing, multi-browser matrix execution, mobile emulation, continuous integration with GitHub Actions, and mitigating flaky tests. This guide also considers your full tech stack, including Prisma, PostgreSQL, Redis (via Valkey 8), RabbitMQ, JWT/Argon2id authentication, SeaweedFS storage, and observability tools like OpenTelemetry and Grafana.

---

## Table of Contents

| Section | Topic |
| --- | --- |
| 1 | Network Interception and API Mocking |
| 2 | Visual Regression Testing with Playwright |
| 3 | Multi-Browser Matrix Testing Strategy |
| 4 | Mobile Emulation and Responsive Testing |
| 5 | CI/CD Integration with GitHub Actions |
| 6 | Detecting and Handling Flaky Tests |
| 7 | Architectural Considerations and Best Practices |

---

## 1. Network Interception and API Mocking

Network interception is essential for isolating E2E tests from volatile backend dependencies, facilitating deterministic test outcomes, and simulating edge cases like server failures or latency. Playwright’s `page.route()` API allows you to intercept, modify, mock, or abort requests matching specified URL patterns.

### Architectural Rationale

Given your backend’s RESTful API versioning under `/api/v1/*`, intercepting API calls allows fine-grained control over test conditions without requiring backend state manipulation or test data seeding. This approach aligns well with the hexagonal architecture and ports/adapters pattern, where external dependencies can be abstracted or stubbed in tests.

### Practical Implementation

Below is an example of mocking a paginated list endpoint with dynamic query parameters, which is common in multi-tenant isolation scenarios using `customer_uuid` scoping:

```typescript
import { test, expect, Page } from '@playwright/test';

const mockApiResponse = {
  data: [
    { id: '1', name: 'Tenant A', customer_uuid: 'uuid-123' },
    { id: '2', name: 'Tenant B', customer_uuid: 'uuid-123' },
  ],
  meta: { page: 1, size: 10, total: 2 }
};

test.beforeEach(async ({ page }) => {
  await page.route('**/api/v1/tenants?*', async (route) => {
    // Extract query parameters for validation or conditional mocking
    const url = new URL(route.request().url());
    const customerUUID = url.searchParams.get('customer_uuid');

    if (customerUUID === 'uuid-123') {
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify(mockApiResponse),
      });
    } else {
      await route.continue(); // Let actual network request proceed
    }
  });
});

test('should display tenant list with mocked API', async ({ page }) => {
  await page.goto('/tenants');
  const tenantNames = await page.getByRole('list').locator('li').allTextContents();
  expect(tenantNames).toContain('Tenant A');
  expect(tenantNames).toContain('Tenant B');
});
```

### Network Failure Simulation

Simulating network failures or slow responses is critical for verifying graceful degradation and error boundary handling. Playwright supports aborting requests or introducing artificial delays:

```typescript
await page.route('**/api/v1/tenants', async (route) => {
  // Simulate network failure with 500 status
  await route.fulfill({
    status: 500,
    contentType: 'application/json',
    body: JSON.stringify({ error: 'Internal Server Error' }),
  });
});
```

Or to simulate network latency:

```typescript
await page.route('**/api/v1/tenants', async (route) => {
  await new Promise(r => setTimeout(r, 3000)); // 3 seconds delay
  await route.continue();
});
```

### HAR Recording and Replay

For deterministic testing, especially when integrating with backend services and third-party APIs, Playwright’s HAR (HTTP Archive) recording feature can be utilized to capture real network interactions and replay them during tests:

```typescript
import { chromium } from '@playwright/test';

(async () => {
  const browser = await chromium.launch();
  const context = await browser.newContext({ recordHar: { path: 'test.har' } });
  const page = await context.newPage();
  await page.goto('https://yourapp.example.com');
  // Perform interactions
  await browser.close();
})();
```

The HAR file can later serve to replay requests, avoiding network variability.

---

## 2. Visual Regression Testing with Playwright

Visual regression testing ensures UI changes do not unintentionally break the design or user experience. Playwright’s built-in screenshot comparison API `expect(page).toHaveScreenshot()` performs pixel-level diffing with configurable tolerance.

### Snapshot Testing Architecture

Snapshots are stored in a directory such as `test-results/`, enabling version control and incremental updates. Playwright supports updating snapshots with the `--update-snapshots` CLI flag.

To integrate visual tests into your Tailwind + shadcn/ui stack, consider capturing components or pages under different states (e.g., light/dark mode, loading, error).

### Example: Full Page and Component Screenshot

```typescript
test('homepage visual regression', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png', { fullPage: true, maxDiffPixels: 50 });
});

test('login button snapshot', async ({ page }) => {
  await page.goto('/login');
  const loginButton = page.getByRole('button', { name: 'Login' });
  await expect(loginButton).toHaveScreenshot('login-button.png', { maxDiffPixels: 10 });
});
```

### Handling Dynamic Content

For UI elements with dynamic content, such as timestamps or user avatars, use Playwright’s `page.locator().evaluate()` to mask or remove volatile DOM nodes before taking screenshots:

```typescript
await page.evaluate(() => {
  document.querySelectorAll('.timestamp').forEach(el => el.textContent = 'TIME_STAMP');
});
```

This reduces false positives in visual diffs.

---

## 3. Multi-Browser Matrix Testing Strategy

Cross-browser compatibility is critical, especially for public-facing sites with multifaceted user interaction (keyboard, mouse, touch). Playwright natively supports Chromium, Firefox, and WebKit.

### Configuration in `playwright.config.ts`

Define projects to run tests across browsers and devices:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  retries: 2,
  workers: 4,
  reporter: [['html'], ['list']],
});
```

Running your suite with this config ensures coverage across all major engines.

### Matrix Execution Considerations

Executing tests in parallel workers maximizes CI efficiency. However, parallelism must be balanced against test isolation requirements, especially when tests share global state, such as session cookies or local storage.

To mitigate cross-test pollution, Playwright allows creating isolated contexts per test:

```typescript
test.use({ storageState: undefined }); // clear storage state per test
```

Or use test fixtures to create separate authenticated sessions.

---

## 4. Mobile Emulation and Responsive Testing

Mobile-first design is standard with Tailwind CSS and shadcn/ui primitives supporting responsive utilities and compound components. Playwright’s device emulations allow testing on virtual devices with appropriate viewport, user agent, and touch capabilities.

### Emulating Popular Devices

Playwright ships with predefined devices for iPhone 13, Pixel 5, and more. Example configuration for mobile tests:

```typescript
import { devices, defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'iPhone 13',
      use: { ...devices['iPhone 13'] },
    },
    {
      name: 'Pixel 5',
      use: { ...devices['Pixel 5'] },
    },
  ],
});
```

### Testing Responsive Layouts

You can programmatically adjust viewport sizes to test breakpoints defined in Tailwind CSS v4. For example:

```typescript
test('renders correctly on tablet breakpoint', async ({ page }) => {
  await page.setViewportSize({ width: 768, height: 1024 }); // Tailwind md breakpoint
  await page.goto('/dashboard');
  await expect(page.getByTestId('sidebar')).toBeVisible();
});
```

### Dark Mode Emulation

Use `page.emulateMedia()` to test dark and light mode variants:

```typescript
test('dark mode rendering', async ({ page }) => {
  await page.emulateMedia({ colorScheme: 'dark' });
  await page.goto('/');
  await expect(page.locator('body')).toHaveCSS('background-color', 'rgb(17, 24, 39)'); // Tailwind slate-900
});
```

---

## 5. CI/CD Integration with GitHub Actions

Automating Playwright tests within GitHub Actions ensures continuous quality verification on every push or pull request.

### Workflow Example: `playwright-e2e.yml`

```yaml
name: Playwright E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install

      - name: Run Playwright Tests on ${{ matrix.browser }}
        run: npx playwright test --project=${{ matrix.browser }} --reporter=html
        env:
          NEXT_PUBLIC_API_URL: ${{ secrets.API_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}

      - name: Upload Playwright Report
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report-${{ matrix.browser }}
          path: playwright-report/
```

### Parallelization and Retry Strategy

The matrix strategy runs tests across multiple browsers in parallel. Including retries in `playwright.config.ts` handles transient failures:

```typescript
retries: process.env.CI ? 2 : 0,
```

Retries are enabled only within CI environments to avoid masking local issues.

### Artifact Management

Uploading the HTML report as an artifact allows developers to inspect test failures directly from GitHub’s UI, improving debugging efficiency.

---

## 6. Detecting and Handling Flaky Tests

Flaky tests produce inconsistent results, undermining confidence in the test suite and slowing development.

### Common Causes in Playwright Context

- Race conditions due to improper waiting for asynchronous UI states.
- Network instability or asynchronous API responses.
- Shared state pollution between tests.
- Dynamic content without deterministic selectors.

### Mitigation Techniques

Playwright’s auto-waiting locators (e.g., `page.getByRole()`) mitigate many timing issues. Avoid brittle selectors like xpath or class-based queries unless necessary.

Implement explicit checks for network idleness or UI stability:

```typescript
await page.waitForResponse('**/api/v1/tenants');
await expect(page.getByRole('list')).toBeVisible();
```

Use `expect.soft()` assertions to collect multiple failures without aborting early:

```typescript
await expect.soft(page.getByText('Welcome')).toBeVisible();
await expect.soft(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
```

Run flaky tests multiple times locally to verify stability:

```bash
npx playwright test --repeat-each 5
```

### Flaky Test Detection Table

| Flake Symptom | Likely Cause | Recommended Action |
| ------------- | ------------ | ------------------ |
| Timeout errors on element visibility | Missing waits, async UI updates | Use Playwright auto-waiting locators, explicit waits |
| Tests failing only on CI | Network instability, environment differences | Use retries, mock network responses, isolate test data |
| Inconsistent API response failures | Backend instability | Mock APIs, HAR replay |
| Shared session or localStorage pollution | Tests not isolated | Use separate contexts or storage state per test |

---

## 7. Architectural Considerations and Best Practices

### Test Isolation and Data Management

Each E2E test should be independent and idempotent. This is critical when dealing with a multi-tenant database with read/write replica splits, to avoid stale data or race conditions. Use API mocking or dedicated test tenants scoped by `customer_uuid` for isolation.

Storing authentication states in Playwright’s storage state files reduces redundant login steps:

```typescript
test.beforeAll(async ({ browser }) => {
  const context = await browser.newContext();
  const page = await context.newPage();
  await page.goto('/login');
  // Perform login steps
  await context.storageState({ path: 'auth.json' });
});
```

Reuse the saved state in tests:

```typescript
test.use({ storageState: 'auth.json' });
```

### Locators and Assertions

Consistently use accessible selectors (`getByRole`, `getByLabel`, `getByTestId`) to align test selectors with user-facing elements. This enhances maintainability and resilience against UI refactoring.

Prefer web-first assertions like:

```typescript
await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
```

Avoid brittle manual assertions that require explicit waiting.

### Observability and Logging

Integrate Playwright with your logging and observability stack (Pino + OpenTelemetry). Inject trace context or emit logs during critical test phases to correlate frontend test events with backend traces or metrics.

### Test Suite Structure

Organize tests according to Next.js’s app router modularization, mirroring `src/app/(protected)/`, `src/app/(public)/`, and API versioning directories. This supports maintainable test suites and easier debugging.

---

## Conclusion

Leveraging Playwright’s advanced capabilities in your Next.js 16 and React 19 environment enables robust, maintainable, and scalable E2E testing. Network interception and API mocking provide deterministic backend isolation, while visual regression ensures UI fidelity amidst rapid UI iteration with Tailwind v4 and shadcn/ui. Multi-browser matrix and mobile emulation validate cross-platform compatibility crucial for user experience. Integration with GitHub Actions streamlines continuous quality assurance, and diligent handling of flaky tests preserves developer trust in automation.

Adhering to Playwright’s best practices combined with your sophisticated architecture—multi-tenant isolation, hexagonal design, OpenTelemetry observability—will yield a resilient E2E testing framework that accelerates delivery without sacrificing quality.

---

## Appendix: Useful Snippets

**Mock Dynamic API with pagination and query parameters**

```typescript
await page.route('**/api/v1/items?*', async (route) => {
  const url = new URL(route.request().url());
  const pageParam = url.searchParams.get('page') || '1';
  const sizeParam = url.searchParams.get('size') || '10';

  const items = generateItems(parseInt(pageParam), parseInt(sizeParam));
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({ data: items, meta: { page: pageParam, size: sizeParam } }),
  });
});
```

**Update Visual Snapshots**

```bash
npx playwright test --update-snapshots
```

**Run Tests Repeatedly to Detect Flakes**

```bash
npx playwright test --repeat-each 5
```

**Save and Reuse Auth State**

```typescript
// Save
await context.storageState({ path: 'auth.json' });

// Reuse
test.use({ storageState: 'auth.json' });
```

**Emulate Mobile Device**

```typescript
test.use({ ...devices['iPhone 13'] });
```

---

This document should serve as a cornerstone for your advanced Playwright E2E testing strategy, combining technical depth and practical guidance aligned to your cutting-edge full-stack environment.