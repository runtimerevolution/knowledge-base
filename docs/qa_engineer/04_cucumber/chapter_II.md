# Chapter II - Writing Feature Files
`(avr. time for this chapter: 1 day)`

This chapter covers advanced Gherkin patterns, best practices for writing maintainable feature files, and common anti-patterns to avoid.

## Feature File Best Practices

### Feature Description

Write clear, business-focused feature descriptions:

```gherkin
# Good - Clear business value
Feature: Password Reset
  As a forgetful user
  I want to reset my password via email
  So that I can regain access to my account

# Bad - Too technical
Feature: Password Reset
  As a user
  I want to call the password reset API
  So that a token is generated in the database
```

### Scenario Naming

Use descriptive, behavior-focused names:

```gherkin
# Good - Describes behavior
Scenario: User receives confirmation email after successful registration
Scenario: Cart total updates when item quantity changes
Scenario: Search returns no results for non-existent product

# Bad - Vague or technical
Scenario: Test registration
Scenario: Cart test 1
Scenario: API returns 404
```

### Step Writing Guidelines

Write steps that are:
- **Declarative** (what, not how)
- **Business-focused** (user perspective)
- **Reusable** (avoid specific values in step text)

```gherkin
# Good - Declarative
Given I am logged in as a premium user
When I add a product to my wishlist
Then I should see the product in my wishlist

# Bad - Imperative (too detailed)
Given I navigate to "https://example.com/login"
And I enter "user@example.com" in the email field
And I enter "password123" in the password field
And I click the button with id "login-btn"
When I click on the heart icon next to product id 123
Then I should see product 123 in the div with class "wishlist-items"
```

## Scenario Outline Patterns

### Basic Scenario Outline

```gherkin
Scenario Outline: Login with different user types
  Given I am on the login page
  When I login as a "<user_type>" user
  Then I should see the "<dashboard>" dashboard

  Examples:
    | user_type | dashboard     |
    | admin     | Admin         |
    | manager   | Management    |
    | employee  | Employee      |
    | guest     | Limited       |
```

### Multiple Example Tables

```gherkin
Scenario Outline: Shipping cost calculation
  Given I have items worth $<subtotal> in my cart
  When I select "<shipping>" shipping to "<region>"
  Then my shipping cost should be $<cost>

  @domestic
  Examples: Domestic Shipping
    | subtotal | shipping | region    | cost  |
    | 50       | standard | East      | 5.99  |
    | 50       | express  | East      | 12.99 |
    | 100      | standard | East      | 0.00  |

  @international
  Examples: International Shipping
    | subtotal | shipping | region    | cost  |
    | 50       | standard | Europe    | 15.99 |
    | 50       | express  | Europe    | 29.99 |
    | 100      | standard | Europe    | 10.99 |
```

### Complex Data in Outlines

```gherkin
Scenario Outline: Form validation messages
  Given I am on the registration form
  When I submit with <field> as "<value>"
  Then I should see the error "<error_message>"

  Examples:
    | field    | value              | error_message                    |
    | email    |                    | Email is required                |
    | email    | invalid            | Please enter a valid email       |
    | email    | test@              | Please enter a valid email       |
    | password |                    | Password is required             |
    | password | 123                | Password must be at least 8 chars|
    | password | password           | Password must contain a number   |
```

## Background Best Practices

### When to Use Background

Use Background for:
- Common preconditions shared by ALL scenarios
- Setup that doesn't change between scenarios

```gherkin
Feature: User Profile Management

  Background:
    Given I am logged in as "john@example.com"
    And I am on my profile page

  Scenario: Update display name
    When I change my display name to "John Smith"
    Then my display name should be "John Smith"

  Scenario: Update email preferences
    When I disable marketing emails
    Then I should not receive marketing emails
```

### When NOT to Use Background

Don't use Background when:
- Only some scenarios need the setup
- The setup is complex and obscures the scenario

```gherkin
# Bad - Background doesn't apply to all scenarios
Feature: Shopping Cart

  Background:
    Given I am logged in
    And I have 3 items in my cart

  Scenario: Add item to cart
    # This scenario doesn't need items in cart!
    When I add a product to my cart
    Then my cart should have 4 items

  Scenario: Empty cart message
    # This scenario needs an EMPTY cart!
    Given my cart is empty  # Contradicts background!
    Then I should see "Your cart is empty"
```

## Hooks and World

### Cucumber Hooks

```typescript
// support/hooks.ts
import { Before, After, BeforeAll, AfterAll, BeforeStep, AfterStep } from '@cucumber/cucumber';

BeforeAll(async function () {
  // Runs once before all scenarios
  console.log('Starting test suite');
});

AfterAll(async function () {
  // Runs once after all scenarios
  console.log('Test suite complete');
});

Before(async function () {
  // Runs before each scenario
  console.log('Starting scenario:', this.pickle.name);
});

After(async function (scenario) {
  // Runs after each scenario
  if (scenario.result?.status === 'FAILED') {
    // Take screenshot on failure
    console.log('Scenario failed:', scenario.pickle.name);
  }
});

// Hooks with tags
Before({ tags: '@database' }, async function () {
  // Only runs for scenarios tagged @database
  await this.seedDatabase();
});

After({ tags: '@database' }, async function () {
  await this.cleanDatabase();
});
```

### World Object

The World provides shared context between steps:

```typescript
// support/world.ts
import { setWorldConstructor, World, IWorldOptions } from '@cucumber/cucumber';

interface CustomWorld extends World {
  currentUser: { email: string; role: string } | null;
  cart: { items: any[]; total: number };
  lastResponse: any;
}

class TestWorld extends World implements CustomWorld {
  currentUser = null;
  cart = { items: [], total: 0 };
  lastResponse = null;

  constructor(options: IWorldOptions) {
    super(options);
  }

  async login(email: string, password: string) {
    // Shared login logic
    this.currentUser = { email, role: 'user' };
  }

  async addToCart(product: any) {
    this.cart.items.push(product);
    this.cart.total += product.price;
  }
}

setWorldConstructor(TestWorld);
```

### Using World in Steps

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { CustomWorld } from '../support/world';

Given('I am logged in as {string}', async function (this: CustomWorld, email: string) {
  await this.login(email, 'password123');
});

When('I add {string} to my cart', async function (this: CustomWorld, productName: string) {
  const product = { name: productName, price: 29.99 };
  await this.addToCart(product);
});

Then('my cart total should be {float}', function (this: CustomWorld, expectedTotal: number) {
  expect(this.cart.total).to.equal(expectedTotal);
});
```

## Parameter Types

### Built-in Parameter Types

```typescript
// {int} - Integer
When('I have {int} items', function (count: number) {});

// {float} - Decimal number
Then('the total is {float}', function (total: number) {});

// {string} - Quoted string
When('I search for {string}', function (query: string) {});

// {word} - Single word without quotes
Given('I am a {word} user', function (role: string) {});
```

### Custom Parameter Types

```typescript
// support/parameter-types.ts
import { defineParameterType } from '@cucumber/cucumber';

// Custom type for user roles
defineParameterType({
  name: 'role',
  regexp: /admin|manager|employee|guest/,
  transformer: (role) => role
});

// Custom type for currency
defineParameterType({
  name: 'currency',
  regexp: /\$[\d,]+\.?\d*/,
  transformer: (amount) => parseFloat(amount.replace(/[$,]/g, ''))
});

// Custom type for dates
defineParameterType({
  name: 'date',
  regexp: /\d{4}-\d{2}-\d{2}/,
  transformer: (dateStr) => new Date(dateStr)
});
```

### Using Custom Types

```gherkin
Scenario: Role-based access
  Given I am logged in as an admin user
  When I access the admin panel
  Then I should see the admin dashboard

Scenario: Price calculation
  Given the product costs $99.99
  When I apply a 10% discount
  Then the final price should be $89.99

Scenario: Scheduled task
  Given I schedule a task for 2024-03-15
  Then the task should appear on 2024-03-15
```

```typescript
Given('I am logged in as a/an {role} user', function (role: string) {
  this.currentUser = { role };
});

Given('the product costs {currency}', function (price: number) {
  this.product = { price };
});

Given('I schedule a task for {date}', function (date: Date) {
  this.scheduledDate = date;
});
```

## Anti-Patterns to Avoid

### 1. Incidental Details

```gherkin
# Bad - Too many irrelevant details
Scenario: User login
  Given I open Chrome browser
  And I navigate to "https://example.com"
  And I wait for the page to load
  And I see the login form
  When I click on the email input field
  And I type "test@example.com"
  And I click on the password input field
  And I type "password123"
  And I click the blue login button
  Then I see the URL change to "/dashboard"

# Good - Focus on behavior
Scenario: User login
  Given I am on the login page
  When I login with valid credentials
  Then I should be on the dashboard
```

### 2. Conjunction Steps

```gherkin
# Bad - Multiple actions in one step
When I login and add a product to cart and proceed to checkout

# Good - Separate steps
When I login with valid credentials
And I add "Laptop" to my cart
And I proceed to checkout
```

### 3. Scenario Dependency

```gherkin
# Bad - Scenarios depend on each other
Scenario: Create user
  When I create user "john@example.com"
  Then the user should be created

Scenario: Login with created user
  # Depends on previous scenario!
  When I login as "john@example.com"
  Then I should be logged in

# Good - Independent scenarios
Scenario: Create user
  When I create user "john@example.com"
  Then the user should be created

Scenario: Login with existing user
  Given a user "john@example.com" exists
  When I login as "john@example.com"
  Then I should be logged in
```

### 4. Feature Envy

```gherkin
# Bad - Testing implementation details
Scenario: User registration
  When I submit the registration form
  Then the users table should have 1 new row
  And the email_queue table should have 1 new row
  And the audit_log should contain "USER_CREATED"

# Good - Testing behavior
Scenario: User registration
  When I submit the registration form
  Then I should receive a confirmation email
  And I should be able to login with my new account
```

## Exercise: Refactor Feature Files

### Exercise 1: Refactor Bad Feature

Refactor this poorly written feature:

```gherkin
Feature: test shopping

Scenario: test 1
  Given I open browser
  And go to http://localhost:3000
  And click login
  And type admin@test.com in email
  And type 123456 in password
  And click submit
  And wait 2 seconds
  When click on product 1
  And click add to cart button
  And click cart icon
  Then see 1 item in cart div

Scenario: test 2
  # Depends on test 1
  When click checkout
  And fill form
  And click pay
  Then see success
```

### Exercise 2: Write Clean Feature

Write a clean, well-structured feature file for:
- User account management (update profile, change password, delete account)
- Include at least 5 scenarios
- Use Background appropriately
- Use Scenario Outline where applicable
- Add meaningful tags

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Write clear, business-focused feature descriptions
- [ ] Use Scenario Outline effectively
- [ ] Apply Background appropriately
- [ ] Create and use custom parameter types
- [ ] Implement hooks and World object
- [ ] Identify and avoid common anti-patterns

## Next Steps

Continue to [Chapter III - Step Definitions](chapter_III.md) to learn how to implement step definitions with browser automation.
