# Chapter II - Writing Tests
`(avr. time for this chapter: 2 days)`

Now that you understand Cypress basics, this chapter covers advanced patterns for writing maintainable, reliable tests. You'll learn about custom commands, fixtures, intercepts, and best practices.

## Custom Commands

Custom commands extend Cypress with reusable functionality.

### Creating Custom Commands

Add to `cypress/support/commands.js`:

```javascript
// Login command
Cypress.Commands.add('login', (email, password) => {
  cy.get('[data-testid="email-input"]').type(email);
  cy.get('[data-testid="password-input"]').type(password);
  cy.get('[data-testid="login-button"]').click();
});

// Login via API (faster)
Cypress.Commands.add('loginByApi', (email, password) => {
  cy.request({
    method: 'POST',
    url: '/api/login',
    body: { email, password }
  }).then((response) => {
    window.localStorage.setItem('token', response.body.token);
  });
});

// Custom assertion
Cypress.Commands.add('shouldBeVisible', { prevSubject: true }, (subject) => {
  cy.wrap(subject).should('be.visible');
});
```

### Using Custom Commands

```javascript
describe('Dashboard', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password123');
  });

  it('shows user dashboard', () => {
    cy.get('[data-testid="dashboard"]').shouldBeVisible();
  });
});
```

## Fixtures

Fixtures are external data files used in tests.

### Creating Fixtures

Create `cypress/fixtures/users.json`:

```json
{
  "validUser": {
    "email": "test@example.com",
    "password": "password123",
    "name": "Test User"
  },
  "adminUser": {
    "email": "admin@example.com",
    "password": "admin123",
    "name": "Admin User"
  },
  "invalidUser": {
    "email": "invalid@example.com",
    "password": "wrongpassword"
  }
}
```

Create `cypress/fixtures/products.json`:

```json
{
  "products": [
    {
      "id": 1,
      "name": "Laptop",
      "price": 999.99,
      "category": "Electronics"
    },
    {
      "id": 2,
      "name": "Headphones",
      "price": 149.99,
      "category": "Electronics"
    },
    {
      "id": 3,
      "name": "Coffee Mug",
      "price": 12.99,
      "category": "Kitchen"
    }
  ]
}
```

### Using Fixtures

```javascript
describe('Login', () => {
  beforeEach(() => {
    cy.fixture('users').as('users');
  });

  it('logs in with valid credentials', function() {
    cy.visit('/login');
    cy.login(this.users.validUser.email, this.users.validUser.password);
    cy.url().should('include', '/dashboard');
  });

  it('shows error with invalid credentials', function() {
    cy.visit('/login');
    cy.login(this.users.invalidUser.email, this.users.invalidUser.password);
    cy.get('[data-testid="error-message"]').should('be.visible');
  });
});
```

## Network Interception

Cypress can intercept and modify network requests.

### Intercepting Requests

```javascript
describe('Products Page', () => {
  beforeEach(() => {
    // Intercept API call and return fixture data
    cy.intercept('GET', '/api/products', { fixture: 'products.json' }).as('getProducts');
    cy.visit('/products');
  });

  it('displays products from API', () => {
    cy.wait('@getProducts');
    cy.get('[data-testid="product-card"]').should('have.length', 3);
  });

  it('shows loading state', () => {
    cy.intercept('GET', '/api/products', {
      fixture: 'products.json',
      delay: 1000
    }).as('getProductsDelayed');
    
    cy.visit('/products');
    cy.get('[data-testid="loading"]').should('be.visible');
    cy.wait('@getProductsDelayed');
    cy.get('[data-testid="loading"]').should('not.exist');
  });
});
```

### Mocking Error Responses

```javascript
it('handles API errors gracefully', () => {
  cy.intercept('GET', '/api/products', {
    statusCode: 500,
    body: { error: 'Internal Server Error' }
  }).as('getProductsError');

  cy.visit('/products');
  cy.wait('@getProductsError');
  cy.get('[data-testid="error-message"]')
    .should('be.visible')
    .and('contain', 'Something went wrong');
});

it('handles network failure', () => {
  cy.intercept('GET', '/api/products', { forceNetworkError: true }).as('networkError');
  
  cy.visit('/products');
  cy.get('[data-testid="error-message"]').should('contain', 'Network error');
});
```

### Modifying Requests

```javascript
it('adds authentication header', () => {
  cy.intercept('GET', '/api/products', (req) => {
    req.headers['Authorization'] = 'Bearer test-token';
  }).as('authenticatedRequest');

  cy.visit('/products');
  cy.wait('@authenticatedRequest').its('request.headers')
    .should('have.property', 'Authorization', 'Bearer test-token');
});
```

## Page Object Model

Organize tests using the Page Object pattern.

### Creating Page Objects

Create `cypress/support/pages/LoginPage.js`:

```javascript
class LoginPage {
  // Selectors
  get emailInput() {
    return cy.get('[data-testid="email-input"]');
  }

  get passwordInput() {
    return cy.get('[data-testid="password-input"]');
  }

  get loginButton() {
    return cy.get('[data-testid="login-button"]');
  }

  get errorMessage() {
    return cy.get('[data-testid="error-message"]');
  }

  get successMessage() {
    return cy.get('[data-testid="success-message"]');
  }

  // Actions
  visit() {
    cy.visit('/login');
  }

  fillEmail(email) {
    this.emailInput.clear().type(email);
    return this;
  }

  fillPassword(password) {
    this.passwordInput.clear().type(password);
    return this;
  }

  submit() {
    this.loginButton.click();
    return this;
  }

  login(email, password) {
    this.fillEmail(email).fillPassword(password).submit();
    return this;
  }

  // Assertions
  assertErrorVisible() {
    this.errorMessage.should('be.visible');
    return this;
  }

  assertSuccessVisible() {
    this.successMessage.should('be.visible');
    return this;
  }
}

export default new LoginPage();
```

### Using Page Objects

```javascript
import LoginPage from '../support/pages/LoginPage';

describe('Login Page', () => {
  beforeEach(() => {
    LoginPage.visit();
  });

  it('logs in successfully', () => {
    LoginPage
      .login('test@example.com', 'password123')
      .assertSuccessVisible();
  });

  it('shows error for invalid credentials', () => {
    LoginPage
      .login('wrong@example.com', 'wrongpassword')
      .assertErrorVisible();
  });

  it('supports method chaining', () => {
    LoginPage
      .fillEmail('test@example.com')
      .fillPassword('password123')
      .submit()
      .assertSuccessVisible();
  });
});
```

## Handling Asynchronous Operations

### Waiting for Elements

```javascript
// Wait for element to appear
cy.get('[data-testid="modal"]', { timeout: 10000 }).should('be.visible');

// Wait for element to disappear
cy.get('[data-testid="loading"]').should('not.exist');

// Wait for specific condition
cy.get('[data-testid="counter"]').should(($el) => {
  expect(parseInt($el.text())).to.be.greaterThan(5);
});
```

### Waiting for API Calls

```javascript
it('waits for data to load', () => {
  cy.intercept('GET', '/api/users').as('getUsers');
  cy.visit('/users');
  
  cy.wait('@getUsers').then((interception) => {
    expect(interception.response.statusCode).to.equal(200);
    expect(interception.response.body).to.have.length.greaterThan(0);
  });
});
```

### Retries and Timeouts

```javascript
// Configure default timeout
// In cypress.config.js
module.exports = defineConfig({
  e2e: {
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 30000,
  }
});

// Override for specific command
cy.get('[data-testid="slow-element"]', { timeout: 30000 });
```

## Working with iframes

```javascript
// Custom command for iframes
Cypress.Commands.add('getIframeBody', (iframeSelector) => {
  return cy
    .get(iframeSelector)
    .its('0.contentDocument.body')
    .should('not.be.empty')
    .then(cy.wrap);
});

// Usage
it('interacts with iframe content', () => {
  cy.getIframeBody('#payment-iframe')
    .find('[data-testid="card-number"]')
    .type('4242424242424242');
});
```

## File Uploads and Downloads

### File Upload

```javascript
it('uploads a file', () => {
  cy.get('[data-testid="file-input"]').selectFile('cypress/fixtures/test-image.png');
  cy.get('[data-testid="upload-success"]').should('be.visible');
});

// Multiple files
it('uploads multiple files', () => {
  cy.get('[data-testid="file-input"]').selectFile([
    'cypress/fixtures/file1.pdf',
    'cypress/fixtures/file2.pdf'
  ]);
});

// Drag and drop
it('uploads via drag and drop', () => {
  cy.get('[data-testid="dropzone"]').selectFile('cypress/fixtures/test.pdf', {
    action: 'drag-drop'
  });
});
```

### File Download

```javascript
it('downloads a file', () => {
  cy.get('[data-testid="download-button"]').click();
  
  // Verify file was downloaded
  cy.readFile('cypress/downloads/report.pdf').should('exist');
});
```

## Exercise: E-Commerce Test Suite

Create a comprehensive test suite for an e-commerce application.

### Step 1: Create Page Objects

Create page objects for:
- `HomePage.js`
- `ProductListPage.js`
- `ProductDetailPage.js`
- `CartPage.js`
- `CheckoutPage.js`

### Step 2: Create Fixtures

Create fixtures for:
- `products.json` - Product data
- `users.json` - User credentials
- `orders.json` - Order data

### Step 3: Write Test Suites

Create test files:

```javascript
// cypress/e2e/e-commerce/home.cy.js
describe('Home Page', () => {
  it('displays featured products');
  it('has working search functionality');
  it('navigates to product categories');
});

// cypress/e2e/e-commerce/products.cy.js
describe('Product Listing', () => {
  it('displays all products');
  it('filters by category');
  it('sorts by price');
  it('paginates results');
});

// cypress/e2e/e-commerce/cart.cy.js
describe('Shopping Cart', () => {
  it('adds product to cart');
  it('updates quantity');
  it('removes product');
  it('shows correct total');
  it('persists cart after page reload');
});

// cypress/e2e/e-commerce/checkout.cy.js
describe('Checkout Flow', () => {
  it('completes purchase as guest');
  it('completes purchase as logged-in user');
  it('validates required fields');
  it('handles payment errors');
});
```

### Step 4: Implement Tests

Fill in each test with actual implementation using:
- Custom commands
- Fixtures
- Network interception
- Page objects

## Best Practices

### Selector Strategy

```javascript
// Best: data-testid attributes
cy.get('[data-testid="submit-button"]');

// Good: Specific CSS selectors
cy.get('form.login button[type="submit"]');

// Avoid: Generic selectors that may change
cy.get('.btn-primary'); // May match multiple elements
cy.get('div > div > button'); // Fragile
```

### Test Independence

```javascript
// Bad: Tests depend on each other
it('creates a user', () => { /* ... */ });
it('logs in with created user', () => { /* depends on previous test */ });

// Good: Each test is independent
beforeEach(() => {
  cy.task('db:seed'); // Reset database
});

it('creates a user', () => { /* ... */ });
it('logs in with existing user', () => { /* uses fixture data */ });
```

### Avoid Conditional Testing

```javascript
// Bad: Conditional logic in tests
cy.get('body').then(($body) => {
  if ($body.find('.modal').length) {
    cy.get('.modal .close').click();
  }
});

// Good: Ensure consistent state
beforeEach(() => {
  cy.clearCookies();
  cy.visit('/');
});
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Create and use custom commands
- [ ] Work with fixtures for test data
- [ ] Intercept and mock network requests
- [ ] Implement the Page Object Model
- [ ] Handle asynchronous operations properly
- [ ] Upload and download files
- [ ] Follow Cypress best practices

## Next Steps

Continue to [Chapter III - Advanced Patterns](chapter_III.md) to learn about visual testing, component testing, and CI/CD integration.
