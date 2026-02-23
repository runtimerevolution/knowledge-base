# Chapter III - Advanced Patterns
`(avr. time for this chapter: 2 days)`

This chapter covers advanced Cypress features including component testing, visual regression testing, CI/CD integration, and performance optimization.

## Component Testing

Cypress supports testing individual components in isolation.

### Setup for React

```bash
npm install --save-dev @cypress/react
```

Update `cypress.config.js`:

```javascript
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite', // or 'webpack'
    },
    specPattern: 'src/**/*.cy.{js,jsx,ts,tsx}',
  },
});
```

### Writing Component Tests

Create `src/components/Button.cy.jsx`:

```javascript
import React from 'react';
import { mount } from 'cypress/react';
import Button from './Button';

describe('Button Component', () => {
  it('renders with text', () => {
    mount(<Button>Click me</Button>);
    cy.get('button').should('contain', 'Click me');
  });

  it('handles click events', () => {
    const onClick = cy.stub().as('clickHandler');
    mount(<Button onClick={onClick}>Click me</Button>);
    
    cy.get('button').click();
    cy.get('@clickHandler').should('have.been.calledOnce');
  });

  it('can be disabled', () => {
    mount(<Button disabled>Disabled</Button>);
    cy.get('button').should('be.disabled');
  });

  it('applies variant styles', () => {
    mount(<Button variant="primary">Primary</Button>);
    cy.get('button').should('have.class', 'btn-primary');
  });
});
```

### Testing with Props and State

```javascript
import React, { useState } from 'react';
import { mount } from 'cypress/react';
import Counter from './Counter';

describe('Counter Component', () => {
  it('increments count', () => {
    mount(<Counter initialCount={0} />);
    
    cy.get('[data-testid="count"]').should('contain', '0');
    cy.get('[data-testid="increment"]').click();
    cy.get('[data-testid="count"]').should('contain', '1');
  });

  it('decrements count', () => {
    mount(<Counter initialCount={5} />);
    
    cy.get('[data-testid="decrement"]').click();
    cy.get('[data-testid="count"]').should('contain', '4');
  });

  it('does not go below zero', () => {
    mount(<Counter initialCount={0} min={0} />);
    
    cy.get('[data-testid="decrement"]').click();
    cy.get('[data-testid="count"]').should('contain', '0');
  });
});
```

## Visual Regression Testing

Detect unintended visual changes with screenshot comparison.

### Using cypress-image-snapshot

```bash
npm install --save-dev @simonsmith/cypress-image-snapshot
```

Setup in `cypress/support/commands.js`:

```javascript
import { addMatchImageSnapshotCommand } from '@simonsmith/cypress-image-snapshot/command';

addMatchImageSnapshotCommand({
  failureThreshold: 0.03,
  failureThresholdType: 'percent',
  customDiffConfig: { threshold: 0.1 },
  capture: 'viewport',
});
```

Setup in `cypress/support/e2e.js`:

```javascript
import '@simonsmith/cypress-image-snapshot/command';
```

### Writing Visual Tests

```javascript
describe('Visual Regression', () => {
  it('matches homepage snapshot', () => {
    cy.visit('/');
    cy.matchImageSnapshot('homepage');
  });

  it('matches login form snapshot', () => {
    cy.visit('/login');
    cy.get('[data-testid="login-form"]').matchImageSnapshot('login-form');
  });

  it('matches button states', () => {
    cy.visit('/components/buttons');
    
    // Default state
    cy.get('[data-testid="primary-button"]').matchImageSnapshot('button-default');
    
    // Hover state
    cy.get('[data-testid="primary-button"]').trigger('mouseover');
    cy.get('[data-testid="primary-button"]').matchImageSnapshot('button-hover');
  });

  it('matches responsive layouts', () => {
    // Desktop
    cy.viewport(1280, 720);
    cy.visit('/');
    cy.matchImageSnapshot('homepage-desktop');
    
    // Tablet
    cy.viewport(768, 1024);
    cy.matchImageSnapshot('homepage-tablet');
    
    // Mobile
    cy.viewport(375, 667);
    cy.matchImageSnapshot('homepage-mobile');
  });
});
```

## API Testing

Cypress can test APIs directly without a browser.

### Basic API Tests

```javascript
describe('API Tests', () => {
  const apiUrl = Cypress.env('API_URL') || 'http://localhost:3001/api';

  describe('Users API', () => {
    it('GET /users returns list of users', () => {
      cy.request(`${apiUrl}/users`).then((response) => {
        expect(response.status).to.eq(200);
        expect(response.body).to.be.an('array');
        expect(response.body.length).to.be.greaterThan(0);
      });
    });

    it('POST /users creates a new user', () => {
      const newUser = {
        name: 'John Doe',
        email: 'john@example.com'
      };

      cy.request('POST', `${apiUrl}/users`, newUser).then((response) => {
        expect(response.status).to.eq(201);
        expect(response.body).to.have.property('id');
        expect(response.body.name).to.eq(newUser.name);
      });
    });

    it('PUT /users/:id updates a user', () => {
      cy.request(`${apiUrl}/users/1`).then((response) => {
        const user = response.body;
        user.name = 'Updated Name';

        cy.request('PUT', `${apiUrl}/users/1`, user).then((updateResponse) => {
          expect(updateResponse.status).to.eq(200);
          expect(updateResponse.body.name).to.eq('Updated Name');
        });
      });
    });

    it('DELETE /users/:id removes a user', () => {
      cy.request('DELETE', `${apiUrl}/users/1`).then((response) => {
        expect(response.status).to.eq(200);
      });

      cy.request({
        url: `${apiUrl}/users/1`,
        failOnStatusCode: false
      }).then((response) => {
        expect(response.status).to.eq(404);
      });
    });
  });

  describe('Authentication API', () => {
    it('returns token for valid credentials', () => {
      cy.request('POST', `${apiUrl}/auth/login`, {
        email: 'test@example.com',
        password: 'password123'
      }).then((response) => {
        expect(response.status).to.eq(200);
        expect(response.body).to.have.property('token');
      });
    });

    it('returns 401 for invalid credentials', () => {
      cy.request({
        method: 'POST',
        url: `${apiUrl}/auth/login`,
        body: {
          email: 'wrong@example.com',
          password: 'wrongpassword'
        },
        failOnStatusCode: false
      }).then((response) => {
        expect(response.status).to.eq(401);
      });
    });
  });
});
```

## CI/CD Integration

### GitHub Actions

Create `.github/workflows/cypress.yml`:

```yaml
name: Cypress Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Start application
        run: npm start &
        env:
          PORT: 3000

      - name: Wait for app
        run: npx wait-on http://localhost:3000

      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          browser: chrome
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Upload screenshots
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots

      - name: Upload videos
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: cypress-videos
          path: cypress/videos
```

### Parallel Test Execution

```yaml
jobs:
  cypress-run:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        containers: [1, 2, 3, 4]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cypress run
        uses: cypress-io/github-action@v6
        with:
          record: true
          parallel: true
          group: 'UI Tests'
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
```

### Docker Configuration

Create `Dockerfile.cypress`:

```dockerfile
FROM cypress/included:13.6.0

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "cypress", "run"]
```

Create `docker-compose.cypress.yml`:

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=test

  cypress:
    build:
      context: .
      dockerfile: Dockerfile.cypress
    depends_on:
      - app
    environment:
      - CYPRESS_baseUrl=http://app:3000
    volumes:
      - ./cypress/screenshots:/app/cypress/screenshots
      - ./cypress/videos:/app/cypress/videos
```

## Performance Optimization

### Test Execution Speed

```javascript
// Use cy.session for login state
Cypress.Commands.add('loginSession', (email, password) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email"]').type(email);
    cy.get('[data-testid="password"]').type(password);
    cy.get('[data-testid="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});

// Usage - session is cached
beforeEach(() => {
  cy.loginSession('test@example.com', 'password123');
  cy.visit('/dashboard');
});
```

### Selective Test Running

```javascript
// Tag tests for selective running
describe('Critical Path', { tags: ['@critical'] }, () => {
  it('completes checkout', () => { /* ... */ });
});

describe('Edge Cases', { tags: ['@edge'] }, () => {
  it('handles empty cart', () => { /* ... */ });
});
```

Run with tags:
```bash
npx cypress run --env grepTags=@critical
```

### Reducing Flakiness

```javascript
// Retry failed tests
// In cypress.config.js
module.exports = defineConfig({
  retries: {
    runMode: 2,
    openMode: 0,
  },
});

// Use stable selectors
cy.get('[data-testid="submit"]'); // Good
cy.get('.btn-primary').first(); // Avoid

// Wait for specific conditions
cy.get('[data-testid="list"]')
  .should('be.visible')
  .find('[data-testid="item"]')
  .should('have.length.at.least', 1);
```

## Exercise: Complete Test Suite

Build a comprehensive test suite for a Todo application.

### Requirements

1. **E2E Tests**
   - User can create, edit, and delete todos
   - User can mark todos as complete
   - User can filter todos (all, active, completed)
   - User can clear completed todos

2. **Component Tests**
   - TodoItem component
   - TodoList component
   - TodoFilter component

3. **Visual Tests**
   - Homepage layout
   - Todo item states (default, completed, editing)
   - Responsive design

4. **API Tests**
   - CRUD operations for todos
   - Error handling

5. **CI/CD**
   - GitHub Actions workflow
   - Parallel execution
   - Artifact upload

### Deliverables

- Page objects for all pages
- Fixtures for test data
- Custom commands for common operations
- Complete test coverage
- CI/CD configuration

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Write component tests with Cypress
- [ ] Implement visual regression testing
- [ ] Test APIs directly with Cypress
- [ ] Configure CI/CD pipelines for Cypress
- [ ] Optimize test execution speed
- [ ] Reduce test flakiness

## Next Steps

You've completed the Cypress section! Continue to [Chapter I - Introduction to Playwright](../03_playwright/chapter_I.md) to learn another powerful testing framework.
