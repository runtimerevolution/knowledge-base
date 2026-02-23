# Chapter I - Introduction to Playwright
`(avr. time for this chapter: 1 day)`

Playwright is a powerful end-to-end testing framework developed by Microsoft. It supports multiple browsers (Chromium, Firefox, WebKit), multiple languages (JavaScript, TypeScript, Python, Java, .NET), and provides excellent cross-browser testing capabilities.

## Why Playwright?

### Advantages

- **Cross-browser**: Test on Chromium, Firefox, and WebKit with one API
- **Cross-platform**: Run on Windows, macOS, and Linux
- **Auto-wait**: Automatically waits for elements to be actionable
- **Web-first assertions**: Built-in assertions that auto-retry
- **Tracing**: Capture screenshots, videos, and trace files
- **Parallel execution**: Run tests in parallel out of the box
- **Multiple languages**: JavaScript, TypeScript, Python, Java, C#

### Playwright vs Cypress

| Feature | Playwright | Cypress |
|---------|------------|---------|
| Browsers | Chromium, Firefox, WebKit | Chromium, Firefox, WebKit |
| Languages | JS, TS, Python, Java, C# | JavaScript only |
| Parallel | Built-in | Requires paid dashboard |
| iframes | Native support | Requires workarounds |
| Multiple tabs | Supported | Not supported |
| Network | Full control | Full control |

## Setting Up Playwright

### Installation

```bash
# Create project directory
mkdir playwright-demo
cd playwright-demo

# Initialize npm project
npm init -y

# Install Playwright
npm init playwright@latest
```

During installation, you'll be asked:
- TypeScript or JavaScript? (Choose TypeScript for better tooling)
- Where to put tests? (Default: `tests`)
- Add GitHub Actions workflow? (Yes for CI/CD)
- Install Playwright browsers? (Yes)

### Project Structure

```
playwright-demo/
├── tests/                    # Test files
│   └── example.spec.ts
├── tests-examples/           # Example tests
├── playwright.config.ts      # Configuration
├── package.json
└── .github/
    └── workflows/
        └── playwright.yml    # CI workflow
```

### Configuration

`playwright.config.ts`:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

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
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Your First Test

Create `tests/first-test.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveTitle(/Welcome/);
});

test('get started link', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('link', { name: 'Get started' }).click();
  await expect(page).toHaveURL(/.*intro/);
});
```

### Running Tests

```bash
# Run all tests
npx playwright test

# Run in headed mode (see browser)
npx playwright test --headed

# Run specific file
npx playwright test tests/first-test.spec.ts

# Run specific browser
npx playwright test --project=chromium

# Run with UI mode
npx playwright test --ui

# Show HTML report
npx playwright show-report
```

## Locators

Playwright provides multiple ways to locate elements.

### Role-based Locators (Recommended)

```typescript
// By role
await page.getByRole('button', { name: 'Submit' });
await page.getByRole('link', { name: 'Home' });
await page.getByRole('textbox', { name: 'Email' });
await page.getByRole('checkbox', { name: 'Remember me' });

// By label
await page.getByLabel('Email');
await page.getByLabel('Password');

// By placeholder
await page.getByPlaceholder('Enter your email');

// By text
await page.getByText('Welcome');
await page.getByText('Welcome', { exact: true });

// By alt text (images)
await page.getByAltText('Company logo');

// By title
await page.getByTitle('Close');

// By test id
await page.getByTestId('submit-button');
```

### CSS and XPath Locators

```typescript
// CSS selector
await page.locator('.submit-button');
await page.locator('#email-input');
await page.locator('[data-testid="login-form"]');

// XPath
await page.locator('xpath=//button[@type="submit"]');

// Combining locators
await page.locator('form').locator('button[type="submit"]');
```

### Filtering Locators

```typescript
// Filter by text
await page.getByRole('listitem').filter({ hasText: 'Product 1' });

// Filter by another locator
await page.getByRole('listitem').filter({
  has: page.getByRole('button', { name: 'Add to cart' })
});

// Filter by not having
await page.getByRole('listitem').filter({
  hasNot: page.getByRole('button', { name: 'Sold out' })
});

// Nth element
await page.getByRole('listitem').nth(0);
await page.getByRole('listitem').first();
await page.getByRole('listitem').last();
```

## Actions

### Click Actions

```typescript
// Simple click
await page.getByRole('button', { name: 'Submit' }).click();

// Double click
await page.getByRole('button').dblclick();

// Right click
await page.getByRole('button').click({ button: 'right' });

// Shift + click
await page.getByRole('button').click({ modifiers: ['Shift'] });

// Click at position
await page.getByRole('button').click({ position: { x: 10, y: 10 } });

// Force click (bypass actionability checks)
await page.getByRole('button').click({ force: true });
```

### Input Actions

```typescript
// Type text
await page.getByLabel('Email').fill('test@example.com');

// Type with delay (simulate real typing)
await page.getByLabel('Email').pressSequentially('test@example.com', { delay: 100 });

// Clear and type
await page.getByLabel('Email').clear();
await page.getByLabel('Email').fill('new@example.com');

// Press keys
await page.getByLabel('Search').press('Enter');
await page.keyboard.press('Control+A');
```

### Select and Checkbox

```typescript
// Select dropdown
await page.getByLabel('Country').selectOption('usa');
await page.getByLabel('Country').selectOption({ label: 'United States' });
await page.getByLabel('Country').selectOption({ value: 'usa' });

// Multiple select
await page.getByLabel('Colors').selectOption(['red', 'blue']);

// Checkbox
await page.getByRole('checkbox', { name: 'Remember me' }).check();
await page.getByRole('checkbox', { name: 'Remember me' }).uncheck();

// Radio button
await page.getByRole('radio', { name: 'Option A' }).check();
```

## Assertions

Playwright provides auto-retrying assertions.

### Page Assertions

```typescript
// Title
await expect(page).toHaveTitle('Home Page');
await expect(page).toHaveTitle(/Home/);

// URL
await expect(page).toHaveURL('https://example.com/home');
await expect(page).toHaveURL(/.*home/);
```

### Element Assertions

```typescript
// Visibility
await expect(page.getByRole('button')).toBeVisible();
await expect(page.getByRole('button')).toBeHidden();

// Enabled/Disabled
await expect(page.getByRole('button')).toBeEnabled();
await expect(page.getByRole('button')).toBeDisabled();

// Text content
await expect(page.getByRole('heading')).toHaveText('Welcome');
await expect(page.getByRole('heading')).toContainText('Welcome');

// Value
await expect(page.getByLabel('Email')).toHaveValue('test@example.com');

// Attribute
await expect(page.getByRole('link')).toHaveAttribute('href', '/home');

// CSS class
await expect(page.getByRole('button')).toHaveClass(/primary/);

// Count
await expect(page.getByRole('listitem')).toHaveCount(5);

// Checked state
await expect(page.getByRole('checkbox')).toBeChecked();
await expect(page.getByRole('checkbox')).not.toBeChecked();
```

### Soft Assertions

```typescript
// Continue test even if assertion fails
await expect.soft(page.getByRole('heading')).toHaveText('Welcome');
await expect.soft(page.getByRole('button')).toBeVisible();

// Check for any soft assertion failures
expect(test.info().errors).toHaveLength(0);
```

## Exercise: Test a Login Page

### Step 1: Create Test Application

Use the same HTML from the Cypress chapter or create a new one.

### Step 2: Write Playwright Tests

Create `tests/login.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Page', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test.describe('Page Elements', () => {
    test('should display the login form', async ({ page }) => {
      // TODO: Verify all form elements are visible
      await expect(page.getByRole('heading', { name: 'Login' })).toBeVisible();
      await expect(page.getByLabel('Email')).toBeVisible();
      await expect(page.getByLabel('Password')).toBeVisible();
      await expect(page.getByRole('button', { name: 'Login' })).toBeVisible();
    });

    test('should have empty inputs initially', async ({ page }) => {
      // TODO: Verify inputs are empty
    });
  });

  test.describe('Form Validation', () => {
    test('should show error for invalid credentials', async ({ page }) => {
      // TODO: Test invalid login
    });
  });

  test.describe('Successful Login', () => {
    test('should show success message with valid credentials', async ({ page }) => {
      // TODO: Test successful login
    });
  });
});
```

### Step 3: Run Tests

```bash
# Run tests
npx playwright test tests/login.spec.ts

# Run with UI mode for debugging
npx playwright test tests/login.spec.ts --ui

# Run in headed mode
npx playwright test tests/login.spec.ts --headed
```

## Debugging

### UI Mode

```bash
npx playwright test --ui
```

### Debug Mode

```bash
# Debug all tests
npx playwright test --debug

# Debug specific test
npx playwright test tests/login.spec.ts --debug
```

### Pause in Test

```typescript
test('debug example', async ({ page }) => {
  await page.goto('/');
  await page.pause(); // Opens inspector
  await page.getByRole('button').click();
});
```

### Trace Viewer

```bash
# Run with trace
npx playwright test --trace on

# View trace
npx playwright show-trace trace.zip
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Install and configure Playwright
- [ ] Understand Playwright architecture
- [ ] Use different locator strategies
- [ ] Perform common actions (click, type, select)
- [ ] Write assertions with auto-retry
- [ ] Debug tests using UI mode and traces

## Next Steps

Continue to [Chapter II - Writing Tests](chapter_II.md) to learn advanced Playwright patterns including fixtures, page objects, and API testing.
