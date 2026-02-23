# Chapter I - Introduction to BDD and Cucumber
`(avr. time for this chapter: 1 day)`

Cucumber is a tool that supports Behavior-Driven Development (BDD). It allows you to write tests in plain language that non-technical stakeholders can understand, bridging the gap between business requirements and technical implementation.

## What is BDD?

Behavior-Driven Development is a software development approach that:

- Focuses on the **behavior** of the system from the user's perspective
- Uses a **shared language** between developers, testers, and business stakeholders
- Creates **living documentation** that stays in sync with the code
- Encourages **collaboration** before implementation

### BDD vs TDD

| Aspect | TDD | BDD |
|--------|-----|-----|
| Focus | Code correctness | System behavior |
| Language | Technical (code) | Natural language |
| Audience | Developers | Everyone |
| Scope | Unit level | Feature level |
| Documentation | Code comments | Feature files |

## Gherkin Syntax

Gherkin is the language used to write Cucumber tests. It uses a specific set of keywords.

### Keywords

```gherkin
Feature: A high-level description of a software feature
  As a [role]
  I want [feature]
  So that [benefit]

  Background: Steps that run before each scenario
    Given some precondition

  Scenario: A specific example of the feature
    Given some context
    When some action is performed
    Then some outcome should occur

  Scenario Outline: A template for multiple examples
    Given I have <count> items
    When I add <more> items
    Then I should have <total> items

    Examples:
      | count | more | total |
      | 1     | 2    | 3     |
      | 5     | 3    | 8     |
```

### Step Keywords

- **Given**: Describes the initial context (preconditions)
- **When**: Describes the action being performed
- **Then**: Describes the expected outcome
- **And/But**: Continues the previous step type

## Setting Up Cucumber

### JavaScript/TypeScript Setup

```bash
# Create project
mkdir cucumber-demo
cd cucumber-demo
npm init -y

# Install Cucumber
npm install --save-dev @cucumber/cucumber

# Install TypeScript support (optional)
npm install --save-dev typescript ts-node @types/node
```

### Project Structure

```
cucumber-demo/
├── features/
│   ├── login.feature
│   ├── cart.feature
│   └── step_definitions/
│       ├── login.steps.ts
│       └── cart.steps.ts
├── support/
│   └── world.ts
├── cucumber.js
├── tsconfig.json
└── package.json
```

### Configuration

Create `cucumber.js`:

```javascript
module.exports = {
  default: {
    requireModule: ['ts-node/register'],
    require: ['features/step_definitions/**/*.ts', 'support/**/*.ts'],
    format: [
      'progress-bar',
      'html:reports/cucumber-report.html',
      'json:reports/cucumber-report.json'
    ],
    formatOptions: { snippetInterface: 'async-await' },
    publishQuiet: true
  }
};
```

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "strict": true,
    "outDir": "./dist",
    "rootDir": "."
  },
  "include": ["features/**/*", "support/**/*"]
}
```

## Your First Feature

### Writing a Feature File

Create `features/login.feature`:

```gherkin
Feature: User Login
  As a registered user
  I want to log in to my account
  So that I can access my personal dashboard

  Background:
    Given I am on the login page

  Scenario: Successful login with valid credentials
    When I enter "test@example.com" as email
    And I enter "password123" as password
    And I click the login button
    Then I should be redirected to the dashboard
    And I should see a welcome message

  Scenario: Failed login with invalid password
    When I enter "test@example.com" as email
    And I enter "wrongpassword" as password
    And I click the login button
    Then I should see an error message "Invalid credentials"
    And I should remain on the login page

  Scenario: Failed login with unregistered email
    When I enter "unknown@example.com" as email
    And I enter "password123" as password
    And I click the login button
    Then I should see an error message "User not found"
```

### Running to Generate Step Definitions

```bash
npx cucumber-js
```

Cucumber will output undefined step snippets that you can copy.

### Implementing Step Definitions

Create `features/step_definitions/login.steps.ts`:

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from 'chai';

// For now, we'll use simple state management
// Later chapters will integrate with Playwright/Cypress

let currentPage = '';
let currentEmail = '';
let currentPassword = '';
let errorMessage = '';
let isLoggedIn = false;

Given('I am on the login page', function () {
  currentPage = 'login';
});

When('I enter {string} as email', function (email: string) {
  currentEmail = email;
});

When('I enter {string} as password', function (password: string) {
  currentPassword = password;
});

When('I click the login button', function () {
  // Simulate login logic
  if (currentEmail === 'test@example.com' && currentPassword === 'password123') {
    isLoggedIn = true;
    currentPage = 'dashboard';
    errorMessage = '';
  } else if (currentEmail === 'unknown@example.com') {
    errorMessage = 'User not found';
  } else {
    errorMessage = 'Invalid credentials';
  }
});

Then('I should be redirected to the dashboard', function () {
  expect(currentPage).to.equal('dashboard');
});

Then('I should see a welcome message', function () {
  expect(isLoggedIn).to.be.true;
});

Then('I should see an error message {string}', function (message: string) {
  expect(errorMessage).to.equal(message);
});

Then('I should remain on the login page', function () {
  expect(currentPage).to.equal('login');
});
```

### Running Tests

```bash
# Run all features
npx cucumber-js

# Run specific feature
npx cucumber-js features/login.feature

# Run specific scenario by name
npx cucumber-js --name "Successful login"

# Run with tags
npx cucumber-js --tags "@smoke"
```

## Tags

Tags help organize and filter scenarios.

### Adding Tags

```gherkin
@login @authentication
Feature: User Login

  @smoke @critical
  Scenario: Successful login with valid credentials
    Given I am on the login page
    ...

  @negative
  Scenario: Failed login with invalid password
    Given I am on the login page
    ...

  @wip
  Scenario: Login with two-factor authentication
    Given I am on the login page
    ...
```

### Running with Tags

```bash
# Run scenarios with specific tag
npx cucumber-js --tags "@smoke"

# Run scenarios with multiple tags (AND)
npx cucumber-js --tags "@smoke and @critical"

# Run scenarios with either tag (OR)
npx cucumber-js --tags "@smoke or @negative"

# Exclude scenarios with tag
npx cucumber-js --tags "not @wip"

# Complex expressions
npx cucumber-js --tags "(@smoke or @critical) and not @wip"
```

## Data Tables

Data tables allow passing structured data to steps.

### Using Data Tables

```gherkin
Feature: User Registration

  Scenario: Register with valid information
    Given I am on the registration page
    When I fill in the registration form with:
      | field     | value              |
      | name      | John Doe           |
      | email     | john@example.com   |
      | password  | SecurePass123!     |
      | confirm   | SecurePass123!     |
    And I click the register button
    Then I should see a success message

  Scenario: Add multiple items to cart
    Given I have an empty cart
    When I add the following items:
      | product    | quantity | price  |
      | Laptop     | 1        | 999.99 |
      | Mouse      | 2        | 29.99  |
      | Keyboard   | 1        | 79.99  |
    Then my cart total should be 1139.96
```

### Implementing Data Table Steps

```typescript
import { When, Then, DataTable } from '@cucumber/cucumber';

When('I fill in the registration form with:', function (dataTable: DataTable) {
  const data = dataTable.rowsHash();
  // data = { name: 'John Doe', email: 'john@example.com', ... }
  
  console.log('Name:', data.name);
  console.log('Email:', data.email);
});

When('I add the following items:', function (dataTable: DataTable) {
  const items = dataTable.hashes();
  // items = [
  //   { product: 'Laptop', quantity: '1', price: '999.99' },
  //   { product: 'Mouse', quantity: '2', price: '29.99' },
  //   ...
  // ]
  
  items.forEach(item => {
    console.log(`Adding ${item.quantity} x ${item.product} at $${item.price}`);
  });
});
```

## Doc Strings

Doc strings allow passing multi-line text to steps.

```gherkin
Scenario: Create a blog post
  Given I am logged in as an author
  When I create a new post with content:
    """
    # Welcome to My Blog
    
    This is my first blog post. I'm excited to share
    my thoughts with you.
    
    ## What to Expect
    
    - Technical tutorials
    - Industry insights
    - Personal experiences
    """
  Then the post should be saved as draft
```

```typescript
When('I create a new post with content:', function (content: string) {
  console.log('Post content:', content);
  // content is the full multi-line string
});
```

## Exercise: Write Feature Files

Create feature files for a shopping application.

### Exercise 1: Shopping Cart Feature

Write a complete feature file for shopping cart functionality:

```gherkin
Feature: Shopping Cart
  As a customer
  I want to manage items in my shopping cart
  So that I can purchase products

  # Write scenarios for:
  # - Adding item to empty cart
  # - Adding multiple items
  # - Updating item quantity
  # - Removing item from cart
  # - Clearing the entire cart
  # - Viewing cart total
```

### Exercise 2: Checkout Feature

Write a complete feature file for checkout:

```gherkin
Feature: Checkout Process
  As a customer with items in my cart
  I want to complete the checkout process
  So that I can receive my products

  # Write scenarios for:
  # - Successful checkout with valid payment
  # - Checkout with invalid credit card
  # - Applying discount code
  # - Selecting shipping method
  # - Guest checkout vs logged-in checkout
```

### Exercise 3: Product Search Feature

Write a complete feature file for product search:

```gherkin
Feature: Product Search
  As a customer
  I want to search for products
  So that I can find what I'm looking for

  # Write scenarios for:
  # - Search by product name
  # - Search with no results
  # - Filter search results by category
  # - Sort search results by price
  # - Pagination of search results
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Explain the difference between BDD and TDD
- [ ] Write feature files using Gherkin syntax
- [ ] Use all Gherkin keywords appropriately
- [ ] Implement basic step definitions
- [ ] Use tags to organize scenarios
- [ ] Work with data tables and doc strings

## Next Steps

Continue to [Chapter II - Writing Feature Files](chapter_II.md) to learn advanced Gherkin patterns and best practices.
