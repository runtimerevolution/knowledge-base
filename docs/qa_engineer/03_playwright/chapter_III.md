# Chapter III - Advanced Features
`(avr. time for this chapter: 2 days)`

This chapter covers advanced Playwright features including visual testing, parallel execution, multi-tab/multi-browser testing, and CI/CD integration.

## Visual Testing

Playwright provides built-in visual comparison capabilities.

### Screenshot Comparison

```typescript
import { test, expect } from '@playwright/test';

test('visual comparison', async ({ page }) => {
  await page.goto('/');
  
  // Full page screenshot
  await expect(page).toHaveScreenshot('homepage.png');
  
  // Element screenshot
  await expect(page.getByTestId('header')).toHaveScreenshot('header.png');
  
  // With options
  await expect(page).toHaveScreenshot('homepage-full.png', {
    fullPage: true,
    maxDiffPixels: 100,
    threshold: 0.2,
  });
});
```

### Updating Snapshots

```bash
# Update all snapshots
npx playwright test --update-snapshots

# Update specific test snapshots
npx playwright test visual.spec.ts --update-snapshots
```

### Visual Testing Best Practices

```typescript
test.describe('Visual Regression', () => {
  test.beforeEach(async ({ page }) => {
    // Disable animations for consistent screenshots
    await page.addStyleTag({
      content: `
        *, *::before, *::after {
          animation-duration: 0s !important;
          transition-duration: 0s !important;
        }
      `
    });
  });

  test('homepage visual test', async ({ page }) => {
    await page.goto('/');
    
    // Wait for images to load
    await page.waitForLoadState('networkidle');
    
    // Hide dynamic content
    await page.locator('[data-testid="timestamp"]').evaluate(
      el => el.style.visibility = 'hidden'
    );
    
    await expect(page).toHaveScreenshot('homepage.png');
  });

  test('responsive visual test', async ({ page }) => {
    // Desktop
    await page.setViewportSize({ width: 1920, height: 1080 });
    await page.goto('/');
    await expect(page).toHaveScreenshot('homepage-desktop.png');
    
    // Tablet
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page).toHaveScreenshot('homepage-tablet.png');
    
    // Mobile
    await page.setViewportSize({ width: 375, height: 667 });
    await expect(page).toHaveScreenshot('homepage-mobile.png');
  });
});
```

## Multi-Tab and Multi-Window Testing

### Working with Multiple Tabs

```typescript
test('multi-tab test', async ({ context }) => {
  // Create two pages in same context
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  
  await page1.goto('/');
  await page2.goto('/about');
  
  // Interact with both pages
  await page1.getByRole('button', { name: 'Open Chat' }).click();
  await page2.getByRole('link', { name: 'Contact' }).click();
  
  // Verify state across tabs
  await expect(page1.getByTestId('chat-window')).toBeVisible();
  await expect(page2).toHaveURL('/contact');
});
```

### Handling Popups

```typescript
test('handle popup window', async ({ page }) => {
  // Listen for popup before triggering it
  const popupPromise = page.waitForEvent('popup');
  
  await page.getByRole('button', { name: 'Open External Link' }).click();
  
  const popup = await popupPromise;
  await popup.waitForLoadState();
  
  // Interact with popup
  await expect(popup).toHaveURL(/external-site/);
  await popup.getByRole('button', { name: 'Accept' }).click();
  
  // Close popup
  await popup.close();
});
```

### Multiple Browser Contexts

```typescript
test('multiple users interaction', async ({ browser }) => {
  // Create two isolated contexts (like incognito windows)
  const userAContext = await browser.newContext();
  const userBContext = await browser.newContext();
  
  const userAPage = await userAContext.newPage();
  const userBPage = await userBContext.newPage();
  
  // Login as different users
  await userAPage.goto('/login');
  await userAPage.getByLabel('Email').fill('usera@example.com');
  await userAPage.getByLabel('Password').fill('password');
  await userAPage.getByRole('button', { name: 'Login' }).click();
  
  await userBPage.goto('/login');
  await userBPage.getByLabel('Email').fill('userb@example.com');
  await userBPage.getByLabel('Password').fill('password');
  await userBPage.getByRole('button', { name: 'Login' }).click();
  
  // Test real-time interaction
  await userAPage.goto('/chat');
  await userBPage.goto('/chat');
  
  await userAPage.getByLabel('Message').fill('Hello from User A');
  await userAPage.getByRole('button', { name: 'Send' }).click();
  
  // Verify User B receives the message
  await expect(userBPage.getByText('Hello from User A')).toBeVisible();
  
  // Cleanup
  await userAContext.close();
  await userBContext.close();
});
```

## File Handling

### File Uploads

```typescript
test('single file upload', async ({ page }) => {
  await page.goto('/upload');
  
  // Upload single file
  await page.getByLabel('Upload file').setInputFiles('tests/fixtures/document.pdf');
  
  await expect(page.getByText('document.pdf')).toBeVisible();
});

test('multiple file upload', async ({ page }) => {
  await page.goto('/upload');
  
  // Upload multiple files
  await page.getByLabel('Upload files').setInputFiles([
    'tests/fixtures/image1.png',
    'tests/fixtures/image2.png'
  ]);
});

test('drag and drop upload', async ({ page }) => {
  await page.goto('/upload');
  
  // Create a file buffer
  const buffer = Buffer.from('test file content');
  
  // Dispatch drop event
  await page.getByTestId('dropzone').dispatchEvent('drop', {
    dataTransfer: {
      files: [{ name: 'test.txt', buffer }]
    }
  });
});
```

### File Downloads

```typescript
test('file download', async ({ page }) => {
  await page.goto('/downloads');
  
  // Wait for download
  const downloadPromise = page.waitForEvent('download');
  await page.getByRole('button', { name: 'Download Report' }).click();
  const download = await downloadPromise;
  
  // Verify download
  expect(download.suggestedFilename()).toBe('report.pdf');
  
  // Save to specific path
  await download.saveAs('tests/downloads/report.pdf');
  
  // Or read content
  const content = await download.createReadStream();
});
```

## Authentication State

### Storing Authentication

```typescript
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Login' }).click();
  
  await page.waitForURL('/dashboard');
  
  // Save authentication state
  await page.context().storageState({ path: authFile });
});
```

### Using Stored Authentication

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    // Setup project
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    
    // Tests that need authentication
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
});
```

### Multiple Authentication States

```typescript
// For different user roles
const adminAuthFile = 'playwright/.auth/admin.json';
const userAuthFile = 'playwright/.auth/user.json';

// playwright.config.ts
export default defineConfig({
  projects: [
    { name: 'setup-admin', testMatch: /admin\.setup\.ts/ },
    { name: 'setup-user', testMatch: /user\.setup\.ts/ },
    {
      name: 'admin-tests',
      testMatch: /.*admin.*\.spec\.ts/,
      use: { storageState: adminAuthFile },
      dependencies: ['setup-admin'],
    },
    {
      name: 'user-tests',
      testMatch: /.*user.*\.spec\.ts/,
      use: { storageState: userAuthFile },
      dependencies: ['setup-user'],
    },
  ],
});
```

## Parallel Execution

### Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  // Run tests in parallel
  fullyParallel: true,
  
  // Number of parallel workers
  workers: process.env.CI ? 4 : undefined,
  
  // Retry failed tests
  retries: process.env.CI ? 2 : 0,
});
```

### Controlling Parallelism

```typescript
// Run tests in this file serially
test.describe.configure({ mode: 'serial' });

test.describe('Sequential Tests', () => {
  test('first', async ({ page }) => {});
  test('second', async ({ page }) => {});
  test('third', async ({ page }) => {});
});

// Run tests in parallel (default)
test.describe.configure({ mode: 'parallel' });
```

### Sharding for CI

```bash
# Split tests across multiple machines
# Machine 1
npx playwright test --shard=1/4

# Machine 2
npx playwright test --shard=2/4

# Machine 3
npx playwright test --shard=3/4

# Machine 4
npx playwright test --shard=4/4
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      
      - name: Run Playwright tests
        run: npx playwright test
      
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Parallel CI with Sharding

```yaml
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run tests
        run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
      
      - name: Upload blob report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: blob-report
          retention-days: 1

  merge-reports:
    if: always()
    needs: [test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - name: Install dependencies
        run: npm ci
      
      - name: Download blob reports
        uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true
      
      - name: Merge reports
        run: npx playwright merge-reports --reporter html ./all-blob-reports
      
      - name: Upload HTML report
        uses: actions/upload-artifact@v4
        with:
          name: html-report
          path: playwright-report
          retention-days: 14
```

## Tracing and Debugging

### Trace Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    // Collect trace on first retry
    trace: 'on-first-retry',
    
    // Or always collect trace
    // trace: 'on',
    
    // Or retain trace only on failure
    // trace: 'retain-on-failure',
  },
});
```

### Viewing Traces

```bash
# View trace file
npx playwright show-trace trace.zip

# Or open trace viewer
npx playwright show-trace
```

### Manual Tracing

```typescript
test('with manual tracing', async ({ page, context }) => {
  // Start tracing
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  await page.goto('/');
  await page.getByRole('button').click();
  
  // Stop and save trace
  await context.tracing.stop({ path: 'trace.zip' });
});
```

## Exercise: Complete Test Framework

Build a production-ready test framework with all advanced features.

### Requirements

1. **Project Structure**
   ```
   tests/
   ├── fixtures/
   │   └── index.ts
   ├── pages/
   │   ├── BasePage.ts
   │   ├── LoginPage.ts
   │   └── DashboardPage.ts
   ├── api/
   │   └── api.spec.ts
   ├── e2e/
   │   ├── auth.spec.ts
   │   └── dashboard.spec.ts
   ├── visual/
   │   └── visual.spec.ts
   └── auth.setup.ts
   ```

2. **Features to Implement**
   - Custom fixtures for authentication
   - Page objects for all pages
   - Visual regression tests
   - API tests
   - Multi-user interaction tests
   - CI/CD pipeline with sharding

3. **Configuration**
   - Multiple browser projects
   - Authentication state reuse
   - Parallel execution
   - Trace collection

### Deliverables

- Complete test framework
- GitHub Actions workflow
- Documentation for running tests
- Test report generation

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Implement visual regression testing
- [ ] Handle multiple tabs and windows
- [ ] Work with file uploads and downloads
- [ ] Manage authentication state efficiently
- [ ] Configure parallel test execution
- [ ] Set up CI/CD pipelines with Playwright
- [ ] Use tracing for debugging

## Next Steps

You've completed the Playwright section! Continue to [Chapter I - Introduction to BDD and Cucumber](../04_cucumber/chapter_I.md) to learn behavior-driven development.
