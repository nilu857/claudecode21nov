# TypeScript Learning Guide for Playwright Testing

## Table of Contents
1. [Basic TypeScript Concepts](#basic-typescript-concepts)
2. [Functions in TypeScript](#functions-in-typescript)
3. [Async/Await Explained](#asyncawait-explained)
4. [TypeScript with Playwright](#typescript-with-playwright)
5. [Best Practices](#best-practices)

---

## Basic TypeScript Concepts

### What is TypeScript?
TypeScript is JavaScript with **type safety**. It helps catch errors before runtime by checking types during development.

### Type Annotations
```typescript
// Basic types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let items: string[] = ["apple", "banana"];
let numbers: number[] = [1, 2, 3];

// Object types
let user: { name: string; age: number } = {
  name: "Alice",
  age: 25
};

// Any (avoid when possible)
let anything: any = "can be anything";
```

**Memory Tip**: Think of `: type` as a label that says "this box only holds this type of thing"

### Interfaces
Interfaces define the structure of objects:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isAdmin?: boolean;  // ? means optional
}

const user: User = {
  id: 1,
  name: "Bob",
  email: "bob@example.com"
};
```

**Memory Tip**: Interface = Contract. "This object must look like this"

### Type Aliases
Similar to interfaces but more flexible:

```typescript
type ID = string | number;  // Union type
type UserRole = "admin" | "user" | "guest";  // Literal type

let userId: ID = 123;  // or "abc123"
let role: UserRole = "admin";
```

---

## Functions in TypeScript

### Basic Function Syntax

```typescript
// Function with typed parameters and return type
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => {
  return a * b;
};

// Concise arrow function
const subtract = (a: number, b: number): number => a - b;
```

**Memory Tip**:
```
function name(param: ParamType): ReturnType { }
         ↑         ↑              ↑
       name    input type    output type
```

### Function with Optional and Default Parameters

```typescript
// Optional parameter (?)
function greet(name: string, greeting?: string): string {
  return greeting ? `${greeting}, ${name}` : `Hello, ${name}`;
}

// Default parameter
function greetWithDefault(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}`;
}

greet("Alice");  // "Hello, Alice"
greet("Alice", "Hi");  // "Hi, Alice"
```

### Function Types

```typescript
// Function as a type
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const multiply: MathOperation = (a, b) => a * b;

// Function accepting another function
function calculate(a: number, b: number, operation: MathOperation): number {
  return operation(a, b);
}

calculate(5, 3, add);  // 8
```

### Void and Never

```typescript
// void: function returns nothing
function logMessage(message: string): void {
  console.log(message);
}

// never: function never returns (throws or infinite loop)
function throwError(message: string): never {
  throw new Error(message);
}
```

**Memory Tip**:
- `void` = "I return nothing useful"
- `never` = "I never return at all"

---

## Async/Await Explained

### Why Do We Use Async/Await?

**Problem**: JavaScript operations like API calls, file reading, or waiting for page loads take time. We don't want to freeze the entire program while waiting.

**Solution**: Async/await allows us to write asynchronous code that looks synchronous (easier to read).

### Promises Basics

Before async/await, we used Promises:

```typescript
// Old way with .then()
function fetchUserOld(id: number): Promise<User> {
  return fetch(`/api/user/${id}`)
    .then(response => response.json())
    .then(data => data as User);
}
```

### Async/Await Syntax

```typescript
// Modern way with async/await
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/user/${id}`);
  const data = await response.json();
  return data as User;
}
```

**Key Rules**:
1. `async` keyword before function → function returns a Promise
2. `await` keyword → "wait for this Promise to complete"
3. `await` can only be used inside `async` functions

**Memory Tip**:
- `async` = "This function does waiting"
- `await` = "Wait here until done"

### Error Handling with Async/Await

```typescript
async function fetchUserSafe(id: number): Promise<User | null> {
  try {
    const response = await fetch(`/api/user/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data as User;
  } catch (error) {
    console.error("Failed to fetch user:", error);
    return null;
  }
}
```

### Multiple Async Operations

```typescript
// Sequential (one after another)
async function sequentialFetch() {
  const user = await fetchUser(1);      // Wait for this
  const posts = await fetchPosts(1);    // Then wait for this
  return { user, posts };
}

// Parallel (at the same time - faster!)
async function parallelFetch() {
  const [user, posts] = await Promise.all([
    fetchUser(1),
    fetchPosts(1)
  ]);
  return { user, posts };
}
```

**Memory Tip**:
- Sequential = One line at a time (slower but dependent)
- Parallel = All at once (faster when independent)

---

## TypeScript with Playwright

### Setting Up Types

Playwright has excellent TypeScript support built-in. Types are imported automatically:

```typescript
import { test, expect, Page, Locator } from '@playwright/test';
```

### Basic Test Structure

```typescript
import { test, expect } from '@playwright/test';

test('basic test example', async ({ page }) => {
  // Navigate to page
  await page.goto('https://example.com');

  // Wait for element and verify
  await expect(page.locator('h1')).toHaveText('Example Domain');
});
```

**Why async/await here?**
- `page.goto()` waits for page to load
- `page.locator()` waits for element to appear
- All these are asynchronous operations

### Typed Page Objects

```typescript
// loginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('#email');
    this.passwordInput = page.locator('#password');
    this.submitButton = page.locator('button[type="submit"]');
  }

  async goto(): Promise<void> {
    await this.page.goto('/login');
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage(): Promise<string> {
    const error = this.page.locator('.error-message');
    return await error.textContent() || '';
  }
}
```

**Usage in test**:
```typescript
test('user can login', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');

  await expect(page).toHaveURL('/dashboard');
});
```

**Memory Tip for Page Objects**:
- `readonly page: Page` = "This class wraps a page"
- `Locator` = "This is an element I can interact with"
- All interaction methods are `async` = "They wait for things"

### Custom Fixtures with Types

```typescript
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/loginPage';

type MyFixtures = {
  loginPage: LoginPage;
  authenticatedPage: Page;
};

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  authenticatedPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('test@example.com', 'password');
    await use(page);
  }
});

export { expect } from '@playwright/test';
```

**Usage**:
```typescript
import { test, expect } from './fixtures';

test('user sees dashboard after login', async ({ authenticatedPage }) => {
  // Already logged in!
  await expect(authenticatedPage).toHaveURL('/dashboard');
});
```

### API Testing with TypeScript

```typescript
import { test, expect } from '@playwright/test';

interface User {
  id: number;
  name: string;
  email: string;
}

test('API returns user data', async ({ request }) => {
  const response = await request.get('/api/users/1');

  expect(response.ok()).toBeTruthy();

  const user: User = await response.json();

  expect(user.id).toBe(1);
  expect(user.name).toBeTruthy();
  expect(user.email).toContain('@');
});

test('API creates user', async ({ request }) => {
  const newUser = {
    name: 'John Doe',
    email: 'john@example.com'
  };

  const response = await request.post('/api/users', {
    data: newUser
  });

  expect(response.ok()).toBeTruthy();

  const createdUser: User = await response.json();
  expect(createdUser.name).toBe(newUser.name);
  expect(createdUser.email).toBe(newUser.email);
});
```

### Hooks with TypeScript

```typescript
import { test, expect, Page } from '@playwright/test';

test.describe('User Dashboard', () => {
  let page: Page;

  test.beforeAll(async ({ browser }) => {
    page = await browser.newPage();
    await page.goto('/login');
    // Perform login once for all tests
  });

  test.beforeEach(async () => {
    await page.goto('/dashboard');
  });

  test.afterEach(async () => {
    // Clean up after each test
    await page.evaluate(() => localStorage.clear());
  });

  test.afterAll(async () => {
    await page.close();
  });

  test('displays user name', async () => {
    const userName = await page.locator('.user-name').textContent();
    expect(userName).toBeTruthy();
  });
});
```

### Working with Complex Selectors

```typescript
import { test, expect, Locator } from '@playwright/test';

test('complex interactions', async ({ page }) => {
  await page.goto('/products');

  // Type-safe selectors
  const productCard: Locator = page.locator('[data-testid="product-card"]').first();
  const addToCartButton: Locator = productCard.locator('button:has-text("Add to Cart")');

  await addToCartButton.click();

  // Wait for multiple conditions
  await Promise.all([
    expect(page.locator('.cart-count')).toHaveText('1'),
    expect(page.locator('.notification')).toBeVisible()
  ]);
});
```

### Custom Matchers and Assertions

```typescript
// custom-matchers.ts
import { expect as baseExpect } from '@playwright/test';

export const expect = baseExpect.extend({
  async toHaveValidationError(locator: Locator, expected: string) {
    const errorElement = locator.locator('~ .error-message');
    const errorText = await errorElement.textContent();

    const pass = errorText === expected;

    return {
      message: () => `expected validation error "${expected}", got "${errorText}"`,
      pass
    };
  }
});
```

**Usage**:
```typescript
import { test } from '@playwright/test';
import { expect } from './custom-matchers';

test('shows validation error', async ({ page }) => {
  await page.goto('/signup');

  const emailInput = page.locator('#email');
  await emailInput.fill('invalid-email');
  await emailInput.blur();

  await expect(emailInput).toHaveValidationError('Please enter a valid email');
});
```

---

## Best Practices

### 1. Use Explicit Types
```typescript
// Bad
const user = await getUser();

// Good
const user: User = await getUser();
```

### 2. Avoid 'any'
```typescript
// Bad
function processData(data: any) { }

// Good
function processData(data: unknown) {
  if (typeof data === 'string') {
    // TypeScript knows data is string here
  }
}
```

### 3. Use Type Guards
```typescript
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object'
    && obj !== null
    && 'id' in obj
    && 'name' in obj;
}

const data: unknown = await response.json();
if (isUser(data)) {
  // TypeScript knows data is User here
  console.log(data.name);
}
```

### 4. Prefer Interfaces for Objects, Types for Unions
```typescript
// Use interface for objects
interface User {
  id: number;
  name: string;
}

// Use type for unions and primitives
type Status = 'pending' | 'active' | 'inactive';
type ID = string | number;
```

### 5. Use Async/Await for Readability
```typescript
// Harder to read
function complexOperation() {
  return fetchUser()
    .then(user => fetchPosts(user.id))
    .then(posts => processPosts(posts))
    .catch(error => handleError(error));
}

// Easier to read
async function complexOperation() {
  try {
    const user = await fetchUser();
    const posts = await fetchPosts(user.id);
    return await processPosts(posts);
  } catch (error) {
    handleError(error);
  }
}
```

### 6. Type Your Test Data
```typescript
// testData.ts
interface TestUser {
  email: string;
  password: string;
  name: string;
}

export const testUsers: Record<string, TestUser> = {
  admin: {
    email: 'admin@test.com',
    password: 'admin123',
    name: 'Admin User'
  },
  regularUser: {
    email: 'user@test.com',
    password: 'user123',
    name: 'Regular User'
  }
};
```

### 7. Use Generics for Reusable Code
```typescript
async function waitForCondition<T>(
  fn: () => Promise<T>,
  condition: (value: T) => boolean,
  timeout: number = 5000
): Promise<T> {
  const startTime = Date.now();

  while (Date.now() - startTime < timeout) {
    const value = await fn();
    if (condition(value)) {
      return value;
    }
    await new Promise(resolve => setTimeout(resolve, 100));
  }

  throw new Error('Timeout waiting for condition');
}

// Usage
const count = await waitForCondition(
  () => page.locator('.item').count(),
  count => count > 5
);
```

---

## Quick Reference Card

### Function Syntax
```typescript
function name(param: Type): ReturnType { }
const arrow = (param: Type): ReturnType => { };
async function asyncFn(): Promise<Type> { }
```

### Common Types
```typescript
string, number, boolean, null, undefined
string[], Array<string>
{ key: Type }
Type | OtherType  // union
Type & OtherType  // intersection
```

### Async/Await Pattern
```typescript
async function fn() {
  try {
    const result = await asyncOperation();
    return result;
  } catch (error) {
    // handle error
  }
}
```

### Playwright Essentials
```typescript
await page.goto(url);
await page.locator(selector).click();
await page.locator(selector).fill(text);
await expect(locator).toBeVisible();
await expect(locator).toHaveText(text);
```

---

## Practice Exercises

### Exercise 1: Type a Function
Convert this JavaScript to TypeScript:
```javascript
function calculateDiscount(price, discountPercent) {
  return price * (1 - discountPercent / 100);
}
```

<details>
<summary>Solution</summary>

```typescript
function calculateDiscount(price: number, discountPercent: number): number {
  return price * (1 - discountPercent / 100);
}
```
</details>

### Exercise 2: Create a Page Object
Create a typed page object for a search page with:
- Search input
- Search button
- Results list

<details>
<summary>Solution</summary>

```typescript
import { Page, Locator } from '@playwright/test';

export class SearchPage {
  readonly page: Page;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly resultsList: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchInput = page.locator('#search-input');
    this.searchButton = page.locator('#search-button');
    this.resultsList = page.locator('.results-list');
  }

  async goto(): Promise<void> {
    await this.page.goto('/search');
  }

  async search(query: string): Promise<void> {
    await this.searchInput.fill(query);
    await this.searchButton.click();
  }

  async getResultCount(): Promise<number> {
    return await this.resultsList.locator('.result-item').count();
  }
}
```
</details>

### Exercise 3: Async Data Fetching
Write an async function that fetches multiple users in parallel and returns only active users.

<details>
<summary>Solution</summary>

```typescript
interface User {
  id: number;
  name: string;
  isActive: boolean;
}

async function fetchActiveUsers(userIds: number[]): Promise<User[]> {
  const userPromises = userIds.map(id => fetchUser(id));
  const users = await Promise.all(userPromises);
  return users.filter(user => user.isActive);
}

async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return await response.json();
}
```
</details>

---

## Common Mistakes and How to Avoid Them

### 1. Forgetting to await
```typescript
// Wrong - returns Promise, not the value
const user = getUserAsync();

// Correct
const user = await getUserAsync();
```

### 2. Not typing function returns
```typescript
// Bad - TypeScript infers, but explicit is better
async function getUser(id) {
  return await fetch(`/users/${id}`).then(r => r.json());
}

// Good
async function getUser(id: number): Promise<User> {
  const response = await fetch(`/users/${id}`);
  return await response.json();
}
```

### 3. Using any instead of unknown
```typescript
// Bad - defeats TypeScript's purpose
function handle(data: any) {
  console.log(data.name);  // No type checking
}

// Good - forces type checking
function handle(data: unknown) {
  if (typeof data === 'object' && data !== null && 'name' in data) {
    console.log((data as { name: string }).name);
  }
}
```

---

## Additional Resources

- [TypeScript Official Docs](https://www.typescriptlang.org/docs/)
- [Playwright TypeScript Guide](https://playwright.dev/docs/test-typescript)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Effective TypeScript Book](https://effectivetypescript.com/)

Happy coding! 🚀
