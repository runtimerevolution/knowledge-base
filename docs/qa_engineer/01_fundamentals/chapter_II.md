# Chapter II - Testing Fundamentals
`(avr. time for this chapter: 1 day)`

Before diving into specific testing frameworks, it's crucial to understand the fundamental concepts of software testing. This chapter covers testing theory, types of tests, and best practices that apply across all frameworks.

## What is Software Testing?

Software testing is the process of evaluating a software application to identify differences between expected and actual behavior. It ensures that the software meets requirements and is free of defects.

### Why Testing Matters

- **Quality Assurance**: Ensures the product meets quality standards
- **Bug Detection**: Finds defects before they reach production
- **Cost Reduction**: Fixing bugs early is cheaper than fixing them later
- **Documentation**: Tests serve as living documentation
- **Confidence**: Enables safe refactoring and feature additions

## The Testing Pyramid

The testing pyramid is a concept that helps teams balance different types of tests:

```
        /\
       /  \
      / E2E\        <- Few, slow, expensive
     /------\
    /  Integ \      <- Some, moderate speed
   /----------\
  /    Unit    \    <- Many, fast, cheap
 /--------------\
```

### Unit Tests (Base)
- Test individual functions/methods in isolation
- Fast execution
- Easy to write and maintain
- High code coverage

### Integration Tests (Middle)
- Test how components work together
- Moderate execution speed
- Test API endpoints, database interactions

### End-to-End Tests (Top)
- Test complete user workflows
- Slowest execution
- Most realistic but most fragile
- Test critical user journeys

## Types of Testing

### Functional Testing

Tests that verify the software does what it's supposed to do.

| Type | Description | Example |
|------|-------------|---------|
| Unit Testing | Test individual components | Testing a `calculateTotal()` function |
| Integration Testing | Test component interactions | Testing API with database |
| System Testing | Test complete system | Full application testing |
| Acceptance Testing | Verify business requirements | User story verification |

### Non-Functional Testing

Tests that verify how the software performs.

| Type | Description | Tools |
|------|-------------|-------|
| Performance Testing | Speed and responsiveness | JMeter, k6, Lighthouse |
| Load Testing | Behavior under load | k6, Gatling |
| Security Testing | Vulnerability detection | OWASP ZAP, Burp Suite |
| Usability Testing | User experience | Manual testing, surveys |
| Accessibility Testing | A11y compliance | axe, WAVE |

## Test-Driven Development (TDD)

TDD is a development approach where you write tests before writing code.

### The TDD Cycle (Red-Green-Refactor)

1. **Red**: Write a failing test
2. **Green**: Write minimal code to pass the test
3. **Refactor**: Improve the code while keeping tests green

### Example TDD Workflow

```javascript
// Step 1: RED - Write failing test
describe('Calculator', () => {
  it('should add two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});

// Step 2: GREEN - Write minimal implementation
function add(a, b) {
  return a + b;
}

// Step 3: REFACTOR - Improve if needed
// (In this case, the implementation is already clean)
```

## Behavior-Driven Development (BDD)

BDD extends TDD by writing tests in natural language that describes behavior.

### Gherkin Syntax

```gherkin
Feature: User Login
  As a registered user
  I want to log in to my account
  So that I can access my dashboard

  Scenario: Successful login
    Given I am on the login page
    When I enter valid credentials
    And I click the login button
    Then I should be redirected to the dashboard
    And I should see a welcome message
```

## Writing Good Tests

### The AAA Pattern

Structure your tests with Arrange, Act, Assert:

```javascript
describe('ShoppingCart', () => {
  it('should calculate total with discount', () => {
    // Arrange
    const cart = new ShoppingCart();
    cart.addItem({ name: 'Book', price: 20 });
    cart.addItem({ name: 'Pen', price: 5 });
    cart.applyDiscount(10); // 10% discount

    // Act
    const total = cart.getTotal();

    // Assert
    expect(total).toBe(22.50);
  });
});
```

### Test Naming Conventions

Good test names describe:
- What is being tested
- Under what conditions
- What the expected outcome is

```javascript
// Bad
it('test1', () => { ... });

// Good
it('should return empty array when input is empty', () => { ... });

// Better (using BDD style)
describe('when the cart is empty', () => {
  it('returns a total of zero', () => { ... });
});
```

### FIRST Principles

Good tests are:

- **F**ast: Tests should run quickly
- **I**ndependent: Tests shouldn't depend on each other
- **R**epeatable: Same result every time
- **S**elf-validating: Pass or fail, no manual checking
- **T**imely: Written at the right time (ideally before code)

## Test Data Management

### Fixtures

Pre-defined test data that provides consistent input:

```javascript
// fixtures/users.js
export const validUser = {
  email: 'test@example.com',
  password: 'SecurePass123!',
  name: 'Test User'
};

export const invalidUser = {
  email: 'invalid-email',
  password: '123',
  name: ''
};
```

### Factories

Functions that generate test data dynamically:

```javascript
// factories/userFactory.js
let userCount = 0;

export function createUser(overrides = {}) {
  userCount++;
  return {
    id: userCount,
    email: `user${userCount}@example.com`,
    name: `User ${userCount}`,
    createdAt: new Date(),
    ...overrides
  };
}
```

## Mocking and Stubbing

### When to Mock

- External services (APIs, databases)
- Time-dependent functions
- Random number generators
- File system operations

### Example Mock

```javascript
// Mocking an API call
jest.mock('./api');
import { fetchUser } from './api';

fetchUser.mockResolvedValue({
  id: 1,
  name: 'John Doe'
});

it('should display user name', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('John Doe');
});
```

## Code Coverage

Code coverage measures how much of your code is executed by tests.

### Coverage Metrics

| Metric | Description |
|--------|-------------|
| Line Coverage | Percentage of lines executed |
| Branch Coverage | Percentage of branches (if/else) executed |
| Function Coverage | Percentage of functions called |
| Statement Coverage | Percentage of statements executed |

### Coverage Goals

- Aim for 80%+ coverage as a baseline
- Focus on critical paths, not 100% coverage
- Don't write tests just to increase coverage

## Exercise: Write Test Cases

For each scenario below, write test cases covering:
- Happy path (expected behavior)
- Edge cases
- Error cases

### Scenario 1: Login Form

```
Feature: User Login
- Users can log in with email and password
- Email must be valid format
- Password must be at least 8 characters
- Show error message for invalid credentials
- Redirect to dashboard on success
```

Write test cases for this feature.

### Scenario 2: Shopping Cart

```
Feature: Shopping Cart
- Users can add items to cart
- Users can remove items from cart
- Users can update item quantities
- Cart shows total price
- Discount codes can be applied
- Empty cart shows appropriate message
```

Write test cases for this feature.

### Scenario 3: Search Functionality

```
Feature: Product Search
- Users can search by product name
- Search is case-insensitive
- Results are paginated (10 per page)
- No results shows appropriate message
- Search can be filtered by category
```

Write test cases for this feature.

## Self-Assessment

After completing this chapter, you should understand:

- [ ] The testing pyramid and when to use each type
- [ ] Difference between functional and non-functional testing
- [ ] TDD and BDD methodologies
- [ ] How to write well-structured tests
- [ ] Test data management strategies
- [ ] When and how to use mocks

## Next Steps

Now that you understand testing fundamentals, you're ready to learn specific testing frameworks. Start with [Chapter I - Introduction to Cypress](../02_cypress/chapter_I.md).
