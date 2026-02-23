# Test Exercises

This document contains exercises for testing the Task Manager application using all four frameworks. Complete each section to build a comprehensive test suite.

## Exercise 1: Cypress Tests

### 1.1 Authentication Tests

Create `cypress/e2e/auth/login.cy.js`:

```javascript
describe('Login', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  // Exercise: Implement these tests
  
  it('displays the login form');
  
  it('logs in with valid credentials');
  
  it('shows error with invalid credentials');
  
  it('validates required fields');
  
  it('redirects to dashboard after login');
  
  it('persists session after page reload');
});
```

Create `cypress/e2e/auth/registration.cy.js`:

```javascript
describe('Registration', () => {
  beforeEach(() => {
    cy.visit('/register');
  });

  // Exercise: Implement these tests
  
  it('displays the registration form');
  
  it('registers a new user successfully');
  
  it('shows error when email already exists');
  
  it('validates password requirements');
  
  it('validates email format');
});
```

### 1.2 Task Management Tests

Create `cypress/e2e/tasks/crud.cy.js`:

```javascript
describe('Task CRUD Operations', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password123');
    cy.visit('/dashboard');
  });

  // Exercise: Implement these tests
  
  describe('Create Task', () => {
    it('creates a new task with all fields');
    it('creates a task with minimum required fields');
    it('validates required title field');
  });

  describe('Read Tasks', () => {
    it('displays all user tasks');
    it('shows task details');
    it('displays empty state when no tasks');
  });

  describe('Update Task', () => {
    it('updates task title');
    it('changes task status');
    it('changes task priority');
    it('updates due date');
  });

  describe('Delete Task', () => {
    it('deletes a task');
    it('confirms before deleting');
    it('shows success message after deletion');
  });
});
```

### 1.3 Custom Commands

Create `cypress/support/commands.js`:

```javascript
// Exercise: Implement these custom commands

Cypress.Commands.add('login', (email, password) => {
  // TODO: Implement login command
});

Cypress.Commands.add('createTask', (taskData) => {
  // TODO: Implement create task command
});

Cypress.Commands.add('deleteTask', (taskId) => {
  // TODO: Implement delete task command
});

Cypress.Commands.add('filterTasks', (filters) => {
  // TODO: Implement filter tasks command
});
```

---

## Exercise 2: Playwright Tests

### 2.1 Page Objects

Create `tests/pages/LoginPage.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  
  // Exercise: Define locators
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    // TODO: Initialize locators
  }

  // Exercise: Implement methods
  async goto() {}
  async login(email: string, password: string) {}
  async expectError(message: string) {}
  async expectLoggedIn() {}
}
```

Create `tests/pages/DashboardPage.ts`:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class DashboardPage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Exercise: Implement these methods
  async goto() {}
  async createTask(task: TaskData) {}
  async getTaskCount(): Promise<number> {}
  async filterByStatus(status: string) {}
  async filterByPriority(priority: string) {}
  async searchTasks(query: string) {}
}

interface TaskData {
  title: string;
  description?: string;
  status?: string;
  priority?: string;
  dueDate?: string;
}
```

### 2.2 Test Suites

Create `tests/e2e/tasks.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('Task Management', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    
    await loginPage.goto();
    await loginPage.login('test@example.com', 'password123');
  });

  // Exercise: Implement these tests

  test('creates a new task', async () => {
    // TODO
  });

  test('updates task status via drag and drop', async () => {
    // TODO
  });

  test('filters tasks by status', async () => {
    // TODO
  });

  test('searches for tasks', async () => {
    // TODO
  });

  test('deletes a task', async () => {
    // TODO
  });
});
```

### 2.3 API Tests

Create `tests/api/tasks.api.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Tasks API', () => {
  let authToken: string;

  test.beforeAll(async ({ request }) => {
    // TODO: Get auth token
  });

  // Exercise: Implement these API tests

  test('GET /api/tasks returns user tasks', async ({ request }) => {
    // TODO
  });

  test('POST /api/tasks creates a new task', async ({ request }) => {
    // TODO
  });

  test('PUT /api/tasks/:id updates a task', async ({ request }) => {
    // TODO
  });

  test('DELETE /api/tasks/:id deletes a task', async ({ request }) => {
    // TODO
  });

  test('GET /api/tasks with filters', async ({ request }) => {
    // TODO
  });
});
```

---

## Exercise 3: Cucumber Tests

### 3.1 Feature Files

Create `features/authentication.feature`:

```gherkin
Feature: User Authentication
  As a user
  I want to authenticate with the application
  So that I can manage my tasks

  # Exercise: Complete these scenarios

  Background:
    Given the following users exist:
      | email             | password    | name      |
      | test@example.com  | password123 | Test User |

  Scenario: Successful login
    # TODO: Write steps

  Scenario: Failed login with wrong password
    # TODO: Write steps

  Scenario: User registration
    # TODO: Write steps

  Scenario: Logout
    # TODO: Write steps
```

Create `features/tasks.feature`:

```gherkin
Feature: Task Management
  As an authenticated user
  I want to manage my tasks
  So that I can track my work

  Background:
    Given I am logged in as "test@example.com"

  # Exercise: Complete these scenarios

  Scenario: Create a new task
    # TODO: Write steps

  Scenario: View all tasks
    # TODO: Write steps

  Scenario: Update task status
    # TODO: Write steps

  Scenario: Delete a task
    # TODO: Write steps

  Scenario Outline: Filter tasks by status
    Given I have tasks with different statuses
    When I filter by status "<status>"
    Then I should only see tasks with status "<status>"

    Examples:
      | status      |
      | todo        |
      | in_progress |
      | done        |

  Scenario: Search for tasks
    # TODO: Write steps
```

### 3.2 Step Definitions

Create `features/step_definitions/auth.steps.ts`:

```typescript
import { Given, When, Then, DataTable } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

// Exercise: Implement these step definitions

Given('the following users exist:', async function (this: CustomWorld, dataTable: DataTable) {
  // TODO
});

Given('I am logged in as {string}', async function (this: CustomWorld, email: string) {
  // TODO
});

When('I login with email {string} and password {string}', async function (
  this: CustomWorld,
  email: string,
  password: string
) {
  // TODO
});

Then('I should be on the dashboard', async function (this: CustomWorld) {
  // TODO
});

Then('I should see an error message {string}', async function (
  this: CustomWorld,
  message: string
) {
  // TODO
});
```

Create `features/step_definitions/tasks.steps.ts`:

```typescript
import { Given, When, Then, DataTable } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../support/world';

// Exercise: Implement these step definitions

When('I create a task with:', async function (this: CustomWorld, dataTable: DataTable) {
  // TODO
});

When('I update the task {string} status to {string}', async function (
  this: CustomWorld,
  taskTitle: string,
  status: string
) {
  // TODO
});

When('I delete the task {string}', async function (this: CustomWorld, taskTitle: string) {
  // TODO
});

When('I filter by status {string}', async function (this: CustomWorld, status: string) {
  // TODO
});

When('I search for {string}', async function (this: CustomWorld, query: string) {
  // TODO
});

Then('I should see {int} tasks', async function (this: CustomWorld, count: number) {
  // TODO
});

Then('I should see the task {string}', async function (this: CustomWorld, taskTitle: string) {
  // TODO
});

Then('I should not see the task {string}', async function (this: CustomWorld, taskTitle: string) {
  // TODO
});
```

---

## Exercise 4: Capybara Tests (Rails)

### 4.1 Feature Specs

Create `spec/features/authentication_spec.rb`:

```ruby
require 'rails_helper'

RSpec.describe 'Authentication', type: :feature do
  let!(:user) { create(:user, email: 'test@example.com', password: 'password123') }

  # Exercise: Implement these tests

  describe 'Login' do
    it 'logs in with valid credentials' do
      # TODO
    end

    it 'shows error with invalid credentials' do
      # TODO
    end

    it 'redirects to requested page after login' do
      # TODO
    end
  end

  describe 'Registration' do
    it 'registers a new user' do
      # TODO
    end

    it 'validates required fields' do
      # TODO
    end

    it 'validates email uniqueness' do
      # TODO
    end
  end

  describe 'Logout' do
    it 'logs out the user' do
      # TODO
    end
  end
end
```

Create `spec/features/tasks_spec.rb`:

```ruby
require 'rails_helper'

RSpec.describe 'Task Management', type: :feature do
  let(:user) { create(:user) }
  let!(:tasks) { create_list(:task, 5, user: user) }

  before do
    login_as(user)
  end

  # Exercise: Implement these tests

  describe 'Viewing tasks' do
    it 'displays all user tasks' do
      # TODO
    end

    it 'shows task details' do
      # TODO
    end
  end

  describe 'Creating tasks' do
    it 'creates a new task' do
      # TODO
    end

    it 'validates required fields' do
      # TODO
    end
  end

  describe 'Updating tasks' do
    it 'updates task title' do
      # TODO
    end

    it 'changes task status' do
      # TODO
    end
  end

  describe 'Deleting tasks' do
    it 'deletes a task' do
      # TODO
    end
  end

  describe 'Filtering tasks' do
    it 'filters by status' do
      # TODO
    end

    it 'filters by priority' do
      # TODO
    end
  end

  describe 'Searching tasks' do
    it 'finds tasks by title' do
      # TODO
    end
  end
end
```

### 4.2 Page Objects

Create `spec/support/pages/login_page.rb`:

```ruby
class LoginPage
  include Capybara::DSL

  # Exercise: Implement this page object

  def visit_page
    # TODO
  end

  def fill_email(email)
    # TODO
  end

  def fill_password(password)
    # TODO
  end

  def submit
    # TODO
  end

  def login(email, password)
    # TODO
  end

  def has_error?(message)
    # TODO
  end
end
```

Create `spec/support/pages/dashboard_page.rb`:

```ruby
class DashboardPage
  include Capybara::DSL

  # Exercise: Implement this page object

  def visit_page
    # TODO
  end

  def create_task(title:, description: nil, priority: 'medium', status: 'todo')
    # TODO
  end

  def task_count
    # TODO
  end

  def has_task?(title)
    # TODO
  end

  def delete_task(title)
    # TODO
  end

  def filter_by_status(status)
    # TODO
  end

  def search(query)
    # TODO
  end
end
```

---

## Exercise 5: CI/CD Configuration

### 5.1 GitHub Actions for Cypress

Create `.github/workflows/cypress.yml`:

```yaml
# Exercise: Complete this workflow

name: Cypress Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  cypress:
    runs-on: ubuntu-latest
    
    steps:
      # TODO: Add steps for:
      # 1. Checkout code
      # 2. Setup Node.js
      # 3. Install dependencies
      # 4. Start application
      # 5. Run Cypress tests
      # 6. Upload artifacts on failure
```

### 5.2 GitHub Actions for Playwright

Create `.github/workflows/playwright.yml`:

```yaml
# Exercise: Complete this workflow

name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  playwright:
    runs-on: ubuntu-latest
    
    steps:
      # TODO: Add steps for:
      # 1. Checkout code
      # 2. Setup Node.js
      # 3. Install dependencies
      # 4. Install Playwright browsers
      # 5. Run Playwright tests
      # 6. Upload test report
```

---

## Submission Checklist

Before submitting your project, ensure you have:

### Cypress
- [ ] Login/logout tests
- [ ] Registration tests
- [ ] Task CRUD tests
- [ ] Filter and search tests
- [ ] Custom commands
- [ ] Fixtures for test data
- [ ] Page objects (optional)

### Playwright
- [ ] Page objects for all pages
- [ ] E2E test suite
- [ ] API test suite
- [ ] Visual regression tests (optional)
- [ ] Cross-browser configuration

### Cucumber
- [ ] Feature files for all features
- [ ] Step definitions
- [ ] World object with Playwright
- [ ] Hooks for setup/teardown
- [ ] Tags for test organization

### Capybara (if using Rails)
- [ ] Feature specs for all features
- [ ] Page objects
- [ ] Factories
- [ ] Shared examples
- [ ] Database cleaner configuration

### CI/CD
- [ ] GitHub Actions workflow for each framework
- [ ] Test reports
- [ ] Artifact upload on failure

### Documentation
- [ ] README with setup instructions
- [ ] Test coverage report
- [ ] Known issues/limitations

## Bonus Challenges

1. **Visual Testing**: Add visual regression tests with Playwright or Percy
2. **Performance Testing**: Add Lighthouse CI for performance metrics
3. **Accessibility Testing**: Add axe-core for a11y testing
4. **Load Testing**: Add k6 scripts for API load testing
5. **Mobile Testing**: Add mobile viewport tests

Good luck with your exercises!
