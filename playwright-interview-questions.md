# Playwright Interview Questions & Answers

## Table of Contents
1. [Basic Concepts](#basic-concepts)
2. [Intermediate Concepts](#intermediate-concepts)
3. [Advanced Concepts](#advanced-concepts)
4. [Architecture & Design](#architecture--design)
5. [AI/Copilot/MCP Integration](#aicopilotmcp-integration)
6. [Troubleshooting & Debugging](#troubleshooting--debugging)

---

## Basic Concepts

### Q1: What is Playwright and how does it differ from Selenium?

**Answer:**
Playwright is a modern end-to-end testing framework developed by Microsoft. Key differences from Selenium:

| Feature | Playwright | Selenium |
|---------|-----------|----------|
| **Browser Control** | Direct protocol (CDP for Chromium, custom for Firefox/WebKit) | WebDriver protocol |
| **Auto-waiting** | Built-in smart waiting | Manual waits needed |
| **Multi-browser** | Chromium, Firefox, WebKit out of box | Requires separate drivers |
| **Speed** | Faster (direct protocol) | Slower (WebDriver overhead) |
| **Multi-tab/context** | Native support | Limited |
| **Network interception** | Built-in | Requires third-party tools |
| **TypeScript** | First-class support | Requires setup |

**Example:**
```typescript
// Playwright - auto-waits for element
await page.locator('#submit').click();

// Selenium - manual waiting needed
WebDriverWait wait = new WebDriverWait(driver, 10);
wait.until(ExpectedConditions.elementToBeClickable(By.id("submit"))).click();
```

---

### Q2: Explain the difference between `page.locator()` and `page.$()`.

**Answer:**
- `page.locator()`: Modern API, auto-waiting, strict mode, chainable
- `page.$()`: Legacy API (from Puppeteer), returns ElementHandle or null, no auto-waiting

```typescript
// Recommended: locator (auto-waits, strict)
const button = page.locator('button.submit');
await button.click();

// Legacy: $ (no auto-wait, can be null)
const button = await page.$('button.submit');
if (button) {
  await button.click();
}
```

**Best Practice:** Always use `locator()` for new code.

---

### Q3: What are the different types of selectors in Playwright?

**Answer:**
```typescript
// 1. CSS Selector
page.locator('.class-name')
page.locator('#id')
page.locator('button[type="submit"]')

// 2. Text Selector
page.locator('text=Sign in')
page.locator('button:has-text("Submit")')

// 3. XPath (use sparingly)
page.locator('xpath=//button[@type="submit"]')

// 4. Data-testid (best practice)
page.locator('[data-testid="submit-button"]')

// 5. Role-based (accessibility)
page.getByRole('button', { name: 'Submit' })
page.getByRole('textbox', { name: 'Email' })

// 6. Label text
page.getByLabel('Email address')

// 7. Placeholder
page.getByPlaceholder('Enter your email')

// 8. Alt text (images)
page.getByAltText('Profile picture')
```

**Best Practice:** Prefer `getByRole()`, `getByLabel()`, and `data-testid` for maintainability.

---

### Q4: How does auto-waiting work in Playwright?

**Answer:**
Playwright automatically waits for elements to be:
1. **Attached** to DOM
2. **Visible** (not `display: none` or `visibility: hidden`)
3. **Stable** (not animating)
4. **Receives Events** (not covered by other elements)
5. **Enabled** (not disabled)

```typescript
// Auto-waits for all conditions before clicking
await page.locator('#submit').click();

// Default timeout: 30 seconds (configurable)
await page.locator('#slow-element').click({ timeout: 60000 });

// You can disable auto-wait if needed (rare)
await page.locator('#element').click({ force: true });
```

---

### Q5: What is the difference between `test.beforeEach()` and `test.beforeAll()`?

**Answer:**
```typescript
import { test } from '@playwright/test';

// beforeAll - runs ONCE before all tests in describe block
test.describe('User Tests', () => {
  test.beforeAll(async ({ browser }) => {
    // Setup: Runs once (e.g., seed database)
    console.log('Setting up test data...');
  });

  // beforeEach - runs BEFORE EACH test
  test.beforeEach(async ({ page }) => {
    // Runs before every single test
    await page.goto('/login');
  });

  test('test 1', async ({ page }) => {
    // beforeEach runs before this
  });

  test('test 2', async ({ page }) => {
    // beforeEach runs again before this
  });
});
```

**Use Cases:**
- `beforeAll`: Database seeding, expensive setup
- `beforeEach`: Navigation, login, resetting state

---

### Q6: How do you handle multiple tabs/windows in Playwright?

**Answer:**
```typescript
import { test, expect } from '@playwright/test';

test('handle multiple tabs', async ({ page, context }) => {
  await page.goto('/');

  // Method 1: Wait for new page event
  const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.locator('a[target="_blank"]').click()
  ]);

  await newPage.waitForLoadState();
  expect(await newPage.title()).toContain('New Tab');

  // Method 2: Get all pages
  const pages = context.pages();
  console.log(`Open pages: ${pages.length}`);

  // Switch between pages
  await pages[0].bringToFront();
  await pages[1].bringToFront();

  // Close specific page
  await newPage.close();
});
```

---

### Q7: What are fixtures in Playwright?

**Answer:**
Fixtures are test setup and teardown mechanisms. They provide isolated environments for each test.

**Built-in fixtures:**
- `page`: Isolated page instance
- `context`: Isolated browser context (like incognito)
- `browser`: Browser instance
- `request`: APIRequestContext for API testing

**Custom fixtures:**
```typescript
// fixtures.ts
import { test as base } from '@playwright/test';

type MyFixtures = {
  authenticatedPage: Page;
  testData: { username: string; password: string };
};

export const test = base.extend<MyFixtures>({
  // Page fixture that's already logged in
  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.fill('#username', 'testuser');
    await page.fill('#password', 'password123');
    await page.click('#submit');
    await page.waitForURL('/dashboard');

    await use(page);

    // Cleanup (logout)
    await page.click('#logout');
  },

  // Simple data fixture
  testData: async ({}, use) => {
    await use({
      username: 'testuser',
      password: 'password123'
    });
  }
});

// Usage in tests
test('dashboard shows user info', async ({ authenticatedPage }) => {
  // Already logged in!
  await expect(authenticatedPage.locator('.user-name')).toBeVisible();
});
```

---

## Intermediate Concepts

### Q8: How do you perform API testing in Playwright?

**Answer:**
```typescript
import { test, expect } from '@playwright/test';

test.describe('API Tests', () => {
  test('GET request', async ({ request }) => {
    const response = await request.get('/api/users/1');

    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(200);

    const user = await response.json();
    expect(user.id).toBe(1);
    expect(user.name).toBeTruthy();
  });

  test('POST request', async ({ request }) => {
    const newUser = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request.post('/api/users', {
      data: newUser,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    expect(response.ok()).toBeTruthy();

    const created = await response.json();
    expect(created.name).toBe(newUser.name);
  });

  test('Authentication with API', async ({ request }) => {
    // Login via API
    const loginResponse = await request.post('/api/auth/login', {
      data: { email: 'user@test.com', password: 'pass123' }
    });

    const { token } = await loginResponse.json();

    // Use token in subsequent requests
    const userResponse = await request.get('/api/users/me', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });

    expect(userResponse.ok()).toBeTruthy();
  });
});
```

**Combining UI and API:**
```typescript
test('create user via API, verify in UI', async ({ page, request }) => {
  // Setup via API (faster)
  const response = await request.post('/api/users', {
    data: { name: 'Test User', email: 'test@example.com' }
  });
  const user = await response.json();

  // Verify in UI
  await page.goto('/users');
  await expect(page.locator(`text=${user.name}`)).toBeVisible();
});
```

---

### Q9: How do you handle file uploads and downloads?

**Answer:**

**File Upload:**
```typescript
test('upload file', async ({ page }) => {
  await page.goto('/upload');

  // Method 1: Using input[type="file"]
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('./test-files/document.pdf');

  // Method 2: Multiple files
  await fileInput.setInputFiles([
    './test-files/file1.pdf',
    './test-files/file2.pdf'
  ]);

  // Method 3: From buffer
  await fileInput.setInputFiles({
    name: 'test.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from('file content')
  });

  // Submit
  await page.click('#upload-button');

  // Verify
  await expect(page.locator('.success-message')).toBeVisible();
});
```

**File Download:**
```typescript
test('download file', async ({ page }) => {
  await page.goto('/downloads');

  // Start waiting for download before clicking
  const downloadPromise = page.waitForEvent('download');
  await page.click('#download-button');
  const download = await downloadPromise;

  // Verify filename
  expect(download.suggestedFilename()).toBe('report.pdf');

  // Save to specific path
  const path = await download.path();
  await download.saveAs('./downloads/report.pdf');

  // Read and verify content
  const fs = require('fs');
  const content = fs.readFileSync('./downloads/report.pdf');
  expect(content.length).toBeGreaterThan(0);
});
```

---

### Q10: How do you intercept and mock network requests?

**Answer:**
```typescript
import { test, expect } from '@playwright/test';

test('mock API response', async ({ page }) => {
  // Mock specific API endpoint
  await page.route('/api/users', async (route) => {
    const json = [
      { id: 1, name: 'Mocked User 1' },
      { id: 2, name: 'Mocked User 2' }
    ];
    await route.fulfill({ json });
  });

  await page.goto('/users');

  // Verify mocked data appears
  await expect(page.locator('text=Mocked User 1')).toBeVisible();
});

test('modify API response', async ({ page }) => {
  await page.route('/api/config', async (route) => {
    // Get original response
    const response = await route.fetch();
    const json = await response.json();

    // Modify it
    json.featureFlag = true;

    // Return modified response
    await route.fulfill({ json });
  });

  await page.goto('/');
});

test('block specific resources', async ({ page }) => {
  // Block all images (faster tests)
  await page.route('**/*.{png,jpg,jpeg,svg}', route => route.abort());

  // Block analytics
  await page.route('**/analytics/**', route => route.abort());

  await page.goto('/');
});

test('spy on network requests', async ({ page }) => {
  const requests: string[] = [];

  page.on('request', request => {
    requests.push(request.url());
  });

  await page.goto('/');

  expect(requests).toContain('https://api.example.com/data');
});
```

---

### Q11: What is the Page Object Model and why use it?

**Answer:**
POM is a design pattern that creates an object repository for web elements, separating test logic from page-specific code.

**Benefits:**
- Code reusability
- Easier maintenance
- Better readability
- Single source of truth for selectors

**Implementation:**
```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign in' });
    this.errorMessage = page.locator('.error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorText(): Promise<string> {
    return await this.errorMessage.textContent() || '';
  }

  async isLoggedIn(): Promise<boolean> {
    return await this.page.url().includes('/dashboard');
  }
}

// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');

  expect(await loginPage.isLoggedIn()).toBeTruthy();
});

test('failed login shows error', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('wrong@example.com', 'wrongpass');

  const error = await loginPage.getErrorText();
  expect(error).toContain('Invalid credentials');
});
```

---

### Q12: How do you handle authentication state?

**Answer:**
```typescript
// auth.setup.ts - Run once to save auth state
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'user@example.com');
  await page.fill('#password', 'password123');
  await page.click('#submit');

  await page.waitForURL('/dashboard');

  // Save auth state
  await page.context().storageState({ path: 'auth.json' });
});

// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    // Use saved auth state
    storageState: 'auth.json'
  },
  // Run setup before all tests
  dependencies: ['authenticate']
});

// tests/dashboard.spec.ts
import { test, expect } from '@playwright/test';

test('view dashboard', async ({ page }) => {
  // Already authenticated!
  await page.goto('/dashboard');
  await expect(page.locator('.user-menu')).toBeVisible();
});

// For tests that need to be logged out
test.use({ storageState: { cookies: [], origins: [] } });
test('login page accessible when logged out', async ({ page }) => {
  await page.goto('/login');
  await expect(page).toHaveURL('/login');
});
```

---

### Q13: How do you run tests in parallel and manage test isolation?

**Answer:**
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Number of workers (parallel tests)
  workers: process.env.CI ? 2 : 4,

  // Fully parallel (default)
  fullyParallel: true,

  // Each test gets isolated context
  use: {
    // Each test gets fresh browser context (cookies, storage, etc.)
    contextOptions: {
      ignoreHTTPSErrors: true
    }
  }
});

// Serial execution for specific tests
test.describe.serial('checkout flow', () => {
  // These tests run in order, sharing state
  test('add item to cart', async ({ page }) => { });
  test('proceed to checkout', async ({ page }) => { });
  test('complete payment', async ({ page }) => { });
});

// Limit parallelism for specific tests
test.describe.configure({ mode: 'parallel', workers: 2 });

// Single worker for tests that modify shared state
test.describe.configure({ mode: 'serial' });
```

**Test Isolation Best Practices:**
```typescript
// Good - each test is independent
test('test 1', async ({ page }) => {
  await page.goto('/');
  // Test uses fresh page/context
});

test('test 2', async ({ page }) => {
  await page.goto('/');
  // New page/context, no state from test 1
});

// Bad - tests share state
let sharedPage: Page;

test.beforeAll(async ({ browser }) => {
  sharedPage = await browser.newPage();
});

test('test 1', async () => {
  await sharedPage.goto('/');
  // Modifies shared page
});

test('test 2', async () => {
  // Uses modified state from test 1 - flaky!
});
```

---

## Advanced Concepts

### Q14: How do you implement custom reporters?

**Answer:**
```typescript
// custom-reporter.ts
import { Reporter, TestCase, TestResult } from '@playwright/test/reporter';

class CustomReporter implements Reporter {
  onBegin(config, suite) {
    console.log(`Starting test run with ${suite.allTests().length} tests`);
  }

  onTestBegin(test: TestCase) {
    console.log(`Starting test: ${test.title}`);
  }

  onTestEnd(test: TestCase, result: TestResult) {
    console.log(`Finished test: ${test.title} - ${result.status}`);

    if (result.status === 'failed') {
      console.error(`  Error: ${result.error?.message}`);
    }
  }

  onEnd(result) {
    console.log(`Test run finished. Status: ${result.status}`);
  }
}

export default CustomReporter;

// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['./custom-reporter.ts'],
    ['html'],
    ['json', { outputFile: 'results.json' }]
  ]
});
```

**Slack/Teams Notification Reporter:**
```typescript
class SlackReporter implements Reporter {
  async onEnd(result) {
    const message = {
      text: `Test Run ${result.status}`,
      attachments: [{
        color: result.status === 'passed' ? 'good' : 'danger',
        fields: [
          { title: 'Passed', value: result.passed, short: true },
          { title: 'Failed', value: result.failed, short: true }
        ]
      }]
    };

    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: 'POST',
      body: JSON.stringify(message)
    });
  }
}
```

---

### Q15: How do you implement retry logic and flaky test handling?

**Answer:**
```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Global retry configuration
  retries: process.env.CI ? 2 : 0,

  // Timeout configurations
  timeout: 30000, // Test timeout
  expect: {
    timeout: 5000 // Assertion timeout
  },

  use: {
    // Action timeout
    actionTimeout: 10000,

    // Navigation timeout
    navigationTimeout: 30000,

    // Screenshot on failure
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure'
  }
});

// Per-test retry configuration
test('flaky test', async ({ page }) => {
  test.setTimeout(60000);

  // Custom retry logic
  await expect(async () => {
    const response = await page.request.get('/api/status');
    expect(response.ok()).toBeTruthy();
  }).toPass({
    intervals: [1000, 2000, 5000],
    timeout: 30000
  });
});

// Conditional skip for known flaky tests
test('potentially flaky', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Flaky on webkit');

  await page.goto('/');
});

// Soft assertions (continue on failure)
test('multiple checks', async ({ page }) => {
  await page.goto('/dashboard');

  // Continue even if these fail
  await expect.soft(page.locator('.user-name')).toBeVisible();
  await expect.soft(page.locator('.user-email')).toBeVisible();

  // This will still run even if above failed
  await expect(page.locator('.dashboard')).toBeVisible();
});
```

---

### Q16: How do you handle iframes?

**Answer:**
```typescript
test('interact with iframe', async ({ page }) => {
  await page.goto('/page-with-iframe');

  // Method 1: Using frameLocator (recommended)
  const iframe = page.frameLocator('iframe#myframe');
  await iframe.locator('#button-inside-iframe').click();

  // Method 2: Using frame()
  const frame = page.frame({ name: 'myframe' });
  if (frame) {
    await frame.locator('#button').click();
  }

  // Nested iframes
  const outerFrame = page.frameLocator('iframe#outer');
  const innerFrame = outerFrame.frameLocator('iframe#inner');
  await innerFrame.locator('#nested-button').click();

  // Wait for iframe to load
  await page.waitForLoadState('networkidle');
  const dynamicFrame = page.frameLocator('iframe.dynamic');
  await dynamicFrame.locator('#content').waitFor();
});
```

---

### Q17: How do you implement visual regression testing?

**Answer:**
```typescript
import { test, expect } from '@playwright/test';

test('visual regression - full page', async ({ page }) => {
  await page.goto('/');

  // Full page screenshot
  await expect(page).toHaveScreenshot('homepage.png', {
    fullPage: true,
    animations: 'disabled'
  });
});

test('visual regression - specific element', async ({ page }) => {
  await page.goto('/product/123');

  // Element screenshot
  const productCard = page.locator('.product-card');
  await expect(productCard).toHaveScreenshot('product-card.png', {
    maxDiffPixels: 100 // Allow small differences
  });
});

test('visual regression - masked areas', async ({ page }) => {
  await page.goto('/dashboard');

  // Mask dynamic content
  await expect(page).toHaveScreenshot('dashboard.png', {
    mask: [
      page.locator('.timestamp'),
      page.locator('.user-avatar')
    ]
  });
});

// playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 50,
      threshold: 0.2
    }
  }
});

// Update screenshots
// npm run test -- --update-snapshots
```

---

### Q18: How do you implement data-driven testing (DDT)?

**Answer:**
```typescript
// Method 1: Simple array
const testData = [
  { email: 'user1@test.com', password: 'pass1' },
  { email: 'user2@test.com', password: 'pass2' },
  { email: 'user3@test.com', password: 'pass3' }
];

for (const data of testData) {
  test(`login with ${data.email}`, async ({ page }) => {
    await page.goto('/login');
    await page.fill('#email', data.email);
    await page.fill('#password', data.password);
    await page.click('#submit');

    await expect(page).toHaveURL('/dashboard');
  });
}

// Method 2: From JSON file
import testUsers from './test-data/users.json';

for (const user of testUsers) {
  test(`user ${user.name} can login`, async ({ page }) => {
    // Test logic
  });
}

// Method 3: From CSV
import fs from 'fs';
import { parse } from 'csv-parse/sync';

const csvData = fs.readFileSync('./test-data/users.csv');
const users = parse(csvData, { columns: true });

for (const user of users) {
  test(`CSV user ${user.email}`, async ({ page }) => {
    // Test logic
  });
}

// Method 4: Parameterized tests with describe
test.describe('Login validation', () => {
  const invalidCredentials = [
    { email: '', password: 'pass', error: 'Email required' },
    { email: 'user@test.com', password: '', error: 'Password required' },
    { email: 'invalid', password: 'pass', error: 'Invalid email format' }
  ];

  for (const { email, password, error } of invalidCredentials) {
    test(`shows error: ${error}`, async ({ page }) => {
      await page.goto('/login');
      await page.fill('#email', email);
      await page.fill('#password', password);
      await page.click('#submit');

      await expect(page.locator('.error')).toHaveText(error);
    });
  }
});
```

---

### Q19: How do you debug Playwright tests?

**Answer:**
```typescript
// 1. Using --debug flag
// npx playwright test --debug

// 2. Using page.pause()
test('debug test', async ({ page }) => {
  await page.goto('/');

  // Pauses execution and opens inspector
  await page.pause();

  await page.click('#button');
});

// 3. Using console.log and trace
test('trace debugging', async ({ page }) => {
  console.log('Current URL:', page.url());

  const element = page.locator('#myElement');
  console.log('Element:', await element.textContent());

  // Take screenshot for debugging
  await page.screenshot({ path: 'debug-screenshot.png' });
});

// 4. Using traces
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry', // or 'on', 'retain-on-failure'
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  }
});

// View trace: npx playwright show-trace trace.zip

// 5. VS Code debugging
// .vscode/launch.json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Playwright Tests",
  "program": "${workspaceFolder}/node_modules/@playwright/test/cli.js",
  "args": ["test", "--headed", "--debug"]
}

// 6. Verbose logging
test('verbose logging', async ({ page }) => {
  // Enable verbose logging
  page.on('console', msg => console.log('PAGE LOG:', msg.text()));
  page.on('request', req => console.log('REQUEST:', req.url()));
  page.on('response', res => console.log('RESPONSE:', res.url(), res.status()));

  await page.goto('/');
});

// 7. Step-by-step with slowMo
// playwright.config.ts
export default defineConfig({
  use: {
    launchOptions: {
      slowMo: 1000 // Slow down by 1 second
    }
  }
});
```

---

### Q20: How do you handle complex user interactions (drag-drop, hover, etc.)?

**Answer:**
```typescript
test('drag and drop', async ({ page }) => {
  await page.goto('/drag-drop');

  // Method 1: Using dragTo
  const source = page.locator('#source');
  const target = page.locator('#target');
  await source.dragTo(target);

  // Method 2: Using mouse events
  const sourceBox = await source.boundingBox();
  const targetBox = await target.boundingBox();

  if (sourceBox && targetBox) {
    await page.mouse.move(
      sourceBox.x + sourceBox.width / 2,
      sourceBox.y + sourceBox.height / 2
    );
    await page.mouse.down();
    await page.mouse.move(
      targetBox.x + targetBox.width / 2,
      targetBox.y + targetBox.height / 2
    );
    await page.mouse.up();
  }
});

test('hover interactions', async ({ page }) => {
  await page.goto('/menu');

  // Hover to reveal submenu
  await page.locator('#menu-item').hover();
  await expect(page.locator('.submenu')).toBeVisible();

  // Hover with position
  await page.locator('#element').hover({
    position: { x: 10, y: 10 }
  });
});

test('double click', async ({ page }) => {
  await page.goto('/editor');

  await page.locator('#text').dblclick();
  await expect(page.locator('#text')).toHaveClass(/selected/);
});

test('right click context menu', async ({ page }) => {
  await page.goto('/editor');

  await page.locator('#element').click({ button: 'right' });
  await expect(page.locator('.context-menu')).toBeVisible();
});

test('keyboard interactions', async ({ page }) => {
  await page.goto('/editor');

  const input = page.locator('#editor');

  // Type text
  await input.type('Hello World');

  // Press keys
  await input.press('Control+A'); // Select all
  await input.press('Control+C'); // Copy
  await input.press('Control+V'); // Paste

  // Keyboard shortcuts
  await page.keyboard.press('Control+Shift+K');

  // Multiple keys
  await page.keyboard.down('Shift');
  await page.keyboard.press('ArrowDown');
  await page.keyboard.up('Shift');
});

test('scroll interactions', async ({ page }) => {
  await page.goto('/long-page');

  // Scroll to element
  await page.locator('#footer').scrollIntoViewIfNeeded();

  // Scroll by amount
  await page.mouse.wheel(0, 500);

  // Scroll to bottom
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
});
```

---

## Architecture & Design

### Q21: How would you structure a large Playwright test suite?

**Answer:**
```
project/
├── tests/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   └── signup.spec.ts
│   ├── e2e/
│   │   ├── checkout.spec.ts
│   │   └── user-journey.spec.ts
│   └── api/
│       ├── users.api.spec.ts
│       └── products.api.spec.ts
├── pages/
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   └── BasePage.ts
├── fixtures/
│   ├── custom-fixtures.ts
│   └── test-data-fixtures.ts
├── utils/
│   ├── helpers.ts
│   ├── constants.ts
│   └── api-client.ts
├── test-data/
│   ├── users.json
│   └── products.csv
├── config/
│   ├── playwright.config.ts
│   └── environments.ts
└── global-setup.ts
```

**BasePage pattern:**
```typescript
// pages/BasePage.ts
import { Page } from '@playwright/test';

export class BasePage {
  constructor(protected page: Page) {}

  async goto(path: string) {
    await this.page.goto(path);
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  async takeScreenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png` });
  }
}

// pages/LoginPage.ts
export class LoginPage extends BasePage {
  async login(email: string, password: string) {
    await this.page.fill('#email', email);
    await this.page.fill('#password', password);
    await this.page.click('#submit');
  }
}
```

---

### Q22: How do you manage different environments (dev, staging, prod)?

**Answer:**
```typescript
// config/environments.ts
export const environments = {
  dev: {
    baseURL: 'https://dev.example.com',
    apiURL: 'https://api-dev.example.com',
    timeout: 60000
  },
  staging: {
    baseURL: 'https://staging.example.com',
    apiURL: 'https://api-staging.example.com',
    timeout: 30000
  },
  prod: {
    baseURL: 'https://example.com',
    apiURL: 'https://api.example.com',
    timeout: 30000
  }
};

// playwright.config.ts
import { defineConfig } from '@playwright/test';
import { environments } from './config/environments';

const env = process.env.TEST_ENV || 'dev';
const config = environments[env];

export default defineConfig({
  use: {
    baseURL: config.baseURL,
    actionTimeout: config.timeout
  }
});

// Usage
// TEST_ENV=staging npx playwright test
// TEST_ENV=prod npx playwright test

// In tests
test('environment-aware test', async ({ page, baseURL }) => {
  await page.goto('/'); // Uses baseURL from config
  console.log('Testing against:', baseURL);
});
```

---

## AI/Copilot/MCP Integration

### Q23: How would you integrate AI/Copilot tools with Playwright testing?

**Answer:**

**1. GitHub Copilot for Test Generation:**
```typescript
// Prompt: "Generate Playwright test for login form validation"
// Copilot suggestion:

test('login form validation', async ({ page }) => {
  await page.goto('/login');

  // Test empty email
  await page.click('#submit');
  await expect(page.locator('.email-error')).toHaveText('Email is required');

  // Test invalid email
  await page.fill('#email', 'invalid-email');
  await page.click('#submit');
  await expect(page.locator('.email-error')).toHaveText('Invalid email format');

  // Test empty password
  await page.fill('#email', 'user@test.com');
  await page.click('#submit');
  await expect(page.locator('.password-error')).toHaveText('Password is required');
});
```

**2. AI-Powered Visual Testing:**
```typescript
import { test, expect } from '@playwright/test';
import { analyzeWithAI } from './utils/ai-vision';

test('AI visual analysis', async ({ page }) => {
  await page.goto('/');

  const screenshot = await page.screenshot();

  // Send to AI vision API (OpenAI, Google Cloud Vision, etc.)
  const analysis = await analyzeWithAI(screenshot, {
    prompt: 'Verify this page has a login button and logo'
  });

  expect(analysis.hasLoginButton).toBeTruthy();
  expect(analysis.hasLogo).toBeTruthy();
});
```

**3. AI Test Result Analysis:**
```typescript
// custom-ai-reporter.ts
import { Reporter, TestResult } from '@playwright/test/reporter';
import { analyzeFailureWithAI } from './utils/ai-analyzer';

class AIReporter implements Reporter {
  async onTestEnd(test, result: TestResult) {
    if (result.status === 'failed') {
      const analysis = await analyzeFailureWithAI({
        testName: test.title,
        error: result.error?.message,
        screenshot: result.attachments.find(a => a.name === 'screenshot'),
        trace: result.attachments.find(a => a.name === 'trace')
      });

      console.log('AI Analysis:', analysis.suggestedFix);
      console.log('Root Cause:', analysis.rootCause);
      console.log('Similar Issues:', analysis.similarIssues);
    }
  }
}
```

---

### Q24: What is MCP (Model Context Protocol) and how can it be used with Playwright?

**Answer:**
MCP is a protocol that allows AI models to interact with external tools and data sources. In testing context:

**Use Cases:**
1. **Dynamic Test Generation**: AI generates tests based on app state
2. **Smart Assertions**: AI determines expected behavior
3. **Autonomous Testing**: AI explores and tests app automatically
4. **Intelligent Debugging**: AI analyzes failures and suggests fixes

**Example Integration:**
```typescript
// mcp-test-generator.ts
import { MCPClient } from '@modelcontextprotocol/sdk';
import { test } from '@playwright/test';

class MCPTestGenerator {
  private mcp: MCPClient;

  constructor() {
    this.mcp = new MCPClient({
      model: 'claude-3-opus',
      tools: ['playwright', 'code-analysis']
    });
  }

  async generateTestsForPage(url: string): Promise<string> {
    const response = await this.mcp.query({
      prompt: `Analyze this page and generate comprehensive Playwright tests`,
      context: { url },
      tools: ['fetch-page', 'analyze-dom']
    });

    return response.generatedCode;
  }

  async suggestFix(error: Error, context: any): Promise<string> {
    const response = await this.mcp.query({
      prompt: `Analyze this test failure and suggest a fix`,
      context: { error, ...context },
      tools: ['code-analysis', 'documentation-search']
    });

    return response.suggestion;
  }
}

// Usage
const generator = new MCPTestGenerator();
const tests = await generator.generateTestsForPage('https://example.com/login');
console.log(tests);
```

**MCP for Self-Healing Tests:**
```typescript
class SelfHealingTest {
  async runWithHealing(testFn: () => Promise<void>, context: any) {
    try {
      await testFn();
    } catch (error) {
      // Ask MCP to analyze and fix
      const fix = await this.mcp.query({
        prompt: 'This test failed. Analyze and suggest a fix.',
        context: { error, testCode: testFn.toString(), ...context }
      });

      if (fix.canAutoFix) {
        // Apply fix and retry
        const fixedTest = eval(fix.fixedCode);
        await fixedTest();
      } else {
        throw error;
      }
    }
  }
}
```

---

### Q25: How would you implement AI-powered test maintenance?

**Answer:**
```typescript
// ai-test-maintainer.ts
import { Octokit } from '@octokit/rest';
import { AIAnalyzer } from './ai-analyzer';

class AITestMaintainer {
  private ai: AIAnalyzer;
  private github: Octokit;

  async analyzeFlakiness(testResults: TestResult[]) {
    const flakyTests = testResults.filter(t =>
      t.retries > 0 || t.failureRate > 0.1
    );

    for (const test of flakyTests) {
      const analysis = await this.ai.analyze({
        prompt: 'Why is this test flaky? Suggest improvements.',
        testCode: test.code,
        failureHistory: test.failures
      });

      // Create GitHub issue
      await this.github.issues.create({
        owner: 'myorg',
        repo: 'myrepo',
        title: `Flaky test: ${test.name}`,
        body: `
## AI Analysis
${analysis.rootCause}

## Suggested Fixes
${analysis.suggestions.map(s => `- ${s}`).join('\n')}

## Failure History
${test.failures.map(f => `- ${f.date}: ${f.reason}`).join('\n')}
        `,
        labels: ['flaky-test', 'ai-generated']
      });
    }
  }

  async suggestSelectorImprovements(page: Page) {
    const elements = await page.locator('*').all();

    for (const element of elements) {
      const currentSelector = element.selector;
      const suggestion = await this.ai.analyze({
        prompt: 'Suggest a more resilient selector',
        element: await element.innerHTML(),
        currentSelector
      });

      if (suggestion.improvement) {
        console.log(`Improve selector: ${currentSelector} → ${suggestion.better}`);
      }
    }
  }
}
```

**AI-Powered Accessibility Testing:**
```typescript
test('AI accessibility audit', async ({ page }) => {
  await page.goto('/');

  const screenshot = await page.screenshot();
  const html = await page.content();

  const audit = await aiClient.analyze({
    prompt: 'Audit this page for WCAG 2.1 AA compliance',
    screenshot,
    html,
    context: {
      guidelines: ['WCAG 2.1 AA', 'ADA compliance']
    }
  });

  console.log('Accessibility Issues:', audit.issues);
  console.log('Severity:', audit.severity);
  console.log('Recommendations:', audit.recommendations);

  // Assert critical issues
  expect(audit.criticalIssues).toHaveLength(0);
});
```

---

## Troubleshooting & Debugging

### Q26: How do you troubleshoot "Element not found" errors?

**Answer:**
```typescript
// Problem: Element not found

// Solution 1: Verify element exists
test('debug element not found', async ({ page }) => {
  await page.goto('/');

  // Check if element exists in DOM
  const count = await page.locator('#myElement').count();
  console.log('Element count:', count);

  // Wait for element
  await page.locator('#myElement').waitFor({ state: 'visible', timeout: 10000 });

  // Check element state
  const isVisible = await page.locator('#myElement').isVisible();
  console.log('Is visible:', isVisible);
});

// Solution 2: Wait for network idle
test('wait for dynamic content', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');

  // Now element should be loaded
  await page.locator('#dynamic-element').click();
});

// Solution 3: Use better selectors
test('resilient selectors', async ({ page }) => {
  // Bad - fragile
  await page.locator('div > div > span.class1.class2').click();

  // Good - semantic
  await page.getByRole('button', { name: 'Submit' }).click();
  await page.getByLabel('Email').fill('user@test.com');
  await page.locator('[data-testid="submit-btn"]').click();
});

// Solution 4: Debug with trace
test('debug with trace', async ({ page }) => {
  await page.goto('/');

  try {
    await page.locator('#missing').click({ timeout: 5000 });
  } catch (error) {
    // Take screenshot
    await page.screenshot({ path: 'error-state.png', fullPage: true });

    // Get page state
    console.log('URL:', page.url());
    console.log('HTML:', await page.content());

    throw error;
  }
});
```

---

### Q27: How do you handle timing issues and race conditions?

**Answer:**
```typescript
// Problem: Race conditions

// Solution 1: Use auto-waiting
test('auto-waiting handles timing', async ({ page }) => {
  await page.goto('/');

  // Playwright automatically waits for element to be:
  // - attached, visible, stable, receives events, enabled
  await page.locator('#button').click();
});

// Solution 2: Wait for specific network requests
test('wait for API call', async ({ page }) => {
  await page.goto('/');

  // Wait for specific API response
  const responsePromise = page.waitForResponse(
    resp => resp.url().includes('/api/data') && resp.status() === 200
  );

  await page.click('#load-data');

  const response = await responsePromise;
  const data = await response.json();
  expect(data).toBeTruthy();
});

// Solution 3: Wait for function to return truthy
test('wait for condition', async ({ page }) => {
  await page.goto('/dashboard');

  // Wait for specific condition
  await page.waitForFunction(() => {
    const elements = document.querySelectorAll('.data-item');
    return elements.length >= 10;
  }, { timeout: 10000 });
});

// Solution 4: Use expect.toPass() for retry logic
test('retry until condition met', async ({ page }) => {
  await page.goto('/');

  // Retries until passes or timeout
  await expect(async () => {
    const count = await page.locator('.item').count();
    expect(count).toBeGreaterThan(5);
  }).toPass({
    intervals: [1000, 2000, 3000],
    timeout: 10000
  });
});

// Solution 5: Wait for multiple events
test('wait for multiple conditions', async ({ page }) => {
  await page.goto('/');

  await Promise.all([
    page.waitForLoadState('networkidle'),
    page.waitForSelector('#content', { state: 'visible' }),
    page.waitForResponse(resp => resp.url().includes('/api/init'))
  ]);

  // Now safe to interact
  await page.click('#button');
});
```

---

### Q28: How would you optimize slow test execution?

**Answer:**
```typescript
// 1. Parallel execution
// playwright.config.ts
export default defineConfig({
  workers: process.env.CI ? 4 : 8,
  fullyParallel: true
});

// 2. Reuse authentication
// auth.setup.ts
test('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'user@test.com');
  await page.fill('#password', 'pass123');
  await page.click('#submit');
  await page.context().storageState({ path: 'auth.json' });
});

// 3. Use API for setup
test('fast setup with API', async ({ page, request }) => {
  // Create test data via API (faster)
  await request.post('/api/users', {
    data: { name: 'Test User' }
  });

  // Then verify in UI
  await page.goto('/users');
  await expect(page.locator('text=Test User')).toBeVisible();
});

// 4. Block unnecessary resources
test.use({
  async context({ context }, use) {
    // Block images, fonts, etc.
    await context.route('**/*.{png,jpg,jpeg,svg,woff,woff2}', route => route.abort());
    await use(context);
  }
});

// 5. Use page.evaluate for bulk checks
test('bulk validation', async ({ page }) => {
  await page.goto('/products');

  // Instead of multiple individual checks
  const results = await page.evaluate(() => {
    const products = Array.from(document.querySelectorAll('.product'));
    return products.map(p => ({
      hasImage: !!p.querySelector('img'),
      hasPrice: !!p.querySelector('.price'),
      hasButton: !!p.querySelector('button')
    }));
  });

  results.forEach(r => {
    expect(r.hasImage).toBeTruthy();
    expect(r.hasPrice).toBeTruthy();
    expect(r.hasButton).toBeTruthy();
  });
});

// 6. Shard tests across machines
// npx playwright test --shard=1/4
// npx playwright test --shard=2/4
// npx playwright test --shard=3/4
// npx playwright test --shard=4/4
```

---

### Q29: What are common Playwright anti-patterns to avoid?

**Answer:**

**1. Using fixed waits (sleep)**
```typescript
// Bad
await page.click('#button');
await page.waitForTimeout(3000); // Never do this!
await page.locator('#result').click();

// Good
await page.click('#button');
await page.locator('#result').waitFor({ state: 'visible' });
await page.locator('#result').click();
```

**2. Not using auto-waiting**
```typescript
// Bad
const element = await page.$('#button');
if (element) {
  await element.click();
}

// Good
await page.locator('#button').click(); // Auto-waits
```

**3. Using brittle selectors**
```typescript
// Bad
page.locator('body > div:nth-child(3) > div > span');

// Good
page.getByRole('button', { name: 'Submit' });
page.locator('[data-testid="submit-btn"]');
```

**4. Not isolating tests**
```typescript
// Bad - tests depend on each other
test('test 1', async ({ page }) => {
  globalState.userId = await createUser();
});

test('test 2', async ({ page }) => {
  await loginWithUser(globalState.userId); // Fails if test 1 skipped
});

// Good - each test is independent
test('test 1', async ({ page }) => {
  const userId = await createUser();
  // Use userId only in this test
});

test('test 2', async ({ page }) => {
  const userId = await createUser(); // Create own data
  await loginWithUser(userId);
});
```

**5. Not using Page Object Model**
```typescript
// Bad - duplicated selectors
test('test 1', async ({ page }) => {
  await page.fill('#email', 'user@test.com');
  await page.fill('#password', 'pass');
});

test('test 2', async ({ page }) => {
  await page.fill('#email', 'other@test.com');
  await page.fill('#password', 'pass');
});

// Good - use Page Objects
class LoginPage {
  constructor(private page: Page) {}

  async login(email: string, password: string) {
    await this.page.fill('#email', email);
    await this.page.fill('#password', password);
  }
}
```

---

### Q30: How do you implement CI/CD integration for Playwright tests?

**Answer:**
```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    strategy:
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]

    steps:
    - uses: actions/checkout@v3

    - uses: actions/setup-node@v3
      with:
        node-version: 18

    - name: Install dependencies
      run: npm ci

    - name: Install Playwright Browsers
      run: npx playwright install --with-deps

    - name: Run Playwright tests
      run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
      env:
        TEST_ENV: staging

    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report-${{ matrix.shardIndex }}
        path: playwright-report/
        retention-days: 30

    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results-${{ matrix.shardIndex }}
        path: test-results/
        retention-days: 30

  merge-reports:
    if: always()
    needs: [test]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: 18

    - name: Download all reports
      uses: actions/download-artifact@v3
      with:
        path: all-reports/

    - name: Merge reports
      run: npx playwright merge-reports --reporter html ./all-reports

    - name: Upload merged report
      uses: actions/upload-artifact@v3
      with:
        name: merged-html-report
        path: playwright-report/

    - name: Comment PR with results
      if: github.event_name == 'pull_request'
      uses: actions/github-script@v6
      with:
        script: |
          const report = require('./test-results.json');
          const comment = `
          ## Playwright Test Results
          - ✅ Passed: ${report.passed}
          - ❌ Failed: ${report.failed}
          - ⏭️ Skipped: ${report.skipped}
          `;

          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: comment
          });
```

**Docker setup:**
```dockerfile
# Dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

**Jenkins pipeline:**
```groovy
pipeline {
  agent {
    docker {
      image 'mcr.microsoft.com/playwright:v1.40.0'
    }
  }

  stages {
    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npx playwright test'
      }
    }

    stage('Publish Report') {
      steps {
        publishHTML([
          reportDir: 'playwright-report',
          reportFiles: 'index.html',
          reportName: 'Playwright Report'
        ])
      }
    }
  }

  post {
    always {
      junit 'test-results/*.xml'
      archiveArtifacts artifacts: 'test-results/**/*'
    }
  }
}
```

---

## Bonus Questions

### Q31: How do you test PWAs (Progressive Web Apps) with Playwright?

**Answer:**
```typescript
test('PWA installation', async ({ page, context }) => {
  await page.goto('/');

  // Wait for service worker
  await page.waitForEvent('serviceworker');

  // Check manifest
  const manifest = await page.evaluate(() => {
    const link = document.querySelector('link[rel="manifest"]');
    return link?.getAttribute('href');
  });
  expect(manifest).toBeTruthy();

  // Test offline mode
  await context.setOffline(true);
  await page.reload();

  // Should still load (service worker cache)
  await expect(page.locator('h1')).toBeVisible();

  // Back online
  await context.setOffline(false);
});

test('push notifications', async ({ page, context }) => {
  // Grant notification permission
  await context.grantPermissions(['notifications']);

  await page.goto('/');

  // Trigger notification
  await page.click('#enable-notifications');

  // Listen for notification
  page.on('notification', notification => {
    expect(notification.title()).toBe('Welcome!');
  });
});
```

### Q32: How do you handle mobile/responsive testing?

**Answer:**
```typescript
// playwright.config.ts
import { devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'Desktop Chrome',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 13'] }
    },
    {
      name: 'Tablet',
      use: { ...devices['iPad Pro'] }
    }
  ]
});

// Custom viewport test
test('responsive design', async ({ page }) => {
  // Desktop
  await page.setViewportSize({ width: 1920, height: 1080 });
  await page.goto('/');
  await expect(page.locator('.desktop-menu')).toBeVisible();

  // Mobile
  await page.setViewportSize({ width: 375, height: 667 });
  await expect(page.locator('.mobile-menu')).toBeVisible();
  await expect(page.locator('.desktop-menu')).toBeHidden();
});

// Mobile gestures
test('mobile swipe', async ({ page }) => {
  await page.goto('/carousel');

  const carousel = page.locator('.carousel');
  const box = await carousel.boundingBox();

  if (box) {
    // Swipe left
    await page.touchscreen.swipe(
      { x: box.x + box.width - 10, y: box.y + box.height / 2 },
      { x: box.x + 10, y: box.y + box.height / 2 }
    );
  }

  await expect(page.locator('.carousel-item-2')).toBeVisible();
});
```

---

## Summary: Key Takeaways

1. **Auto-waiting** is Playwright's superpower - use it
2. **Locator API** over ElementHandle
3. **Page Object Model** for maintainability
4. **Fixtures** for reusable setup
5. **Parallel execution** for speed
6. **API testing** for fast setup
7. **Network interception** for mocking
8. **Traces** for debugging
9. **TypeScript** for type safety
10. **CI/CD integration** for automation

---

## Additional Resources

- [Playwright Official Docs](https://playwright.dev/)
- [Playwright GitHub](https://github.com/microsoft/playwright)
- [Playwright Discord Community](https://discord.com/invite/playwright)
- [Awesome Playwright](https://github.com/mxschmitt/awesome-playwright)
- [Playwright Solutions](https://playwright.solutions/)

---

Good luck with your Playwright interviews! 🎭🚀
