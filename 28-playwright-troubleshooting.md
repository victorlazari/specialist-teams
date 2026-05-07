# Playwright Troubleshooting & Diagnostics Guide

## 1. Introduction
Playwright is a powerful framework for end-to-end testing and automation of modern web apps. However, as with any complex automation tool, users often encounter issues related to network, timing, browser contexts, and environment configurations. This comprehensive guide provides deep-dive troubleshooting strategies, diagnostic techniques, error code analysis, and health checks to ensure robust Playwright execution.

## 2. Common Error Codes and Exceptions

### 2.1. TimeoutError
**Description:** The most common error in Playwright. It occurs when an action (like `click`, `waitForSelector`, or navigation) exceeds the configured timeout.
**Root Causes:**
- The element is not present in the DOM.
- The element is present but not visible or actionable (e.g., covered by another element, disabled).
- Network latency causing the page to load slower than the timeout.
- Incorrect selector.

**Recovery Strategies:**
- **Increase Timeout:** Temporarily increase the timeout for the specific action to determine if it's a timing issue. `await page.click('.btn', { timeout: 10000 });`
- **Use Auto-Waiting:** Rely on Playwright's auto-waiting mechanisms. Ensure you are not bypassing them with raw DOM queries.
- **Check Element State:** Use `page.waitForSelector('.btn', { state: 'visible' })` to ensure the element is ready.
- **Trace Viewer:** Enable Playwright Trace Viewer to see the exact state of the DOM and network at the time of the timeout.

### 2.2. TargetClosedError
**Description:** Occurs when the browser, context, or page is closed unexpectedly while an operation is still pending.
**Root Causes:**
- The browser crashed due to out-of-memory (OOM) issues.
- The test runner forcefully closed the browser before the test finished.
- A script on the page called `window.close()`.

**Recovery Strategies:**
- **Resource Monitoring:** Monitor memory and CPU usage during test execution. If running in CI/CD, ensure the container has sufficient resources.
- **Headless Mode:** If running headed, try headless mode to reduce resource consumption.
- **Update Playwright:** Ensure you are using the latest version of Playwright, as browser crashes are often fixed in newer releases.

### 2.3. ElementHandleError: Element is not attached to the DOM
**Description:** Occurs when you have a reference to an element (ElementHandle), but the element has been removed from the DOM or the page has navigated.
**Root Causes:**
- Single Page Application (SPA) re-renders. The framework (React, Vue, Angular) destroyed the old DOM node and created a new one.
- Page navigation occurred after the element was queried.

**Recovery Strategies:**
- **Use Locators:** Always prefer Locators (`page.locator()`) over ElementHandles (`page.$()`). Locators are strict and resolve the element at the moment of the action, automatically handling re-renders.
- **Re-query:** If you must use ElementHandles, re-query the element immediately before interacting with it.

## 3. Diagnostic Tools and Techniques

### 3.1. Playwright Trace Viewer
The Trace Viewer is the most powerful diagnostic tool in Playwright. It captures a full trace of your test execution, including DOM snapshots, network requests, console logs, and action timelines.

**How to Enable:**
```javascript
import { test } from '@playwright/test';

test.use({ trace: 'on-first-retry' });
```
Or via command line: `npx playwright test --trace on`

**How to Analyze:**
- Open the trace file using `npx playwright show-trace trace.zip`.
- Inspect the "Actions" tab to see exactly what Playwright was trying to do.
- Use the "Network" tab to identify failed or slow requests.
- View the DOM snapshot at the exact moment of failure to verify if the element was present and visible.

### 3.2. Playwright Inspector
The Inspector is a GUI tool that allows you to step through your tests, inspect the DOM, and evaluate selectors in real-time.

**How to Enable:**
Run your tests with the `PWDEBUG=1` environment variable.
`PWDEBUG=1 npx playwright test`

**Features:**
- **Step-by-step execution:** Pause, resume, and step over actions.
- **Selector Playground:** Test and refine your selectors directly in the browser.
- **Action Logs:** View detailed logs of each Playwright action.

### 3.3. Verbose Logging
Playwright provides detailed debug logs that can help identify low-level issues, such as protocol errors or browser launch failures.

**How to Enable:**
Set the `DEBUG` environment variable.
`DEBUG=pw:api npx playwright test` (Logs API calls)
`DEBUG=pw:browser npx playwright test` (Logs browser interactions)

## 4. Health Checks and Environment Validation

### 4.1. Browser Installation Check
Ensure that the required browsers are installed and compatible with your Playwright version.
**Command:** `npx playwright install --with-deps`
This command installs the browsers and any missing system dependencies (especially useful in Linux/CI environments).

### 4.2. Network Connectivity Check
Playwright tests often fail due to network issues, especially when accessing internal environments or third-party APIs.
- **Proxy Configuration:** Ensure Playwright is configured to use the correct proxy settings if your network requires it.
- **Bypass CSP:** Sometimes Content Security Policy (CSP) blocks scripts or resources. You can bypass it for testing purposes: `await page.route('**/*', route => route.continue());` (Use with caution).

### 4.3. CI/CD Environment Validation
Running Playwright in CI/CD (GitHub Actions, GitLab CI, Jenkins) introduces specific challenges.
- **Headless Mode:** Ensure tests run in headless mode (`headless: true`).
- **XVFB:** If you must run headed tests on Linux, ensure Xvfb is installed and configured.
- **Resource Limits:** CI runners often have limited CPU and memory. Use `workers: 1` or a lower number of workers to prevent resource exhaustion.

## 5. Advanced Troubleshooting Scenarios

### 5.1. Flaky Tests
Flaky tests pass sometimes and fail other times without any code changes. They are the bane of automation.
**Causes:**
- Race conditions (interacting with elements before they are fully ready).
- Network instability.
- Unpredictable animations or transitions.

**Solutions:**
- **Strict Locators:** Ensure your locators uniquely identify a single element.
- **Wait for State:** Use `waitForLoadState('networkidle')` or wait for specific API responses instead of arbitrary `page.waitForTimeout()`.
- **Disable Animations:** Disable CSS animations and transitions during testing to ensure consistent timing.
- **Retries:** Configure Playwright to retry failed tests automatically. `test.use({ retries: 2 });`

### 5.2. Authentication Issues
Testing authenticated flows can be tricky, especially with multi-factor authentication (MFA) or complex login sequences.
**Solutions:**
- **Reuse Authentication State:** Log in once, save the storage state (cookies, localStorage), and reuse it across tests.
```javascript
// Save state
await page.context().storageState({ path: 'state.json' });

// Load state
const context = await browser.newContext({ storageState: 'state.json' });
```
- **API Login:** Bypass the UI login entirely by making an API request to authenticate and setting the cookies directly in the browser context.

### 5.3. Handling Iframes
Interacting with elements inside iframes requires switching the context to the iframe.
**Solutions:**
- Use `page.frameLocator()` to target the iframe and then query elements inside it.
```javascript
const frame = page.frameLocator('#my-iframe');
await frame.locator('.btn').click();
```
- Ensure the iframe has fully loaded before interacting with it.

## 6. Performance Tuning and Optimization

### 6.1. Parallel Execution
Playwright supports running tests in parallel across multiple workers.
- **Configuration:** Set the number of workers in `playwright.config.ts`.
- **Isolation:** Ensure tests are fully isolated and do not share state (e.g., database records) to prevent conflicts during parallel execution.

### 6.2. Resource Blocking
Block unnecessary resources (images, fonts, analytics scripts) to speed up page load times and reduce flakiness.
```javascript
await page.route('**/*.{png,jpg,jpeg,woff,woff2}', route => route.abort());
```

### 6.3. API Mocking
Mock external APIs to isolate your application and ensure tests run quickly and reliably, regardless of the external service's status.
```javascript
await page.route('**/api/data', route => {
  route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({ key: 'value' })
  });
});
```

## 7. Conclusion
Troubleshooting Playwright requires a systematic approach, leveraging the built-in diagnostic tools like Trace Viewer and Inspector. By understanding common error codes, implementing robust health checks, and following best practices for locators and waiting mechanisms, you can build a highly reliable and maintainable end-to-end testing suite. Always prioritize auto-waiting and strict locators to minimize flakiness and ensure consistent test execution across different environments.

## 8. Deep Dive into Specific Subsystems

### 8.1. Network Interception and Mocking
Playwright allows you to intercept network requests and mock responses. This is incredibly useful for testing edge cases, simulating server errors, or speeding up tests by avoiding real network calls.

**Common Issues:**
- **Unmatched Routes:** If a route is not matched by your interception logic, it will proceed normally. Ensure your URL patterns are correct.
- **Timing Issues:** If you set up interception after the request has already been made, it won't be intercepted. Always set up routes before navigating or triggering the action that causes the request.

**Troubleshooting Steps:**
- Use `page.on('request', request => console.log(request.url()))` to log all requests and verify the URL you are trying to intercept.
- Check the Network tab in the Trace Viewer to see if the request was intercepted and what response was returned.

### 8.2. File Uploads and Downloads
Handling file uploads and downloads can be tricky in automated tests.

**Upload Issues:**
- Ensure the input element is of type `file`.
- Use `page.setInputFiles('input[type="file"]', 'path/to/file.txt')`.
- If the upload is triggered by a custom UI component, you might need to interact with the underlying hidden input element.

**Download Issues:**
- Use the `page.waitForEvent('download')` to capture the download object.
- Ensure the download completes before trying to access the file.
- Check the file path and permissions if you are saving the file to disk.

### 8.3. Mobile Emulation
Playwright supports mobile emulation, allowing you to test your application on different devices and screen sizes.

**Common Issues:**
- **Viewport Size:** Ensure the viewport size matches the target device.
- **User Agent:** Some applications rely on the User-Agent string to serve mobile-specific content. Ensure it is set correctly.
- **Touch Events:** Emulate touch events if your application relies on them.

**Troubleshooting Steps:**
- Use the predefined devices in Playwright: `const { devices } = require('@playwright/test'); const iPhone = devices['iPhone 12'];`
- Verify the layout and functionality in the Trace Viewer or by taking screenshots.

## 9. Integrating with Third-Party Tools

### 9.1. Allure Reporting
Allure is a popular reporting tool that integrates well with Playwright.

**Common Issues:**
- **Missing Results:** Ensure the Allure reporter is configured correctly in `playwright.config.ts`.
- **Incomplete Data:** Make sure you are attaching screenshots, traces, and other artifacts to the Allure report.

**Troubleshooting Steps:**
- Check the console output for any errors related to the Allure reporter.
- Verify the generated XML files in the `allure-results` directory.

### 9.2. Docker and Containerization
Running Playwright in Docker containers is common for CI/CD pipelines.

**Common Issues:**
- **Missing Dependencies:** Ensure the Docker image has all the required system dependencies for the browsers.
- **IPC and SHM:** Browsers require sufficient shared memory (SHM) and inter-process communication (IPC) resources. Use `--ipc=host` or increase the SHM size when running the container.

**Troubleshooting Steps:**
- Use the official Playwright Docker image: `mcr.microsoft.com/playwright:v1.39.0-jammy`.
- Check the container logs for any browser crash or resource exhaustion errors.

## 10. Best Practices for Maintainable Tests

### 10.1. Page Object Model (POM)
The Page Object Model is a design pattern that helps organize your test code and make it more maintainable.

**Benefits:**
- **Reusability:** Encapsulate page-specific logic and locators in a single class.
- **Readability:** Make your tests easier to read and understand.
- **Maintainability:** Update locators and actions in one place when the UI changes.

**Implementation:**
- Create a class for each page or component in your application.
- Define locators and actions as methods in the class.
- Instantiate the class in your tests and call the methods.

### 10.2. Data-Driven Testing
Data-driven testing allows you to run the same test with different sets of data.

**Benefits:**
- **Coverage:** Test multiple scenarios with minimal code duplication.
- **Flexibility:** Easily add or remove test cases by updating the data source.

**Implementation:**
- Use an array of objects or a CSV file to store the test data.
- Iterate over the data and generate test cases dynamically using `test.describe` and `test`.

### 10.3. Continuous Integration and Delivery (CI/CD)
Integrating Playwright into your CI/CD pipeline ensures that your tests run automatically on every code change.

**Benefits:**
- **Early Feedback:** Catch bugs early in the development cycle.
- **Confidence:** Ensure that new features do not break existing functionality.

**Implementation:**
- Configure your CI/CD tool (e.g., GitHub Actions, GitLab CI) to run the Playwright tests.
- Set up artifacts to save test reports, traces, and screenshots for failed tests.
- Use caching to speed up the installation of dependencies and browsers.

## 11. Advanced Debugging Techniques

### 11.1. Using the Node.js Debugger
You can use the built-in Node.js debugger to step through your Playwright test code.

**How to Enable:**
- Run your tests with the `--inspect` or `--inspect-brk` flag.
- Open Chrome and navigate to `chrome://inspect`.
- Click on "Open dedicated DevTools for Node" to attach the debugger.

**Benefits:**
- **Breakpoints:** Set breakpoints in your test code to pause execution.
- **Variable Inspection:** Inspect the values of variables and objects.
- **Call Stack:** View the call stack to understand the execution flow.

### 11.2. Custom Loggers and Reporters
You can create custom loggers and reporters to capture specific information during test execution.

**Implementation:**
- Implement the `Reporter` interface provided by Playwright.
- Override methods like `onTestBegin`, `onTestEnd`, and `onError` to log custom messages or send data to external systems.

**Benefits:**
- **Tailored Output:** Customize the output format to match your team's preferences.
- **Integration:** Send test results to custom dashboards or alerting systems.

## 12. Handling Complex UI Interactions

### 12.1. Drag and Drop
Playwright provides a built-in `dragTo` method for simple drag and drop interactions.

**Implementation:**
```javascript
await page.locator('#source').dragTo(page.locator('#target'));
```

**Troubleshooting:**
- If the built-in method doesn't work, you might need to simulate the mouse events manually using `page.mouse.down()`, `page.mouse.move()`, and `page.mouse.up()`.

### 12.2. Keyboard Shortcuts
You can simulate keyboard shortcuts using the `page.keyboard.press` method.

**Implementation:**
```javascript
await page.keyboard.press('Control+A');
await page.keyboard.press('Delete');
```

**Troubleshooting:**
- Ensure the correct element has focus before pressing the keys.
- Use the correct key names as defined in the Playwright documentation.

### 12.3. Hover and Tooltips
Interacting with elements that appear on hover can be challenging.

**Implementation:**
```javascript
await page.locator('#hover-target').hover();
await page.locator('#tooltip').waitFor({ state: 'visible' });
```

**Troubleshooting:**
- Ensure the hover action triggers the expected behavior.
- Use the Trace Viewer to verify the state of the DOM after the hover action.

## 13. Security and Privacy Considerations

### 13.1. Handling Sensitive Data
Avoid hardcoding sensitive data (e.g., passwords, API keys) in your test code.

**Implementation:**
- Use environment variables to store sensitive data.
- Access the variables in your tests using `process.env`.

### 13.2. Bypassing Captchas and Anti-Bot Measures
Playwright is an automation tool and can be blocked by captchas and anti-bot measures.

**Troubleshooting:**
- Use services that provide captcha solving capabilities.
- Configure Playwright to mimic human behavior (e.g., random delays, realistic mouse movements).
- Note that bypassing these measures might violate the terms of service of the target application.

## 14. Final Thoughts
Mastering Playwright troubleshooting requires a combination of understanding the framework's internals, utilizing the available diagnostic tools, and applying best practices for test design. By continuously learning and adapting to new challenges, you can build a robust and reliable automation suite that provides valuable feedback and confidence in your application's quality.