# Chapter III - Step Definitions
`(avr. time for this chapter: 2 days)`

This chapter covers implementing step definitions with browser automation using Playwright, organizing step definitions, and integrating Cucumber with CI/CD.

## Integrating Cucumber with Playwright

### Setup

```bash
npm install --save-dev @cucumber/cucumber @playwright/test playwright
```

### World with Playwright

Create `support/world.ts`:

```typescript
import { setWorldConstructor, World, IWorldOptions } from '@cucumber/cucumber';
import { Browser, BrowserContext, Page, chromium } from '@playwright/test';

export interface CustomWorld extends World {
  browser: Browser;
  context: BrowserContext;
  page: Page;
}

class PlaywrightWorld extends World implements CustomWorld {
  browser!: Browser;
  context!: BrowserContext;
  page!: Page;

  constructor(options: IWorldOptions) {
    super(options);
  }

  async init() {
    this.browser = await chromium.launch({ headless: true });
    this.context = await this.browser.newContext();
    this.page = await this.context.newPage();
  }

  async cleanup() {
    await this.page?.close();
    await this.context?.close();
    await this.browser?.close();
  }
}

setWorldConstructor(PlaywrightWorld);
```

### Hooks for Browser Management

Create `support/hooks.ts`:

```typescript
import { Before, After, BeforeAll, AfterAll, Status } from '@cucumber/cucumber';
import { CustomWorld } from './world';

Before(async function (this: CustomWorld) {
  await this.init();
});

After(async function (this: CustomWorld, scenario) {
  if (scenario.result?.status === Status.FAILED) {
    // Take screenshot on failure
    const screenshot = await this.page.screenshot();
    this.attach(screenshot, 'image/png');
  }
  await this.cleanup();
});
```

## Implementing Step Definitions

### Navigation Steps

Create `features/step_definitions/navigation.steps.ts`:

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

Given('I am on the {string} page', async function (this: CustomWorld, pageName: string) {
  const urls: Record<string, string> = {
    'home': '/',
    'login': '/login',
    'register': '/register',
    'products': '/products',
    'cart': '/cart',
  };
  
  const url = urls[pageName.toLowerCase()];
  if (!url) throw new Error(`Unknown page: ${pageName}`);
  
  await this.page.goto(`http://localhost:3000${url}`);
});

When('I navigate to {string}', async function (this: CustomWorld, path: string) {
  await this.page.goto(`http://localhost:3000${path}`);
});

Then('I should be on the {string} page', async function (this: CustomWorld, pageName: string) {
  const expectedUrls: Record<string, RegExp> = {
    'home': /\/$/,
    'login': /\/login/,
    'dashboard': /\/dashboard/,
    'products': /\/products/,
  };
  
  const pattern = expectedUrls[pageName.toLowerCase()];
  await expect(this.page).toHaveURL(pattern);
});

Then('the page title should be {string}', async function (this: CustomWorld, title: string) {
  await expect(this.page).toHaveTitle(title);
});
```

### Form Interaction Steps

Create `features/step_definitions/forms.steps.ts`:

```typescript
import { When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

When('I fill in {string} with {string}', async function (
  this: CustomWorld,
  fieldName: string,
  value: string
) {
  await this.page.getByLabel(fieldName).fill(value);
});

When('I fill in the {string} field with {string}', async function (
  this: CustomWorld,
  fieldName: string,
  value: string
) {
  await this.page.getByPlaceholder(fieldName).fill(value);
});

When('I enter {string} in the {string} input', async function (
  this: CustomWorld,
  value: string,
  testId: string
) {
  await this.page.getByTestId(testId).fill(value);
});

When('I select {string} from {string}', async function (
  this: CustomWorld,
  option: string,
  selectName: string
) {
  await this.page.getByLabel(selectName).selectOption(option);
});

When('I check the {string} checkbox', async function (
  this: CustomWorld,
  checkboxName: string
) {
  await this.page.getByLabel(checkboxName).check();
});

When('I uncheck the {string} checkbox', async function (
  this: CustomWorld,
  checkboxName: string
) {
  await this.page.getByLabel(checkboxName).uncheck();
});

Then('the {string} field should have value {string}', async function (
  this: CustomWorld,
  fieldName: string,
  expectedValue: string
) {
  await expect(this.page.getByLabel(fieldName)).toHaveValue(expectedValue);
});

Then('the {string} field should be empty', async function (
  this: CustomWorld,
  fieldName: string
) {
  await expect(this.page.getByLabel(fieldName)).toHaveValue('');
});
```

### Button and Link Steps

Create `features/step_definitions/actions.steps.ts`:

```typescript
import { When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

When('I click the {string} button', async function (
  this: CustomWorld,
  buttonText: string
) {
  await this.page.getByRole('button', { name: buttonText }).click();
});

When('I click the {string} link', async function (
  this: CustomWorld,
  linkText: string
) {
  await this.page.getByRole('link', { name: linkText }).click();
});

When('I click on {string}', async function (
  this: CustomWorld,
  text: string
) {
  await this.page.getByText(text).click();
});

When('I double click on {string}', async function (
  this: CustomWorld,
  text: string
) {
  await this.page.getByText(text).dblclick();
});

When('I press {string}', async function (
  this: CustomWorld,
  key: string
) {
  await this.page.keyboard.press(key);
});

Then('the {string} button should be disabled', async function (
  this: CustomWorld,
  buttonText: string
) {
  await expect(this.page.getByRole('button', { name: buttonText })).toBeDisabled();
});

Then('the {string} button should be enabled', async function (
  this: CustomWorld,
  buttonText: string
) {
  await expect(this.page.getByRole('button', { name: buttonText })).toBeEnabled();
});
```

### Visibility and Content Steps

Create `features/step_definitions/assertions.steps.ts`:

```typescript
import { Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

Then('I should see {string}', async function (
  this: CustomWorld,
  text: string
) {
  await expect(this.page.getByText(text)).toBeVisible();
});

Then('I should not see {string}', async function (
  this: CustomWorld,
  text: string
) {
  await expect(this.page.getByText(text)).toBeHidden();
});

Then('I should see the {string} element', async function (
  this: CustomWorld,
  testId: string
) {
  await expect(this.page.getByTestId(testId)).toBeVisible();
});

Then('I should see {int} {string} elements', async function (
  this: CustomWorld,
  count: number,
  testId: string
) {
  await expect(this.page.getByTestId(testId)).toHaveCount(count);
});

Then('the {string} should contain {string}', async function (
  this: CustomWorld,
  testId: string,
  text: string
) {
  await expect(this.page.getByTestId(testId)).toContainText(text);
});

Then('I should see an error message {string}', async function (
  this: CustomWorld,
  message: string
) {
  await expect(this.page.getByRole('alert')).toContainText(message);
});

Then('I should see a success message {string}', async function (
  this: CustomWorld,
  message: string
) {
  await expect(this.page.getByRole('status')).toContainText(message);
});
```

### Authentication Steps

Create `features/step_definitions/auth.steps.ts`:

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

Given('I am logged in as {string}', async function (
  this: CustomWorld,
  email: string
) {
  await this.page.goto('http://localhost:3000/login');
  await this.page.getByLabel('Email').fill(email);
  await this.page.getByLabel('Password').fill('password123');
  await this.page.getByRole('button', { name: 'Login' }).click();
  await expect(this.page).toHaveURL(/dashboard|home/);
});

Given('I am logged in as an admin', async function (this: CustomWorld) {
  await this.page.goto('http://localhost:3000/login');
  await this.page.getByLabel('Email').fill('admin@example.com');
  await this.page.getByLabel('Password').fill('admin123');
  await this.page.getByRole('button', { name: 'Login' }).click();
  await expect(this.page).toHaveURL(/admin|dashboard/);
});

Given('I am not logged in', async function (this: CustomWorld) {
  await this.page.context().clearCookies();
});

When('I login with email {string} and password {string}', async function (
  this: CustomWorld,
  email: string,
  password: string
) {
  await this.page.getByLabel('Email').fill(email);
  await this.page.getByLabel('Password').fill(password);
  await this.page.getByRole('button', { name: 'Login' }).click();
});

When('I logout', async function (this: CustomWorld) {
  await this.page.getByRole('button', { name: 'Logout' }).click();
});

Then('I should be logged in', async function (this: CustomWorld) {
  await expect(this.page.getByTestId('user-menu')).toBeVisible();
});

Then('I should be logged out', async function (this: CustomWorld) {
  await expect(this.page.getByRole('link', { name: 'Login' })).toBeVisible();
});
```

## Data Table Steps

Create `features/step_definitions/data-table.steps.ts`:

```typescript
import { When, Then, DataTable } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

When('I fill in the form with:', async function (
  this: CustomWorld,
  dataTable: DataTable
) {
  const data = dataTable.rowsHash();
  
  for (const [field, value] of Object.entries(data)) {
    await this.page.getByLabel(field).fill(value as string);
  }
});

When('I add the following items to cart:', async function (
  this: CustomWorld,
  dataTable: DataTable
) {
  const items = dataTable.hashes();
  
  for (const item of items) {
    await this.page.goto(`http://localhost:3000/products`);
    await this.page.getByText(item.product).click();
    
    if (item.quantity && parseInt(item.quantity) > 1) {
      await this.page.getByLabel('Quantity').fill(item.quantity);
    }
    
    await this.page.getByRole('button', { name: 'Add to Cart' }).click();
  }
});

Then('I should see the following products:', async function (
  this: CustomWorld,
  dataTable: DataTable
) {
  const expectedProducts = dataTable.hashes();
  
  for (const product of expectedProducts) {
    const row = this.page.getByRole('row', { name: new RegExp(product.name) });
    await expect(row).toBeVisible();
    
    if (product.price) {
      await expect(row).toContainText(product.price);
    }
  }
});

Then('the cart should contain:', async function (
  this: CustomWorld,
  dataTable: DataTable
) {
  const expectedItems = dataTable.hashes();
  
  for (const item of expectedItems) {
    const cartItem = this.page.getByTestId(`cart-item-${item.product.toLowerCase().replace(/\s+/g, '-')}`);
    await expect(cartItem).toBeVisible();
    await expect(cartItem.getByTestId('quantity')).toHaveText(item.quantity);
  }
});
```

## Organizing Step Definitions

### By Domain

```
features/
├── step_definitions/
│   ├── common/
│   │   ├── navigation.steps.ts
│   │   ├── forms.steps.ts
│   │   └── assertions.steps.ts
│   ├── auth/
│   │   └── auth.steps.ts
│   ├── products/
│   │   └── products.steps.ts
│   ├── cart/
│   │   └── cart.steps.ts
│   └── checkout/
│       └── checkout.steps.ts
```

### Shared Step Library

Create reusable step definitions that can be imported:

```typescript
// features/step_definitions/common/index.ts
export * from './navigation.steps';
export * from './forms.steps';
export * from './assertions.steps';
export * from './actions.steps';
```

## Reporting

### HTML Report

```javascript
// cucumber.js
module.exports = {
  default: {
    format: [
      'progress-bar',
      'html:reports/cucumber-report.html',
      'json:reports/cucumber-report.json'
    ],
  }
};
```

### Custom Reporter

```typescript
// support/reporter.ts
import { Formatter, IFormatterOptions } from '@cucumber/cucumber';

export default class CustomReporter extends Formatter {
  constructor(options: IFormatterOptions) {
    super(options);
    
    options.eventBroadcaster.on('envelope', (envelope) => {
      if (envelope.testCaseFinished) {
        // Handle test case finished
      }
    });
  }
}
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/cucumber.yml
name: Cucumber Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      
      - name: Start application
        run: npm start &
        env:
          PORT: 3000
      
      - name: Wait for app
        run: npx wait-on http://localhost:3000
      
      - name: Run Cucumber tests
        run: npm run test:cucumber
      
      - name: Upload report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: cucumber-report
          path: reports/
```

## Exercise: Complete Step Definition Library

Create a comprehensive step definition library for an e-commerce application.

### Requirements

1. **Navigation Steps**
   - Visit pages by name
   - Navigate to URLs
   - Verify current page

2. **Form Steps**
   - Fill text inputs
   - Select dropdowns
   - Check/uncheck checkboxes
   - Submit forms

3. **Authentication Steps**
   - Login as different user types
   - Logout
   - Verify authentication state

4. **Product Steps**
   - Search for products
   - Filter products
   - View product details

5. **Cart Steps**
   - Add to cart
   - Update quantity
   - Remove from cart
   - Verify cart contents

6. **Checkout Steps**
   - Fill shipping info
   - Fill payment info
   - Complete order

### Deliverables

- Complete step definition files
- Feature files using the steps
- Hooks for setup/teardown
- CI/CD configuration

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Integrate Cucumber with Playwright
- [ ] Implement step definitions for common actions
- [ ] Handle data tables in step definitions
- [ ] Organize step definitions effectively
- [ ] Configure reporting
- [ ] Set up CI/CD for Cucumber tests

## Next Steps

You've completed the Cucumber section! Continue to [Chapter I - Introduction to Capybara](../05_capybara/chapter_I.md) to learn Ruby-based acceptance testing.
