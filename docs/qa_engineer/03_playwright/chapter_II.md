# Chapter II - Writing Tests
`(avr. time for this chapter: 2 days)`

This chapter covers advanced Playwright patterns including fixtures, page object model, API testing, and test organization.

## Test Fixtures

Fixtures provide isolated, reusable test setup and teardown.

### Built-in Fixtures

```typescript
import { test, expect } from '@playwright/test';

test('uses built-in fixtures', async ({ page, context, browser, request }) => {
  // page - isolated page for each test
  // context - isolated browser context
  // browser - shared browser instance
  // request - API request context
});
```

### Custom Fixtures

Create `tests/fixtures.ts`:

```typescript
import { test as base, expect } from '@playwright/test';

// Define fixture types
type MyFixtures = {
  authenticatedPage: Page;
  testUser: { email: string; password: string };
  todoPage: TodoPage;
};

// Extend base test with custom fixtures
export const test = base.extend<MyFixtures>({
  // Simple fixture
  testUser: async ({}, use) => {
    await use({
      email: 'test@example.com',
      password: 'password123'
    });
  },

  // Fixture with setup and teardown
  authenticatedPage: async ({ page }, use) => {
    // Setup: Login
    await page.goto('/login');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Login' }).click();
    await page.waitForURL('/dashboard');

    // Use the authenticated page in test
    await use(page);

    // Teardown: Logout
    await page.getByRole('button', { name: 'Logout' }).click();
  },

  // Page object fixture
  todoPage: async ({ page }, use) => {
    const todoPage = new TodoPage(page);
    await todoPage.goto();
    await use(todoPage);
  },
});

export { expect };
```

### Using Custom Fixtures

```typescript
import { test, expect } from './fixtures';

test('uses authenticated page', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/profile');
  await expect(authenticatedPage.getByRole('heading')).toHaveText('Profile');
});

test('uses test user data', async ({ page, testUser }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(testUser.email);
  await page.getByLabel('Password').fill(testUser.password);
});
```

## Page Object Model

Organize tests with page objects for maintainability.

### Creating Page Objects

Create `tests/pages/LoginPage.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.loginButton = page.getByRole('button', { name: 'Login' });
    this.errorMessage = page.getByTestId('error-message');
    this.successMessage = page.getByTestId('success-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async expectError(message?: string) {
    await expect(this.errorMessage).toBeVisible();
    if (message) {
      await expect(this.errorMessage).toContainText(message);
    }
  }

  async expectSuccess() {
    await expect(this.successMessage).toBeVisible();
  }
}
```

Create `tests/pages/DashboardPage.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class DashboardPage {
  readonly page: Page;
  readonly welcomeMessage: Locator;
  readonly userMenu: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.welcomeMessage = page.getByTestId('welcome-message');
    this.userMenu = page.getByTestId('user-menu');
    this.logoutButton = page.getByRole('button', { name: 'Logout' });
  }

  async goto() {
    await this.page.goto('/dashboard');
  }

  async logout() {
    await this.userMenu.click();
    await this.logoutButton.click();
  }

  async expectWelcomeMessage(name: string) {
    await expect(this.welcomeMessage).toContainText(`Welcome, ${name}`);
  }
}
```

### Using Page Objects

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { DashboardPage } from './pages/DashboardPage';

test.describe('Authentication', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
  });

  test('successful login redirects to dashboard', async () => {
    await loginPage.goto();
    await loginPage.login('test@example.com', 'password123');
    await dashboardPage.expectWelcomeMessage('Test User');
  });

  test('invalid credentials show error', async () => {
    await loginPage.goto();
    await loginPage.login('wrong@example.com', 'wrongpassword');
    await loginPage.expectError('Invalid credentials');
  });

  test('logout returns to login page', async ({ page }) => {
    await loginPage.goto();
    await loginPage.login('test@example.com', 'password123');
    await dashboardPage.logout();
    await expect(page).toHaveURL('/login');
  });
});
```

## API Testing

Playwright provides powerful API testing capabilities.

### Basic API Requests

```typescript
import { test, expect } from '@playwright/test';

test.describe('API Tests', () => {
  test('GET request', async ({ request }) => {
    const response = await request.get('/api/users');
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(200);
    
    const users = await response.json();
    expect(users).toBeInstanceOf(Array);
    expect(users.length).toBeGreaterThan(0);
  });

  test('POST request', async ({ request }) => {
    const response = await request.post('/api/users', {
      data: {
        name: 'John Doe',
        email: 'john@example.com'
      }
    });

    expect(response.status()).toBe(201);
    
    const user = await response.json();
    expect(user.id).toBeDefined();
    expect(user.name).toBe('John Doe');
  });

  test('PUT request', async ({ request }) => {
    const response = await request.put('/api/users/1', {
      data: {
        name: 'Jane Doe'
      }
    });

    expect(response.ok()).toBeTruthy();
    
    const user = await response.json();
    expect(user.name).toBe('Jane Doe');
  });

  test('DELETE request', async ({ request }) => {
    const response = await request.delete('/api/users/1');
    expect(response.status()).toBe(200);
  });
});
```

### API Authentication

```typescript
test.describe('Authenticated API', () => {
  let authToken: string;

  test.beforeAll(async ({ request }) => {
    const response = await request.post('/api/auth/login', {
      data: {
        email: 'test@example.com',
        password: 'password123'
      }
    });
    const data = await response.json();
    authToken = data.token;
  });

  test('access protected endpoint', async ({ request }) => {
    const response = await request.get('/api/profile', {
      headers: {
        'Authorization': `Bearer ${authToken}`
      }
    });

    expect(response.ok()).toBeTruthy();
    const profile = await response.json();
    expect(profile.email).toBe('test@example.com');
  });
});
```

### Combining API and UI Tests

```typescript
test('create user via API and verify in UI', async ({ page, request }) => {
  // Create user via API
  const response = await request.post('/api/users', {
    data: {
      name: 'New User',
      email: 'newuser@example.com'
    }
  });
  const user = await response.json();

  // Verify in UI
  await page.goto('/users');
  await expect(page.getByText('New User')).toBeVisible();
  await expect(page.getByText('newuser@example.com')).toBeVisible();
});

test('login via API and access dashboard', async ({ page, request }) => {
  // Login via API
  const response = await request.post('/api/auth/login', {
    data: {
      email: 'test@example.com',
      password: 'password123'
    }
  });
  const { token } = await response.json();

  // Set token in browser storage
  await page.goto('/');
  await page.evaluate((token) => {
    localStorage.setItem('authToken', token);
  }, token);

  // Access protected page
  await page.goto('/dashboard');
  await expect(page.getByRole('heading')).toHaveText('Dashboard');
});
```

## Network Interception

### Mocking API Responses

```typescript
test('mock API response', async ({ page }) => {
  // Mock the API response
  await page.route('/api/users', async (route) => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'Mock User 1' },
        { id: 2, name: 'Mock User 2' }
      ])
    });
  });

  await page.goto('/users');
  await expect(page.getByText('Mock User 1')).toBeVisible();
  await expect(page.getByText('Mock User 2')).toBeVisible();
});
```

### Intercepting and Modifying Requests

```typescript
test('modify request headers', async ({ page }) => {
  await page.route('/api/**', async (route) => {
    const headers = {
      ...route.request().headers(),
      'X-Custom-Header': 'test-value'
    };
    await route.continue({ headers });
  });

  await page.goto('/');
});

test('delay response', async ({ page }) => {
  await page.route('/api/users', async (route) => {
    await new Promise(resolve => setTimeout(resolve, 2000));
    await route.continue();
  });

  await page.goto('/users');
  await expect(page.getByTestId('loading')).toBeVisible();
});
```

### Simulating Errors

```typescript
test('handle API error', async ({ page }) => {
  await page.route('/api/users', async (route) => {
    await route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Internal Server Error' })
    });
  });

  await page.goto('/users');
  await expect(page.getByTestId('error-message')).toBeVisible();
});

test('handle network failure', async ({ page }) => {
  await page.route('/api/users', async (route) => {
    await route.abort('failed');
  });

  await page.goto('/users');
  await expect(page.getByText('Network error')).toBeVisible();
});
```

## Test Organization

### Test Hooks

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  // Runs once before all tests in this describe block
  test.beforeAll(async ({ request }) => {
    // Seed database
    await request.post('/api/test/seed');
  });

  // Runs once after all tests in this describe block
  test.afterAll(async ({ request }) => {
    // Clean up database
    await request.post('/api/test/cleanup');
  });

  // Runs before each test
  test.beforeEach(async ({ page }) => {
    await page.goto('/users');
  });

  // Runs after each test
  test.afterEach(async ({ page }) => {
    // Take screenshot on failure is automatic
  });

  test('displays user list', async ({ page }) => {
    await expect(page.getByRole('table')).toBeVisible();
  });
});
```

### Test Annotations

```typescript
// Skip test
test.skip('feature not implemented', async ({ page }) => {
  // This test will be skipped
});

// Skip conditionally
test('skip on webkit', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Feature not supported on WebKit');
});

// Mark as failing (expected to fail)
test.fail('known bug', async ({ page }) => {
  // This test is expected to fail
});

// Focus on specific test (only runs this test)
test.only('debug this test', async ({ page }) => {
  // Only this test runs
});

// Slow test (triples timeout)
test('slow operation', async ({ page }) => {
  test.slow();
  // Test with extended timeout
});

// Add annotations
test('annotated test', async ({ page }) => {
  test.info().annotations.push({
    type: 'issue',
    description: 'https://github.com/org/repo/issues/123'
  });
});
```

### Parameterized Tests

```typescript
const users = [
  { email: 'admin@example.com', role: 'admin' },
  { email: 'user@example.com', role: 'user' },
  { email: 'guest@example.com', role: 'guest' },
];

for (const user of users) {
  test(`login as ${user.role}`, async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill(user.email);
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Login' }).click();
    await expect(page.getByTestId('role')).toHaveText(user.role);
  });
}
```

## Exercise: E-Commerce Test Suite

Create a comprehensive test suite for an e-commerce application.

### Step 1: Create Page Objects

```typescript
// tests/pages/ProductListPage.ts
export class ProductListPage {
  // Implement page object
}

// tests/pages/CartPage.ts
export class CartPage {
  // Implement page object
}

// tests/pages/CheckoutPage.ts
export class CheckoutPage {
  // Implement page object
}
```

### Step 2: Create Fixtures

```typescript
// tests/fixtures.ts
export const test = base.extend({
  // Add fixtures for authenticated user, cart with items, etc.
});
```

### Step 3: Write Test Suites

```typescript
// tests/e-commerce/products.spec.ts
test.describe('Product Listing', () => {
  test('displays all products');
  test('filters by category');
  test('sorts by price');
  test('searches for products');
});

// tests/e-commerce/cart.spec.ts
test.describe('Shopping Cart', () => {
  test('adds product to cart');
  test('updates quantity');
  test('removes product');
  test('calculates total correctly');
});

// tests/e-commerce/checkout.spec.ts
test.describe('Checkout', () => {
  test('completes purchase');
  test('validates required fields');
  test('applies discount code');
});
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Create and use custom fixtures
- [ ] Implement the Page Object Model
- [ ] Write API tests with Playwright
- [ ] Mock and intercept network requests
- [ ] Organize tests with hooks and annotations
- [ ] Create parameterized tests

## Next Steps

Continue to [Chapter III - Advanced Features](chapter_III.md) to learn about visual testing, parallel execution, and CI/CD integration.
