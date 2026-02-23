# Chapter I - Introduction to Cypress
`(avr. time for this chapter: 1 day)`

Cypress is a modern, JavaScript-based end-to-end testing framework built for the modern web. Unlike Selenium-based tools, Cypress runs directly in the browser, providing faster execution and better debugging capabilities.

## Why Cypress?

### Advantages

- **Fast**: Runs directly in the browser without network lag
- **Real-time Reloads**: Tests reload automatically on file changes
- **Time Travel**: Snapshots at each step for debugging
- **Automatic Waiting**: No need for explicit waits or sleeps
- **Network Control**: Easy stubbing of API responses
- **Screenshots & Videos**: Automatic capture on failures

### When to Use Cypress

- Testing modern web applications (React, Vue, Angular)
- E2E testing of user workflows
- Component testing
- API testing
- Visual regression testing (with plugins)

## Setting Up Cypress

### Prerequisites

- Node.js (v18 or higher)
- npm or yarn
- A web application to test

### Installation

Create a new project and install Cypress:

```bash
# Create project directory
mkdir cypress-demo
cd cypress-demo

# Initialize npm project
npm init -y

# Install Cypress
npm install cypress --save-dev
```

### Opening Cypress

```bash
# Open Cypress Test Runner
npx cypress open
```

On first run, Cypress will create a folder structure:

```
cypress/
├── e2e/              # Your test files
├── fixtures/         # Test data files
├── support/          # Custom commands and setup
│   ├── commands.js   # Custom commands
│   └── e2e.js        # Support file loaded before tests
└── downloads/        # Downloaded files during tests
```

### Configuration

Create or modify `cypress.config.js`:

```javascript
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
  },
});
```

## Your First Test

Create a file `cypress/e2e/first-test.cy.js`:

```javascript
describe('My First Test', () => {
  it('visits the app', () => {
    cy.visit('/');
    cy.contains('Welcome');
  });
});
```

### Running Tests

```bash
# Run in interactive mode
npx cypress open

# Run in headless mode (CI)
npx cypress run

# Run specific test file
npx cypress run --spec "cypress/e2e/first-test.cy.js"
```

## Cypress Architecture

Understanding how Cypress works helps write better tests:

```
┌─────────────────────────────────────────┐
│              Node.js Server             │
│  (Runs in background, handles files)    │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│              Browser                     │
│  ┌─────────────────────────────────┐    │
│  │     Your Application            │    │
│  │     (iframe)                    │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │     Cypress Test Runner         │    │
│  │     (Same origin)               │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

Key points:
- Cypress runs **inside** the browser
- Tests have direct access to the DOM
- No network latency between test and app
- Can intercept and modify network requests

## Basic Commands

### Navigation

```javascript
// Visit a URL
cy.visit('/');
cy.visit('https://example.com');

// Go back/forward
cy.go('back');
cy.go('forward');

// Reload page
cy.reload();
```

### Selecting Elements

```javascript
// By CSS selector
cy.get('.button');
cy.get('#submit-btn');
cy.get('[data-testid="login-form"]');

// By content
cy.contains('Submit');
cy.contains('button', 'Submit');

// By attribute
cy.get('[type="email"]');
cy.get('[name="username"]');
```

### Interacting with Elements

```javascript
// Click
cy.get('button').click();
cy.get('button').dblclick();
cy.get('button').rightclick();

// Type
cy.get('input').type('Hello World');
cy.get('input').type('{enter}');
cy.get('input').clear().type('New text');

// Select
cy.get('select').select('Option 1');
cy.get('select').select(['Option 1', 'Option 2']);

// Checkbox/Radio
cy.get('[type="checkbox"]').check();
cy.get('[type="checkbox"]').uncheck();
cy.get('[type="radio"]').check('value');
```

### Assertions

```javascript
// Should assertions
cy.get('h1').should('be.visible');
cy.get('h1').should('contain', 'Welcome');
cy.get('input').should('have.value', 'test@example.com');
cy.get('.error').should('not.exist');

// Multiple assertions
cy.get('button')
  .should('be.visible')
  .and('be.enabled')
  .and('contain', 'Submit');

// Expect assertions
cy.get('h1').then(($el) => {
  expect($el.text()).to.equal('Welcome');
});
```

## Exercise: Test a Login Page

Create a simple HTML login page to test:

### Step 1: Create Test Application

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Login Page</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 400px;
      margin: 50px auto;
      padding: 20px;
    }
    .form-group {
      margin-bottom: 15px;
    }
    label {
      display: block;
      margin-bottom: 5px;
    }
    input {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
    }
    button {
      width: 100%;
      padding: 10px;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background-color: #0056b3;
    }
    .error {
      color: red;
      margin-top: 10px;
      display: none;
    }
    .success {
      color: green;
      margin-top: 10px;
      display: none;
    }
  </style>
</head>
<body>
  <h1>Login</h1>
  <form id="login-form">
    <div class="form-group">
      <label for="email">Email</label>
      <input type="email" id="email" name="email" data-testid="email-input" required>
    </div>
    <div class="form-group">
      <label for="password">Password</label>
      <input type="password" id="password" name="password" data-testid="password-input" required>
    </div>
    <button type="submit" data-testid="login-button">Login</button>
  </form>
  <p class="error" data-testid="error-message">Invalid email or password</p>
  <p class="success" data-testid="success-message">Login successful! Redirecting...</p>

  <script>
    document.getElementById('login-form').addEventListener('submit', function(e) {
      e.preventDefault();
      const email = document.getElementById('email').value;
      const password = document.getElementById('password').value;
      
      document.querySelector('.error').style.display = 'none';
      document.querySelector('.success').style.display = 'none';
      
      if (email === 'test@example.com' && password === 'password123') {
        document.querySelector('.success').style.display = 'block';
      } else {
        document.querySelector('.error').style.display = 'block';
      }
    });
  </script>
</body>
</html>
```

### Step 2: Serve the Application

```bash
# Install a simple server
npm install -g http-server

# Serve the application
http-server -p 3000
```

### Step 3: Write Cypress Tests

Create `cypress/e2e/login.cy.js`:

```javascript
describe('Login Page', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  describe('Page Elements', () => {
    it('should display the login form', () => {
      // TODO: Verify all form elements are visible
    });

    it('should have empty inputs initially', () => {
      // TODO: Verify inputs are empty
    });
  });

  describe('Form Validation', () => {
    it('should show error for invalid credentials', () => {
      // TODO: Test invalid login
    });

    it('should require email field', () => {
      // TODO: Test email validation
    });

    it('should require password field', () => {
      // TODO: Test password validation
    });
  });

  describe('Successful Login', () => {
    it('should show success message with valid credentials', () => {
      // TODO: Test successful login
    });
  });
});
```

### Step 4: Complete the Tests

Fill in the TODO sections with actual test code. Use the commands learned in this chapter.

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Install and configure Cypress
- [ ] Understand Cypress architecture
- [ ] Write basic tests with navigation and assertions
- [ ] Select elements using various strategies
- [ ] Interact with form elements
- [ ] Run tests in interactive and headless modes

## Next Steps

Continue to [Chapter II - Writing Tests](chapter_II.md) to learn more advanced Cypress patterns and best practices.
