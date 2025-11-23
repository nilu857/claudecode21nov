# Playwright Automation Testing - Interviewer's Guide

## How to Use This Guide
- Each question includes what it reveals about the candidate
- Expected responses are categorized by skill level: Junior, Mid-level, Senior
- Red flags and green flags are highlighted
- Follow-up questions help you dig deeper

---

## Table of Contents
1. [Foundation & Basics](#foundation--basics)
2. [Practical Implementation](#practical-implementation)
3. [Problem-Solving & Debugging](#problem-solving--debugging)
4. [Architecture & Design](#architecture--design)
5. [Advanced Concepts](#advanced-concepts)
6. [Process & Collaboration](#process--collaboration)
7. [Behavioral & Scenario-Based](#behavioral--scenario-based)
8. [Coding Exercises](#coding-exercises)

---

## Foundation & Basics

### Q1: "Walk me through how you would set up a Playwright project from scratch."

**What This Reveals:**
- Basic understanding of project setup
- Knowledge of dependencies and configuration
- Awareness of best practices from the start
- Experience with real projects

**Expected Responses:**

**Junior Level:**
```
"I would run npm init playwright@latest, which creates the basic structure.
Then I'd install dependencies and write some tests."
```
✅ **Green Flags:** Knows the quick start command
🚩 **Red Flags:** Doesn't know how to initialize, suggests manual setup

**Mid-Level:**
```
"I'd start with npm init playwright@latest, then configure playwright.config.ts
for multiple browsers, set up parallel workers, configure base URL, and organize
the folder structure with separate directories for tests, pages, and fixtures.
I'd also set up TypeScript properly and add scripts to package.json."
```
✅ **Green Flags:** Mentions configuration, structure, TypeScript
🚩 **Red Flags:** Doesn't mention configuration or organization

**Senior Level:**
```
"I'd initialize with npm init playwright@latest, then customize playwright.config.ts
with multiple projects for different browsers and environments. I'd set up:
- Environment-specific configs (dev, staging, prod)
- Custom reporter configuration
- Global setup/teardown for test data
- Authentication state management
- CI/CD integration from the start
- Pre-commit hooks for running tests
- Folder structure following POM pattern
- Shared fixtures and utilities
- ESLint and Prettier for code quality"
```
✅ **Green Flags:** Mentions CI/CD, global setup, authentication, code quality
🚩 **Red Flags:** Doesn't think about scalability or team collaboration

**Follow-up Questions:**
- "Why would you use TypeScript over JavaScript?"
- "How would you structure tests for a team of 10 engineers?"
- "What would you include in your global setup?"

---

### Q2: "Explain the difference between page.locator(), page.$(), and page.getByRole(). When would you use each?"

**What This Reveals:**
- Understanding of Playwright's selector APIs
- Knowledge of modern vs legacy APIs
- Awareness of accessibility best practices

**Expected Responses:**

**Junior Level:**
```
"page.locator() is the new way to find elements, page.$() is the old way.
I usually use page.locator()."
```
✅ **Green Flags:** Knows locator is preferred
🚩 **Red Flags:** Can't explain why or doesn't know page.getByRole()

**Mid-Level:**
```
"page.locator() is the modern API with auto-waiting and strict mode.
page.$() is legacy from Puppeteer, returns ElementHandle or null, no auto-waiting.
page.getByRole() is accessibility-focused, finding elements by ARIA roles.

I use getByRole() when possible for accessibility, then locator() with
data-testid, and avoid $() entirely."
```
✅ **Green Flags:** Explains auto-waiting, strict mode, accessibility preference
🚩 **Red Flags:** Doesn't know when to use each

**Senior Level:**
```
"These represent Playwright's API evolution:

1. page.$() - Legacy Puppeteer API
   - Returns ElementHandle or null
   - No auto-waiting
   - Can cause flaky tests
   - Only use when absolutely needed for low-level operations

2. page.locator() - Modern API
   - Auto-waiting for actionability
   - Strict mode by default (fails if multiple matches)
   - Lazy evaluation
   - Chainable
   - Better for reliability

3. page.getByRole() - Accessibility-first
   - Encourages accessible markup
   - Self-documenting tests
   - More resilient to implementation changes
   - Tests how users actually interact

My priority: getByRole() > getByLabel() > getByTestId() > locator() > never $()

This ensures tests are accessible, maintainable, and resilient."
```
✅ **Green Flags:** Explains philosophy, provides clear hierarchy, mentions strict mode
🚩 **Red Flags:** Still advocates for using $() regularly

**Follow-up Questions:**
- "Give me an example where strict mode would fail and how you'd fix it."
- "How would you convince a team to migrate from $() to locator()?"

---

### Q3: "How does Playwright's auto-waiting work? What conditions does it check?"

**What This Reveals:**
- Deep understanding of core Playwright features
- Ability to troubleshoot timing issues
- Knowledge of edge cases

**Expected Responses:**

**Junior Level:**
```
"Playwright waits for elements to be ready before interacting with them,
so we don't need to add manual waits."
```
✅ **Green Flags:** Knows auto-waiting exists
🚩 **Red Flags:** Can't explain what "ready" means

**Mid-Level:**
```
"Auto-waiting checks that elements are:
- Attached to DOM
- Visible
- Stable (not animating)
- Enabled

This happens before every action like click, fill, etc. Default timeout is 30 seconds."
```
✅ **Green Flags:** Lists conditions, knows timeout
🚩 **Red Flags:** Missing "receives events" condition

**Senior Level:**
```
"Auto-waiting performs actionability checks before every action:

1. Attached - Element exists in DOM
2. Visible - Not display:none, visibility:hidden, or 0 size
3. Stable - Not animating or position not changing
4. Receives Events - Not covered by another element
5. Enabled - Not disabled attribute (for form controls)

Different actions check different conditions:
- click() checks all 5
- fill() checks all 5 plus editable state
- isVisible() only checks visibility, no waiting

Default timeout: 30s (configurable globally or per-action)

When auto-wait isn't enough, I use:
- waitForLoadState() for network activity
- waitForResponse() for specific API calls
- waitForFunction() for custom conditions
- expect.toPass() for retry logic

Common gotcha: Auto-wait doesn't guarantee data is loaded,
only that element is actionable."
```
✅ **Green Flags:** Comprehensive explanation, mentions gotchas, knows different actions
🚩 **Red Flags:** Thinks auto-waiting solves all timing issues

**Follow-up Questions:**
- "When would auto-waiting not be sufficient?"
- "How would you handle a case where an element is visible but disabled?"

---

## Practical Implementation

### Q4: "Show me how you would implement a login test using Page Object Model."

**What This Reveals:**
- Code organization skills
- Understanding of design patterns
- Real-world experience
- Code quality standards

**Expected Responses:**

**Junior Level:**
```typescript
class LoginPage {
  async login(email, password) {
    await this.page.fill('#email', email);
    await this.page.fill('#password', password);
    await this.page.click('#submit');
  }
}
```
✅ **Green Flags:** Basic POM structure
🚩 **Red Flags:** No types, no constructor, no locator definitions, no validations

**Mid-Level:**
```typescript
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('#email');
    this.passwordInput = page.locator('#password');
    this.submitButton = page.locator('#submit');
    this.errorMessage = page.locator('.error');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage(): Promise<string> {
    return await this.errorMessage.textContent() || '';
  }
}

// Test file
test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('user@test.com', 'password');
  await expect(page).toHaveURL('/dashboard');
});
```
✅ **Green Flags:** TypeScript, locators defined, separation of concerns
🚩 **Red Flags:** Hardcoded selectors, no reusability considerations

**Senior Level:**
```typescript
// pages/BasePage.ts
import { Page } from '@playwright/test';

export abstract class BasePage {
  constructor(protected readonly page: Page) {}

  protected async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  protected async takeScreenshot(name: string): Promise<void> {
    await this.page.screenshot({
      path: `screenshots/${name}-${Date.now()}.png`,
      fullPage: true
    });
  }
}

// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  // Selectors - easy to maintain in one place
  private readonly selectors = {
    email: '[data-testid="email-input"]',
    password: '[data-testid="password-input"]',
    submit: '[data-testid="submit-button"]',
    error: '[data-testid="error-message"]',
    loadingSpinner: '[data-testid="loading-spinner"]'
  };

  // Locators
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  private readonly loadingSpinner: Locator;

  constructor(page: Page) {
    super(page);
    this.emailInput = page.locator(this.selectors.email);
    this.passwordInput = page.locator(this.selectors.password);
    this.submitButton = page.locator(this.selectors.submit);
    this.errorMessage = page.locator(this.selectors.error);
    this.loadingSpinner = page.locator(this.selectors.loadingSpinner);
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
    await this.waitForPageLoad();
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
    await this.waitForLoginComplete();
  }

  async loginWithEnterKey(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.passwordInput.press('Enter');
    await this.waitForLoginComplete();
  }

  private async waitForLoginComplete(): Promise<void> {
    // Wait for loading to disappear
    await this.loadingSpinner.waitFor({ state: 'hidden', timeout: 5000 })
      .catch(() => {}); // Loading might not appear for cached users
  }

  async getErrorMessage(): Promise<string> {
    await this.errorMessage.waitFor({ state: 'visible' });
    return await this.errorMessage.textContent() || '';
  }

  async hasError(): Promise<boolean> {
    return await this.errorMessage.isVisible();
  }

  async isEmailValid(): Promise<boolean> {
    const ariaInvalid = await this.emailInput.getAttribute('aria-invalid');
    return ariaInvalid !== 'true';
  }
}

// fixtures/pages.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

type PageFixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<PageFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await use(loginPage);
  }
});

export { expect } from '@playwright/test';

// tests/login.spec.ts
import { test, expect } from '../fixtures/pages';

test.describe('Login Functionality', () => {
  test('successful login with valid credentials', async ({ loginPage, page }) => {
    await loginPage.login('user@test.com', 'ValidPass123!');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('shows error for invalid credentials', async ({ loginPage }) => {
    await loginPage.login('invalid@test.com', 'wrongpass');

    expect(await loginPage.hasError()).toBeTruthy();
    const error = await loginPage.getErrorMessage();
    expect(error).toContain('Invalid credentials');
  });

  test('validates email format', async ({ loginPage }) => {
    await loginPage.emailInput.fill('not-an-email');
    await loginPage.emailInput.blur();

    expect(await loginPage.isEmailValid()).toBeFalsy();
  });

  test('can submit with Enter key', async ({ loginPage, page }) => {
    await loginPage.loginWithEnterKey('user@test.com', 'ValidPass123!');

    await expect(page).toHaveURL('/dashboard');
  });
});
```
✅ **Green Flags:**
- Base class for reusability
- data-testid selectors
- Multiple interaction methods
- Helper methods for validation
- Custom fixtures
- Loading state handling
- Comprehensive test coverage

🚩 **Red Flags:**
- None for senior, this is excellent

**Follow-up Questions:**
- "How would you handle this page if it had multiple forms?"
- "What if the login flow changes based on user role?"
- "How would you test this across different locales?"

---

### Q5: "How would you handle authentication in your test suite to avoid logging in for every test?"

**What This Reveals:**
- Performance optimization awareness
- Knowledge of Playwright features
- Real-world experience with test suites
- Understanding of test isolation vs efficiency

**Expected Responses:**

**Junior Level:**
```
"I would create a beforeEach hook that logs in before each test."
```
✅ **Green Flags:** Knows about hooks
🚩 **Red Flags:** Doesn't understand this is slow and doesn't scale

**Mid-Level:**
```
"I would use Playwright's storage state feature:
1. Create a setup file that logs in once
2. Save the authentication state to a file
3. Reuse that state in all tests

This way we only login once instead of before every test."
```
✅ **Green Flags:** Knows about storage state, understands performance benefit
🚩 **Red Flags:** Doesn't mention how to implement or potential issues

**Senior Level:**
```
"I use Playwright's storage state with a global setup pattern:

// auth.setup.ts
import { test as setup } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('[data-testid="email"]', process.env.TEST_USER_EMAIL);
  await page.fill('[data-testid="password"]', process.env.TEST_USER_PASSWORD);
  await page.click('[data-testid="submit"]');

  // Wait for auth to complete
  await page.waitForURL('/dashboard');

  // Save authentication state
  await page.context().storageState({ path: authFile });
});

// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/
    },
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: authFile
      },
      dependencies: ['setup']
    }
  ]
});

For multiple user roles:

// fixtures/auth.ts
export const test = base.extend<{
  authenticatedUser: Page;
  adminUser: Page;
}>({
  authenticatedUser: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'playwright/.auth/user.json'
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },

  adminUser: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'playwright/.auth/admin.json'
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  }
});

For API-based auth:

// fixtures/api-auth.ts
export const test = base.extend({
  authenticatedPage: async ({ page, request }, use) => {
    // Login via API (faster than UI)
    const response = await request.post('/api/auth/login', {
      data: {
        email: process.env.TEST_USER_EMAIL,
        password: process.env.TEST_USER_PASSWORD
      }
    });

    const { token } = await response.json();

    // Set token in context
    await page.context().addCookies([{
      name: 'auth_token',
      value: token,
      domain: 'example.com',
      path: '/'
    }]);

    await use(page);
  }
});

For tests that need to be logged out:
test.use({ storageState: { cookies: [], origins: [] } });

This approach:
- Runs login once per test run
- Reduces test execution time by 70%+
- Maintains test isolation (each test gets fresh context)
- Supports multiple user types
- Works in CI/CD
- Can use faster API auth when available
"
```
✅ **Green Flags:**
- Complete implementation
- Multiple approaches
- Environment variables
- Multiple user roles
- Performance metrics
- CI/CD consideration

🚩 **Red Flags:** None

**Follow-up Questions:**
- "What if the session expires during test execution?"
- "How would you handle auth tokens that expire in 5 minutes?"
- "How would you test the login flow itself if you're always authenticated?"

---

## Problem-Solving & Debugging

### Q6: "A test is failing intermittently in CI but passes locally. How would you debug this?"

**What This Reveals:**
- Debugging methodology
- Understanding of flakiness causes
- CI/CD experience
- Systematic problem-solving approach

**Expected Responses:**

**Junior Level:**
```
"I would check the error message and try running the test multiple times locally.
Maybe add some wait times to make it more stable."
```
✅ **Green Flags:** Knows to check error message
🚩 **Red Flags:** Suggests fixed waits, no systematic approach

**Mid-Level:**
```
"I would:
1. Check CI logs and screenshots
2. Look for timing issues - CI is usually slower
3. Check for hardcoded values that might differ in CI
4. Run tests in headed mode in CI to see what's happening
5. Add retry logic to see if it's consistently flaky
6. Check network requests for failures

Common causes:
- Different screen sizes
- Missing environment variables
- Race conditions
- External dependencies (APIs, databases)
"
```
✅ **Green Flags:** Systematic approach, knows common causes
🚩 **Red Flags:** Doesn't mention traces or specific debugging tools

**Senior Level:**
```
"I follow a systematic debugging process:

Phase 1: Gather Information
1. Enable traces in CI (trace: 'retain-on-failure')
2. Check artifacts: screenshots, videos, traces
3. Review CI logs for errors, warnings, timeouts
4. Check if failure is browser-specific
5. Check failure pattern: random, first test, after certain test
6. Compare CI vs local environment (OS, resources, network)

Phase 2: Reproduce Locally
1. Use same viewport size as CI
2. Run with --workers=1 to check for isolation issues
3. Run with network throttling:
   use: { launchOptions: { slowMo: 50 } }
4. Set TEST_ENV to match CI
5. Run in Docker container matching CI environment
6. Use --repeat-each=10 to catch intermittent failures

Phase 3: Identify Root Cause
Common issues and solutions:

A. Race Conditions
   Problem: Actions happening before elements ready
   Solution:
   - Use waitForResponse() for API calls
   - Use waitForLoadState('networkidle')
   - Avoid force: true, it masks problems

B. Resource Constraints
   Problem: CI has less memory/CPU
   Solution:
   - Reduce parallel workers
   - Increase timeouts in CI
   - Optimize test data size

C. Environment Differences
   Problem: Different configs, data, services
   Solution:
   - Use environment variables consistently
   - Document all dependencies
   - Use docker-compose for consistent environment

D. Test Isolation Issues
   Problem: Tests affect each other
   Solution:
   - Run with --workers=1 to verify
   - Check for shared state
   - Ensure proper cleanup in afterEach

E. External Dependencies
   Problem: API flakiness, database state
   Solution:
   - Mock external services
   - Use fixtures for test data
   - Add retry for network calls

Phase 4: Fix and Validate
1. Implement fix based on root cause
2. Run locally 50+ times:
   npx playwright test --repeat-each=50
3. Run in CI multiple times before merging
4. Add monitoring/logging if issue is complex
5. Document the issue and fix for team

Example Fix:
// Before (flaky)
await page.click('#submit');
await expect(page.locator('.success')).toBeVisible();

// After (stable)
const [response] = await Promise.all([
  page.waitForResponse(resp => resp.url().includes('/api/submit')),
  page.click('#submit')
]);
await expect(response).toBeOK();
await expect(page.locator('.success')).toBeVisible();

Prevention:
- Code review checklist for common flakiness patterns
- Require traces on all test failures
- Monitor flakiness rate metrics
- Quarantine flaky tests until fixed
- Never merge with 'skip' or ignore flakiness
"
```
✅ **Green Flags:**
- Systematic multi-phase approach
- Specific tools and commands
- Multiple root cause scenarios
- Prevention strategies
- Concrete examples

🚩 **Red Flags:** None

**Follow-up Questions:**
- "Give me a real example of a flaky test you debugged."
- "How would you prevent flakiness from being merged?"
- "What metrics would you track for test stability?"

---

### Q7: "How do you handle dynamic elements or content that changes on each page load?"

**What This Reveals:**
- Experience with real-world scenarios
- Knowledge of advanced assertions
- Creative problem-solving

**Expected Responses:**

**Junior Level:**
```
"I would use regular expressions or check if the element just exists
instead of checking exact text."
```
✅ **Green Flags:** Basic understanding
🚩 **Red Flags:** Vague, no concrete examples

**Mid-Level:**
```
"For dynamic content, I would:
1. Use partial text matching: expect(text).toContain('partial')
2. Use regex: expect(text).toMatch(/pattern/)
3. For timestamps, check format not exact value
4. For IDs, just verify they exist and are non-empty
5. Use toHaveCount() for lists that vary in size

For screenshots, I'd mask dynamic areas:
await expect(page).toHaveScreenshot({
  mask: [page.locator('.timestamp')]
});
"
```
✅ **Green Flags:** Multiple strategies, knows masking
🚩 **Red Flags:** Doesn't mention property-based testing

**Senior Level:**
```
"I use different strategies based on the type of dynamic content:

1. Timestamps/Dates
// Instead of exact match
await expect(page.locator('.timestamp')).toHaveText('2024-01-15');

// Check format and recency
const timestampText = await page.locator('.timestamp').textContent();
const timestamp = new Date(timestampText);
expect(timestamp).toBeInstanceOf(Date);
expect(Date.now() - timestamp.getTime()).toBeLessThan(5000); // Within 5s

2. Generated IDs
// Don't check exact value
const userId = await page.locator('.user-id').textContent();
expect(userId).toMatch(/^user-[a-f0-9]{8}-[a-f0-9]{4}/); // UUID format

3. Dynamic Lists
// Use count ranges or property checks
const items = page.locator('.item');
await expect(items).toHaveCount.toBeGreaterThan(0);

// Check all items have required properties
const itemCount = await items.count();
for (let i = 0; i < itemCount; i++) {
  await expect(items.nth(i).locator('.title')).toBeVisible();
  await expect(items.nth(i).locator('.price')).toHaveText(/\\$\\d+\\.\\d{2}/);
}

// Or use evaluate for bulk checks
const allValid = await page.evaluate(() => {
  const items = document.querySelectorAll('.item');
  return Array.from(items).every(item =>
    item.querySelector('.title')?.textContent &&
    item.querySelector('.price')?.textContent
  );
});
expect(allValid).toBeTruthy();

4. Visual Regression with Dynamic Content
// Mask dynamic areas
await expect(page).toHaveScreenshot('dashboard.png', {
  mask: [
    page.locator('.timestamp'),
    page.locator('.user-avatar'),
    page.locator('.ad-banner')
  ],
  animations: 'disabled'
});

// Or replace dynamic content before screenshot
await page.evaluate(() => {
  document.querySelectorAll('.timestamp').forEach(el => {
    el.textContent = 'MOCKED_TIMESTAMP';
  });
});

5. API Responses with Dynamic Data
const response = await request.get('/api/users');
const users = await response.json();

// Schema validation instead of exact values
expect(users).toEqual(
  expect.arrayContaining([
    expect.objectContaining({
      id: expect.any(Number),
      name: expect.any(String),
      email: expect.stringMatching(/^.*@.*\\..*$/),
      createdAt: expect.any(String)
    })
  ])
);

// Property-based testing
expect(users.every(u => u.id > 0)).toBeTruthy();
expect(users.every(u => u.name.length > 0)).toBeTruthy();

6. Random Order
// Don't check order, check presence
const productNames = ['Product A', 'Product B', 'Product C'];
for (const name of productNames) {
  await expect(page.locator(`text=${name}`)).toBeVisible();
}

7. Test Data Approach
// Create known test data, verify against it
const testProduct = await createTestProduct({
  name: 'Test Product',
  price: 99.99
});

await page.goto('/products');
await expect(page.locator(`[data-product-id="${testProduct.id}"]`)).toBeVisible();

// Cleanup
await deleteTestProduct(testProduct.id);

8. Snapshot Testing for Complex Objects
// Save snapshot on first run, compare on subsequent
expect(apiResponse).toMatchSnapshot({
  id: expect.any(String),
  timestamp: expect.any(String)
});

Key Principles:
- Test behavior, not implementation details
- Verify properties/patterns, not exact values
- Use schema validation for structured data
- Control test data when possible
- Mask or mock truly random elements
"
```
✅ **Green Flags:**
- Multiple scenarios covered
- Code examples for each
- Principles explained
- Both UI and API approaches

🚩 **Red Flags:** None

**Follow-up Questions:**
- "How would you test a live stock ticker or real-time chat?"
- "What if the entire page content is AI-generated and different each time?"

---

## Architecture & Design

### Q8: "How would you structure a test framework for a large application with 1000+ tests?"

**What This Reveals:**
- Architectural thinking
- Experience with scale
- Team collaboration awareness
- Long-term maintainability focus

**Expected Responses:**

**Junior Level:**
```
"I would organize tests by feature and use page objects for reusable code."
```
✅ **Green Flags:** Knows basics of organization
🚩 **Red Flags:** No details, doesn't think about scale challenges

**Mid-Level:**
```
"I would structure it like this:

tests/
  auth/
  dashboard/
  checkout/
pages/
  LoginPage.ts
  DashboardPage.ts
fixtures/
  custom-fixtures.ts
utils/
  helpers.ts

Use tags for different test types: @smoke, @regression
Configure parallel execution in playwright.config.ts
Use page objects for maintainability
Separate API tests from UI tests
"
```
✅ **Green Flags:** Clear structure, mentions tags, parallel execution
🚩 **Red Flags:** Doesn't address 1000+ test scale challenges

**Senior Level:**
```
"For a large-scale framework, I'd implement:

1. Directory Structure
project-root/
├── tests/
│   ├── e2e/              # End-to-end critical paths
│   │   ├── smoke/        # @smoke - run on every commit
│   │   ├── regression/   # @regression - nightly
│   │   └── sanity/       # @sanity - post-deployment
│   ├── integration/      # Feature integration tests
│   ├── api/              # API contract tests
│   └── visual/           # Visual regression
├── src/
│   ├── pages/            # Page Object Models
│   │   ├── base/
│   │   │   └── BasePage.ts
│   │   ├── auth/
│   │   │   ├── LoginPage.ts
│   │   │   └── SignupPage.ts
│   │   └── index.ts      # Barrel exports
│   ├── components/       # Reusable component objects
│   │   ├── Header.ts
│   │   ├── Modal.ts
│   │   └── index.ts
│   ├── fixtures/
│   │   ├── auth.fixture.ts
│   │   ├── data.fixture.ts
│   │   └── pages.fixture.ts
│   ├── utils/
│   │   ├── api-client.ts
│   │   ├── test-data-builder.ts
│   │   ├── date-helpers.ts
│   │   └── env-config.ts
│   ├── config/
│   │   ├── environments.ts
│   │   └── test-data.ts
│   └── types/
│       ├── api.types.ts
│       └── test.types.ts
├── playwright.config.ts
├── .env.example
└── README.md

2. Playwright Configuration
// playwright.config.ts
export default defineConfig({
  // Global settings
  timeout: 60000,
  expect: { timeout: 10000 },
  fullyParallel: true,

  // Workers based on environment
  workers: process.env.CI ? 4 : 8,

  // Retries
  retries: process.env.CI ? 2 : 0,

  // Reporter configuration
  reporter: [
    ['html'],
    ['json', { outputFile: 'results.json' }],
    ['junit', { outputFile: 'results.xml' }],
    ['./src/reporters/slack-reporter.ts'],
    ['./src/reporters/metrics-reporter.ts']
  ],

  // Projects for different test types
  projects: [
    // Setup
    { name: 'setup', testMatch: /.*\\.setup\\.ts/ },

    // Smoke tests (fast, critical)
    {
      name: 'smoke-chromium',
      testMatch: /.*smoke.*\\.spec\\.ts/,
      use: { ...devices['Desktop Chrome'] },
      dependencies: ['setup']
    },

    // Regression (comprehensive)
    {
      name: 'regression-chromium',
      testMatch: /.*regression.*\\.spec\\.ts/,
      use: { ...devices['Desktop Chrome'] },
      dependencies: ['setup']
    },

    // Cross-browser
    {
      name: 'firefox',
      testMatch: /.*smoke.*\\.spec\\.ts/,
      use: { ...devices['Desktop Firefox'] }
    },

    // Mobile
    {
      name: 'mobile',
      testMatch: /.*mobile.*\\.spec\\.ts/,
      use: { ...devices['iPhone 13'] }
    },

    // API tests (no browser)
    {
      name: 'api',
      testMatch: /.*api\\.spec\\.ts/,
      use: {
        baseURL: process.env.API_BASE_URL
      }
    }
  ]
});

3. Tagging Strategy
// tests/checkout/purchase.spec.ts
test.describe('Checkout Flow', () => {
  test('complete purchase @smoke @critical @checkout', async () => {});
  test('apply coupon @regression @checkout', async () => {});
  test('guest checkout @regression @checkout @guest', async () => {});
});

// Run specific tags
// npx playwright test --grep @smoke
// npx playwright test --grep "@checkout.*@critical"
// npx playwright test --grep-invert @flaky

4. Shared Fixtures
// fixtures/pages.fixture.ts
export const test = base.extend<PageFixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  }
});

// fixtures/auth.fixture.ts
export const test = base.extend({
  authenticatedUser: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'auth/user.json'
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  }
});

5. Test Data Management
// utils/test-data-builder.ts
export class UserBuilder {
  private user = {
    email: faker.internet.email(),
    password: 'Test123!',
    firstName: faker.person.firstName(),
    lastName: faker.person.lastName()
  };

  withEmail(email: string) {
    this.user.email = email;
    return this;
  }

  withAdmin() {
    this.user.role = 'admin';
    return this;
  }

  build() {
    return this.user;
  }

  async create(request: APIRequestContext) {
    const response = await request.post('/api/users', {
      data: this.user
    });
    return await response.json();
  }
}

// Usage
const user = await new UserBuilder()
  .withAdmin()
  .create(request);

6. Execution Strategy
// package.json
{
  "scripts": {
    "test": "playwright test",
    "test:smoke": "playwright test --grep @smoke",
    "test:regression": "playwright test --grep @regression",
    "test:api": "playwright test --project=api",
    "test:headed": "playwright test --headed",
    "test:debug": "playwright test --debug",
    "test:ui": "playwright test --ui",
    "test:parallel": "playwright test --workers=8",
    "test:serial": "playwright test --workers=1",
    "report": "playwright show-report",
    "codegen": "playwright codegen"
  }
}

7. CI/CD Integration
# .github/workflows/tests.yml
- Smoke tests: Every PR
- Regression: Nightly
- Full suite: Weekly
- Sharding: 4-8 parallel jobs

8. Monitoring & Metrics
- Track test duration trends
- Flakiness rate per test
- Code coverage
- Test execution time by tag
- Success rate by browser

9. Best Practices Enforcement
- Pre-commit hooks with lint-staged
- ESLint rules for test patterns
- Code review checklist
- Automated dependency updates
- Documentation requirements

10. Team Guidelines
- Naming conventions
- When to use which test type
- How to handle flaky tests
- Data cleanup requirements
- Review process

This scales because:
✓ Clear separation of concerns
✓ Reusable components
✓ Tag-based selective execution
✓ Parallel execution optimized
✓ Easy to onboard new team members
✓ Maintainable with TypeScript
✓ Comprehensive reporting
✓ CI/CD ready
"
```
✅ **Green Flags:**
- Comprehensive structure
- Scalability considerations
- Team collaboration
- CI/CD integration
- Metrics and monitoring
- Code examples

🚩 **Red Flags:** None

**Follow-up Questions:**
- "How would you handle test data cleanup at this scale?"
- "What would you do if tests start taking 4 hours to run?"
- "How would you onboard a new team member to this framework?"

---

### Q9: "When would you choose UI testing vs API testing vs unit testing?"

**What This Reveals:**
- Understanding of testing pyramid
- Strategic thinking
- Cost/benefit analysis
- Practical experience

**Expected Responses:**

**Junior Level:**
```
"UI testing tests the whole application through the browser.
API testing tests the backend.
Unit testing tests individual functions."
```
✅ **Green Flags:** Basic definitions correct
🚩 **Red Flags:** No understanding of when to use each

**Mid-Level:**
```
"I follow the testing pyramid:
- Unit tests: Many, fast, cheap - test business logic
- API tests: Medium, faster than UI - test integrations
- UI tests: Few, slow, expensive - test critical user paths

I use:
- Unit tests for calculations, validators, utility functions
- API tests for endpoints, data flow, integrations
- UI tests for end-to-end critical user journeys

Example: For a checkout flow
- Unit: Tax calculation, discount logic
- API: Create order endpoint, payment processing
- UI: Complete purchase from cart to confirmation
"
```
✅ **Green Flags:** Understands pyramid, provides examples
🚩 **Red Flags:** Doesn't mention specific criteria for choosing

**Senior Level:**
```
"I use a strategic approach based on risk, speed, and coverage:

1. Unit Tests (70-80%)
When:
- Pure business logic
- Complex calculations
- Validators, formatters
- Utility functions
- Edge cases and error conditions

Why:
- Milliseconds to run
- Pinpoint failures
- Easy to write and maintain
- Can run thousands quickly

Example:
// calculateShipping.test.ts
describe('calculateShipping', () => {
  it('calculates standard shipping', () => {
    expect(calculateShipping(100, 'standard')).toBe(10);
  });

  it('free shipping over $50', () => {
    expect(calculateShipping(100, 'standard')).toBe(0);
  });
});

2. API Tests (15-25%)
When:
- Contract verification
- Data validation
- Authentication/authorization
- Integration between services
- Database operations
- Third-party integrations
- State transitions

Why:
- Faster than UI (no browser)
- More reliable (no UI flakiness)
- Better for test data setup
- Can test error scenarios UI can't reach
- Better test isolation

Example:
test('create order API', async ({ request }) => {
  const order = await request.post('/api/orders', {
    data: {
      items: [{ id: 1, quantity: 2 }],
      shipping: 'express'
    }
  });

  expect(order.ok()).toBeTruthy();
  const data = await order.json();
  expect(data.total).toBe(220); // Verifies calculation
  expect(data.status).toBe('pending');
});

3. UI E2E Tests (5-10%)
When:
- Critical user journeys (happy paths)
- Cross-cutting concerns (auth flow)
- Visual requirements
- Browser-specific behavior
- Accessibility
- Things that matter to revenue/users

Why:
- Verifies actual user experience
- Catches integration issues
- Tests UI logic and interactions
- Validates visual design
- Required for confidence in deployment

Example: Only test the critical path
test('purchase flow', async ({ page }) => {
  // Setup via API (fast)
  await setupTestProduct(request);

  // UI critical path only
  await page.goto('/products');
  await page.click('[data-testid="add-to-cart"]');
  await page.click('[data-testid="checkout"]');
  await fillShippingForm(page);
  await fillPaymentForm(page);
  await page.click('[data-testid="place-order"]');

  // Verify critical outcome
  await expect(page).toHaveURL(/\\/order\\/success/);
  await expect(page.locator('.confirmation')).toBeVisible();
});

Decision Framework:

┌─────────────────────────────────────────────────┐
│ Can this be tested without UI?                  │
│                                                 │
│ YES → API or Unit Test                          │
│ NO → Continue                                   │
└─────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────┐
│ Is this business logic without external deps?  │
│                                                 │
│ YES → Unit Test                                 │
│ NO → Continue                                   │
└─────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────┐
│ Does this involve multiple services/database?  │
│                                                 │
│ YES → API Test                                  │
│ NO → Continue                                   │
└─────────────────────────────────────────────────┘
            ↓
┌─────────────────────────────────────────────────┐
│ Is this a critical user journey or visual req? │
│                                                 │
│ YES → UI E2E Test                               │
│ NO → Reconsider if test is needed              │
└─────────────────────────────────────────────────┘

Real Example: E-commerce Checkout

Feature: Apply coupon code

Unit Test:
✓ Valid coupon format
✓ Discount calculation logic
✓ Expiration date validation
✓ Usage limit enforcement

API Test:
✓ POST /api/coupons/validate
✓ Coupon applied to order total
✓ Invalid coupon returns 400
✓ Expired coupon returns 403
✓ Database updated with usage

UI E2E Test:
✓ User can enter coupon in checkout
✓ Success message displays
✓ Total updates on screen
✓ Order confirmation shows discount

Coverage:
- 20 unit tests (< 1 second)
- 8 API tests (5 seconds)
- 1 UI test (30 seconds)
= Comprehensive coverage in 36 seconds

vs. All UI tests:
- 29 UI tests (15 minutes)
- Flaky, slow, expensive

Hybrid Approach for Playwright:
// Use API for setup, UI for verification
test('user sees personalized recommendations', async ({ page, request }) => {
  // Setup via API (fast, reliable)
  const user = await createUser(request);
  await addPurchaseHistory(request, user.id, ['book1', 'book2']);
  await generateRecommendations(request, user.id);

  // UI verification (what users see)
  await loginAs(page, user);
  await page.goto('/recommendations');

  await expect(page.locator('.recommendation')).toHaveCount(5);
  await expect(page.locator('text=Based on your purchases')).toBeVisible();
});

Key Principle:
Test at the lowest level that gives confidence.
Use UI tests sparingly for critical paths.
Use API tests for integrations and data.
Use unit tests for everything else.
"
```
✅ **Green Flags:**
- Clear decision framework
- Percentages and ratios
- Real examples
- Hybrid approach
- Strategic thinking

🚩 **Red Flags:** None

**Follow-up Questions:**
- "How would you convince a team that does only UI testing to adopt this pyramid?"
- "What metrics would you use to validate the right test distribution?"

---

## Advanced Concepts

### Q10: "How would you implement visual regression testing?"

**What This Reveals:**
- Knowledge of advanced testing techniques
- Experience with visual testing
- Understanding of challenges and solutions

**Expected Responses:**

**Junior Level:**
```
"I would take screenshots and compare them to see if anything changed."
```
✅ **Green Flags:** Basic concept understood
🚩 **Red Flags:** No knowledge of tools or challenges

**Mid-Level:**
```
"I would use Playwright's toHaveScreenshot() matcher:

await expect(page).toHaveScreenshot('homepage.png');

This takes a screenshot and compares it to the baseline.
On first run it saves the baseline, on subsequent runs it compares.

For dynamic content, I'd mask it:
await expect(page).toHaveScreenshot({
  mask: [page.locator('.timestamp')]
});
"
```
✅ **Green Flags:** Knows Playwright feature, understands masking
🚩 **Red Flags:** Doesn't discuss challenges or strategy

**Senior Level:**
```
"Visual regression testing requires careful strategy:

1. Implementation with Playwright
// tests/visual/homepage.visual.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Homepage Visual Tests', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    // Wait for critical content
    await page.waitForLoadState('networkidle');
    // Disable animations for consistency
    await page.addStyleTag({
      content: '*, *::before, *::after { animation: none !important; transition: none !important; }'
    });
  });

  test('homepage above fold', async ({ page }) => {
    await expect(page).toHaveScreenshot('homepage-hero.png', {
      fullPage: false,
      mask: [
        page.locator('.live-timestamp'),
        page.locator('.user-avatar'),
        page.locator('.ad-banner')
      ],
      maxDiffPixels: 100, // Allow minor rendering differences
      threshold: 0.2 // 20% threshold
    });
  });

  test('homepage full page', async ({ page }) => {
    await expect(page).toHaveScreenshot('homepage-full.png', {
      fullPage: true,
      animations: 'disabled'
    });
  });

  test('component level - product card', async ({ page }) => {
    const productCard = page.locator('[data-testid="product-card"]').first();

    await expect(productCard).toHaveScreenshot('product-card.png', {
      mask: [productCard.locator('.price')] // Mask dynamic pricing
    });
  });
});

2. Configuration
// playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 50,
      threshold: 0.2,
      animations: 'disabled'
    }
  },

  projects: [
    {
      name: 'visual-chromium',
      testMatch: /.*\\.visual\\.spec\\.ts/,
      use: {
        ...devices['Desktop Chrome'],
        viewport: { width: 1920, height: 1080 }, // Fixed viewport
        deviceScaleFactor: 1 // Prevent retina differences
      }
    }
  ]
});

3. Handling Dynamic Content
// Replace dynamic content
await page.evaluate(() => {
  // Mock timestamps
  document.querySelectorAll('[data-timestamp]').forEach(el => {
    el.textContent = '2024-01-01 12:00:00';
  });

  // Mock user-specific data
  document.querySelectorAll('[data-user-name]').forEach(el => {
    el.textContent = 'Test User';
  });

  // Remove random elements
  document.querySelectorAll('.ad-rotation').forEach(el => el.remove());
});

await expect(page).toHaveScreenshot('normalized-page.png');

4. Responsive Visual Testing
const viewports = [
  { name: 'mobile', width: 375, height: 667 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'desktop', width: 1920, height: 1080 }
];

for (const viewport of viewports) {
  test(`homepage ${viewport.name}`, async ({ page }) => {
    await page.setViewportSize(viewport);
    await page.goto('/');

    await expect(page).toHaveScreenshot(`homepage-${viewport.name}.png`);
  });
}

5. Updating Baselines
# Update all screenshots
npx playwright test --update-snapshots

# Update specific test
npx playwright test homepage --update-snapshots

# Update specific project
npx playwright test --project=visual-chromium --update-snapshots

6. CI/CD Integration
# .github/workflows/visual-tests.yml
- name: Run visual tests
  run: npx playwright test --project=visual-chromium

- name: Upload diff images on failure
  if: failure()
  uses: actions/upload-artifact@v3
  with:
    name: visual-diff
    path: |
      test-results/**/*-diff.png
      test-results/**/*-actual.png

7. Challenges and Solutions

Challenge 1: Font rendering differences across OS
Solution:
- Use docker containers for consistent environment
- Use web fonts, not system fonts
- Set deviceScaleFactor explicitly

Challenge 2: Images loading at different times
Solution:
await Promise.all(
  await page.getByRole('img').all().map(img => img.waitFor({ state: 'visible' }))
);

Challenge 3: Animations causing differences
Solution:
await page.addStyleTag({
  content: `
    *, *::before, *::after {
      animation-duration: 0s !important;
      transition-duration: 0s !important;
    }
  `
});

Challenge 4: External content (ads, social widgets)
Solution:
- Block external domains in config
- Mock external iframes
- Remove elements before screenshot

await page.route('**/*.doubleclick.net/**', route => route.abort());

Challenge 5: Large diffs hard to review
Solution:
- Component-level screenshots instead of full page
- Focus on critical UI areas
- Use tolerable thresholds

8. Best Practices
✓ Start with critical pages only
✓ Use component-level screenshots
✓ Mask dynamic content, don't ignore it
✓ Run in Docker for consistency
✓ Set fixed viewport sizes
✓ Disable animations
✓ Wait for fonts to load
✓ Review diffs in PR process
✓ Don't auto-update without review
✓ Track baseline update frequency

9. When NOT to use visual testing
✗ Highly dynamic content
✗ Personalized experiences
✗ Real-time data displays
✗ A/B tests running
✗ Frequent design changes

10. Metrics to Track
- False positive rate
- Time to run visual tests
- Baseline update frequency
- Caught visual bugs
- Review time for diffs

Alternative: Percy, Applitools
For more advanced features:
- Cross-browser rendering
- AI-powered diff detection
- Parallel screenshot processing
- Baseline management UI
- Better diff visualization
"
```
✅ **Green Flags:**
- Complete implementation
- Challenges identified
- Solutions provided
- CI/CD integration
- Best practices
- Knows when NOT to use

🚩 **Red Flags:** None

**Follow-up Questions:**
- "How would you handle visual tests for a white-label product?"
- "What's your experience with Percy or Applitools?"

---

## Process & Collaboration

### Q11: "How do you ensure test quality and prevent flaky tests from being merged?"

**What This Reveals:**
- Team process experience
- Quality standards
- Code review approach
- Prevention mindset

**Expected Responses:**

**Junior Level:**
```
"I run tests multiple times before committing to make sure they pass."
```
✅ **Green Flags:** Basic quality awareness
🚩 **Red Flags:** No systematic approach or team process

**Mid-Level:**
```
"I would:
1. Require all tests to pass in CI before merge
2. Run tests multiple times locally
3. Have code review check for common issues
4. No hardcoded waits allowed
5. Require proper error handling

I'd also have a checklist:
- Uses auto-waiting, not fixed waits
- Has proper assertions
- Cleans up test data
- No dependencies on other tests
"
```
✅ **Green Flags:** Multiple strategies, checklist approach
🚩 **Red Flags:** Doesn't mention metrics or monitoring

**Senior Level:**
```
"I implement multiple layers of prevention and detection:

1. Pre-commit Validation
// .husky/pre-commit
#!/bin/sh
npm run lint
npm run type-check
npm run test:changed -- --repeat-each=3

// Only run tests related to changed files 3 times

2. PR Requirements
# .github/workflows/pr-validation.yml
jobs:
  test-stability:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests 10 times
        run: npx playwright test --repeat-each=10 --workers=1

      - name: Fail if flaky
        run: |
          if grep -q "flaky" test-results.json; then
            echo "Flaky tests detected!"
            exit 1
          fi

3. Code Review Checklist
Create PR template:
```markdown
## Test Quality Checklist
- [ ] No `page.waitForTimeout()` or fixed waits
- [ ] Uses `data-testid` or semantic selectors
- [ ] No dependencies between tests
- [ ] Proper cleanup in `afterEach`
- [ ] Tests pass 10/10 times locally
- [ ] No force clicks or disabling auto-wait
- [ ] Proper error messages in assertions
- [ ] Test data is isolated
```

4. ESLint Rules
// .eslintrc.js
module.exports = {
  rules: {
    // Ban waitForTimeout
    'playwright/no-wait-for-timeout': 'error',

    // Require test isolation
    'playwright/no-conditional-in-test': 'error',

    // Enforce best practices
    'playwright/prefer-web-first-assertions': 'error',
    'playwright/prefer-to-be': 'error',
    'playwright/prefer-to-have-length': 'error',

    // Ban force actions
    'playwright/no-force-option': 'warn'
  }
};

5. Automated Flakiness Detection
// scripts/detect-flakiness.ts
async function detectFlakiness() {
  const results = [];

  // Run each test 50 times
  for (let i = 0; i < 50; i++) {
    const result = await runTests();
    results.push(result);
  }

  // Analyze results
  const flaky = results.filter(r => r.failed > 0);

  if (flaky.length > 0 && flaky.length < 50) {
    console.error('FLAKY TESTS DETECTED');
    console.error(`Failed ${flaky.length}/50 runs`);
    process.exit(1);
  }
}

6. Monitoring Dashboard
Track metrics:
- Test success rate over time
- Average test duration trend
- Flakiness rate per test
- Most common failure reasons
- Retry rate

Alert when:
- Success rate < 95%
- Duration increases > 20%
- Same test fails 3+ times in week

7. Quarantine Process
// playwright.config.ts
// Auto-quarantine flaky tests
const quarantinedTests = loadQuarantineList();

export default defineConfig({
  grep: new RegExp(`^(?!.*(${quarantinedTests.join('|')})).*$`),

  projects: [
    // Regular tests
    { name: 'stable', grep: /@stable/ },

    // Quarantined tests (don't block CI)
    {
      name: 'quarantine',
      grep: /@quarantine/,
      retries: 5
    }
  ]
});

8. Test Stability Gates
// CI/CD pipeline stages
Stage 1: Smoke tests (must pass 100%)
Stage 2: Regression tests (must pass 98%)
Stage 3: Quarantined tests (informational only)

Deploy only if Stage 1 & 2 pass.

9. Team Guidelines Document
# Test Writing Guidelines

## Anti-Patterns (Never Do)
❌ page.waitForTimeout(5000)
❌ await page.click('#button', { force: true })
❌ Depend on test execution order
❌ Share state between tests
❌ Use production data
❌ Ignore intermittent failures

## Best Practices (Always Do)
✅ Use auto-waiting
✅ Data-testid selectors
✅ Independent tests
✅ Proper cleanup
✅ Meaningful assertions
✅ Run 10 times before PR

## Common Flakiness Causes
1. Race conditions → Use waitForResponse()
2. Animations → Disable or wait for
3. External APIs → Mock them
4. Shared test data → Isolate per test
5. Network timing → Wait for specific responses

10. Automated Analysis
// Custom reporter
class StabilityReporter implements Reporter {
  onTestEnd(test: TestCase, result: TestResult) {
    // Detect common issues
    if (result.status === 'flaky') {
      this.analyzeFlakiness(test, result);
    }

    // Check for anti-patterns in code
    if (test.source.includes('waitForTimeout')) {
      console.warn(`❌ ${test.title} uses waitForTimeout`);
    }

    if (test.source.includes('force: true')) {
      console.warn(`❌ ${test.title} uses force click`);
    }
  }

  analyzeFlakiness(test: TestCase, result: TestResult) {
    const error = result.error?.message;

    if (error?.includes('Timeout')) {
      console.log(`💡 Suggestion: Add explicit waits for ${test.title}`);
    }

    if (error?.includes('Element is not visible')) {
      console.log(`💡 Suggestion: Wait for element visibility`);
    }
  }
}

11. Continuous Improvement
Weekly:
- Review flaky test report
- Update quarantine list
- Share learnings in team meeting

Monthly:
- Analyze flakiness trends
- Update guidelines
- Refactor common patterns

Quarterly:
- Framework health review
- Dependency updates
- Training sessions

12. Enforcement
Pull Request Flow:
1. CI runs tests 5 times ← Catches flakiness
2. Linter checks anti-patterns ← Prevents bad code
3. Code review ← Human validation
4. Merge only if all green ← Quality gate

Results:
- Flakiness rate: < 1%
- False failures: Rare
- Developer confidence: High
- Maintenance: Low
"
```
✅ **Green Flags:**
- Multi-layered approach
- Automation
- Team process
- Monitoring
- Continuous improvement
- Specific metrics

🚩 **Red Flags:** None

**Follow-up Questions:**
- "What would you do if senior developers push back on these processes?"
- "How do you balance speed with quality in a fast-paced startup?"

---

## Behavioral & Scenario-Based

### Q12: "Tell me about a time you found a critical bug that automated tests missed. How did you prevent that in the future?"

**What This Reveals:**
- Real experience
- Learning from failures
- Improvement mindset
- Problem-solving approach

**What to Listen For:**

**Red Flags:**
- "Our tests never miss anything"
- Blames others
- No learning or improvement
- Defensive

**Green Flags:**
- Specific example
- Takes ownership
- Explains root cause
- Describes prevention strategy
- Mentions implementation

**Example Good Answer:**
```
"At my last company, we had a critical production bug where users couldn't
complete checkout if they had more than 10 items in cart.

Our tests missed it because:
1. We only tested with 1-3 items
2. The bug was in pagination logic that only triggered at 10+ items
3. We focused on happy path only

Impact:
- Lost revenue for 4 hours
- Customer complaints
- Emergency hotfix

Root Cause Analysis:
- Insufficient edge case testing
- No boundary testing
- Test data wasn't varied enough

Prevention Steps I Implemented:
1. Added boundary testing:
   - 0 items, 1 item, 10 items, 50 items, 100 items

2. Property-based testing:
   for (const itemCount of [0, 1, 5, 10, 15, 50, 100]) {
     test(`checkout with ${itemCount} items`, async ({ page }) => {
       await addItemsToCart(page, itemCount);
       await completeCheckout(page);
       await expect(page).toHaveURL(/success/);
     });
   }

3. Exploratory testing checklist for QA:
   - Test with extreme values
   - Test with empty states
   - Test with maximum limits

4. Monitoring in production:
   - Alert on checkout failures
   - Track completion rate

5. Production smoke tests:
   - Run critical path tests against production
   - Alert if failure rate > 1%

Result:
- No similar bugs in 18 months
- Test coverage increased from 60% to 85%
- Team adopted boundary testing as standard

Lesson Learned:
Don't just test happy path. Think about edge cases, boundaries, and
what assumptions you're making."
```

**Follow-up Questions:**
- "How did you convince the team to spend time on edge cases?"
- "What's your process for deciding which edge cases to test?"

---

### Q13: "How do you handle disagreements with developers about whether a test failure is valid?"

**What This Reveals:**
- Communication skills
- Collaboration approach
- Technical credibility
- Conflict resolution

**What to Listen For:**

**Red Flags:**
- "I'm always right"
- "Developers don't understand testing"
- Adversarial attitude
- Can't provide examples

**Green Flags:**
- Collaborative approach
- Data-driven discussion
- Seeks to understand
- Provides clear evidence
- Finds common ground

**Example Good Answer:**
```
"This happens occasionally. My approach:

1. Gather Evidence
   - Screenshots/video of failure
   - Error messages and stack traces
   - Steps to reproduce
   - Environment details
   - Trace files

2. Reproduce Consistently
   - "Can you reproduce this?"
   - If not, it might be environmental
   - Run test 10 times to prove consistency
   - Try in different environments

3. Collaborative Discussion
   - Show, don't tell
   - "Here's what I'm seeing..."
   - "Walk me through the expected behavior"
   - "Is there something I'm misunderstanding?"

4. Determine Root Cause Together
   Options:
   a) Valid bug → File ticket with evidence
   b) Test is wrong → Fix the test
   c) Expected behavior changed → Update test
   d) Environmental issue → Fix environment

Real Example:
Developer said my test was wrong because it expected error message
but the API was "working fine."

My process:
1. Showed network logs: API returned 500
2. Showed expected vs actual response
3. Developer checked server logs
4. Found database connection pool exhausted under load
5. Real bug discovered

We:
- Fixed the bug
- Added connection pool monitoring
- Added API test for concurrent requests
- Developer thanked me

Key: Stay curious, not confrontational.
Questions like "Help me understand..." work better than "You're wrong."

When I'm wrong (which happens):
- Acknowledge quickly
- Fix the test
- Learn from it
- Thank them for catching it

Goal: Ship quality software together, not be right."
```

---

## Coding Exercises

### Q14: "Live Coding: Write a test for a login form with these requirements..."

**Exercise Setup:**
```
Login form requirements:
1. Email and password fields
2. Submit button
3. Show error on invalid credentials
4. Redirect to /dashboard on success
5. Show loading spinner during login
6. Disable submit button while loading
```

**What to Evaluate:**

**Junior Expected:**
```typescript
test('login test', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'password');
  await page.click('#submit');
  await expect(page).toHaveURL('/dashboard');
});
```
✅ Basic test structure
🚩 No edge cases, no error handling

**Mid-Level Expected:**
```typescript
import { test, expect } from '@playwright/test';

test.describe('Login', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('successful login', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="submit"]');

    await expect(page).toHaveURL('/dashboard');
  });

  test('shows error on invalid credentials', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'wrong@example.com');
    await page.fill('[data-testid="password"]', 'wrongpass');
    await page.click('[data-testid="submit"]');

    await expect(page.locator('.error')).toHaveText('Invalid credentials');
  });

  test('disables button while loading', async ({ page }) => {
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');

    const submitButton = page.locator('[data-testid="submit"]');
    await submitButton.click();

    await expect(submitButton).toBeDisabled();
  });
});
```
✅ Multiple test cases, better selectors
🚩 No POM, no fixtures, hardcoded data

**Senior Expected:**
```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  readonly loadingSpinner: Locator;

  constructor(private page: Page) {
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.locator('[data-testid="error-message"]');
    this.loadingSpinner = page.locator('[data-testid="loading-spinner"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage(): Promise<string> {
    await this.errorMessage.waitFor({ state: 'visible' });
    return await this.errorMessage.textContent() || '';
  }

  async isLoading(): Promise<boolean> {
    return await this.loadingSpinner.isVisible();
  }

  async isSubmitDisabled(): Promise<boolean> {
    return await this.submitButton.isDisabled();
  }
}

// fixtures/testData.ts
export const testUsers = {
  valid: {
    email: process.env.TEST_USER_EMAIL || 'test@example.com',
    password: process.env.TEST_USER_PASSWORD || 'Test123!'
  },
  invalid: {
    email: 'invalid@example.com',
    password: 'wrongpassword'
  }
};

// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { testUsers } from '../fixtures/testData';

test.describe('Login Functionality', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test('successful login redirects to dashboard', async ({ page }) => {
    await loginPage.login(testUsers.valid.email, testUsers.valid.password);

    // Verify redirect
    await expect(page).toHaveURL('/dashboard');

    // Verify user is actually logged in
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('invalid credentials show error message', async ({ page }) => {
    await loginPage.login(testUsers.invalid.email, testUsers.invalid.password);

    // Should not redirect
    await expect(page).toHaveURL('/login');

    // Should show error
    const errorMessage = await loginPage.getErrorMessage();
    expect(errorMessage).toContain('Invalid credentials');
  });

  test('submit button disabled during login request', async ({ page }) => {
    // Slow down network to see loading state
    await page.route('**/api/auth/login', async route => {
      await new Promise(resolve => setTimeout(resolve, 1000));
      await route.continue();
    });

    await loginPage.login(testUsers.valid.email, testUsers.valid.password);

    // Verify loading state
    await expect(loginPage.submitButton).toBeDisabled();
    await expect(loginPage.loadingSpinner).toBeVisible();
  });

  test('form validation for empty fields', async ({ page }) => {
    await loginPage.submitButton.click();

    // Should show validation errors
    await expect(loginPage.emailInput).toHaveAttribute('aria-invalid', 'true');
  });

  test('form validation for invalid email format', async ({ page }) => {
    await loginPage.emailInput.fill('not-an-email');
    await loginPage.emailInput.blur();

    await expect(page.locator('.email-error')).toHaveText('Invalid email format');
  });

  test('can submit form with Enter key', async ({ page }) => {
    await loginPage.emailInput.fill(testUsers.valid.email);
    await loginPage.passwordInput.fill(testUsers.valid.password);
    await loginPage.passwordInput.press('Enter');

    await expect(page).toHaveURL('/dashboard');
  });

  test('password field is type password', async ({ page }) => {
    await expect(loginPage.passwordInput).toHaveAttribute('type', 'password');
  });

  test.describe('Accessibility', () => {
    test('form has proper labels', async ({ page }) => {
      await expect(loginPage.emailInput).toHaveAccessibleName('Email');
      await expect(loginPage.passwordInput).toHaveAccessibleName('Password');
    });

    test('error messages are announced', async ({ page }) => {
      await loginPage.login(testUsers.invalid.email, testUsers.invalid.password);

      await expect(loginPage.errorMessage).toHaveAttribute('role', 'alert');
    });
  });
});
```
✅ **Excellent:**
- Page Object Model
- Semantic selectors
- Environment variables
- Multiple test scenarios
- Loading states
- Accessibility
- Keyboard interaction
- Form validation

---

## Evaluation Rubric

### Junior (0-2 years)
**Technical Knowledge:**
- [ ] Knows basic Playwright syntax
- [ ] Understands locators
- [ ] Can write simple tests
- [ ] Knows about auto-waiting

**Gaps (Acceptable):**
- Limited debugging skills
- No architecture experience
- Basic error handling
- Minimal CI/CD knowledge

**Red Flags:**
- Doesn't know auto-waiting
- Uses fixed waits extensively
- Can't explain selectors
- No testing theory knowledge

---

### Mid-Level (2-5 years)
**Technical Knowledge:**
- [ ] Strong Playwright API knowledge
- [ ] Uses Page Object Model
- [ ] Understands test isolation
- [ ] Can debug flaky tests
- [ ] Knows CI/CD basics
- [ ] API testing experience

**Gaps (Acceptable):**
- Limited architecture design
- Some flakiness issues
- Basic monitoring

**Red Flags:**
- Can't explain flakiness causes
- No POM experience
- Doesn't know fixtures
- Never used CI/CD

---

### Senior (5+ years)
**Technical Knowledge:**
- [ ] Expert Playwright knowledge
- [ ] Architectural design experience
- [ ] Advanced debugging
- [ ] Performance optimization
- [ ] Framework design
- [ ] Team leadership
- [ ] CI/CD expertise
- [ ] Monitoring and metrics

**Expected:**
- Can design framework from scratch
- Mentors others
- Prevents problems before they occur
- Strategic thinking
- ROI awareness

**Red Flags:**
- Can't explain trade-offs
- No team experience
- Hasn't designed frameworks
- Doesn't think about scale
- No mentoring experience

---

## Interview Questions Summary

### Must Ask (Everyone):
1. Explain auto-waiting
2. Show me a login test with POM
3. How do you handle flaky tests?
4. Debugging scenario question

### Mid-Level Additional:
5. Authentication strategy
6. API vs UI testing
7. Structure large test suite

### Senior Additional:
8. Framework architecture
9. Team process and quality gates
10. Scaling and performance

---

## Final Notes for Interviewers

**Green Flags to Look For:**
✅ Asks clarifying questions
✅ Thinks about edge cases
✅ Considers maintainability
✅ Mentions team collaboration
✅ Provides real examples
✅ Admits when unsure
✅ Learns from mistakes
✅ Strategic thinking

**Red Flags to Watch:**
🚩 Overconfident without knowledge
🚩 Blames others for failures
🚩 No real-world examples
🚩 Doesn't ask questions
🚩 Recommends bad practices
🚩 Can't explain reasoning
🚩 No growth mindset
🚩 Adversarial attitude

**Cultural Fit:**
- Collaborative mindset
- Continuous learning
- Quality-focused
- Communication skills
- Problem-solving approach
- Team player

---

Good luck with your interviews! 🎭
