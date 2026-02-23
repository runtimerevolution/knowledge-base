# Practice Project Overview
`(avr. time for this section: 1 week)`

This section brings together everything you've learned by building and testing a complete web application. You'll create a Task Management application and write comprehensive tests using all four frameworks: Cypress, Playwright, Cucumber, and Capybara.

## Project Description

You will build a **Task Management Application** (similar to Trello/Todoist) with the following features:

### Core Features

1. **User Authentication**
   - User registration
   - Login/logout
   - Password reset
   - Profile management

2. **Task Management**
   - Create, read, update, delete tasks
   - Task status (Todo, In Progress, Done)
   - Task priority (Low, Medium, High)
   - Due dates
   - Task descriptions

3. **Project Organization**
   - Create projects
   - Assign tasks to projects
   - Project dashboard

4. **Collaboration**
   - Share projects with other users
   - Assign tasks to team members
   - Comments on tasks

5. **Search and Filter**
   - Search tasks
   - Filter by status, priority, due date
   - Sort tasks

## Technology Stack Options

Choose one of the following stacks:

### Option A: React + Node.js (Recommended for Cypress/Playwright)

```
Frontend: React + TypeScript + Vite
Backend: Node.js + Express + PostgreSQL
Testing: Cypress, Playwright, Cucumber (JS)
```

### Option B: Ruby on Rails (Recommended for Capybara)

```
Full-stack: Ruby on Rails 7
Frontend: Hotwire (Turbo + Stimulus)
Testing: Capybara, Cucumber (Ruby)
```

### Option C: Next.js Full-Stack

```
Full-stack: Next.js 14 + TypeScript
Database: PostgreSQL with Prisma
Testing: Cypress, Playwright, Cucumber (JS)
```

## Project Structure

### React + Node.js Structure

```
task-manager/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types/
│   ├── cypress/
│   │   ├── e2e/
│   │   ├── fixtures/
│   │   └── support/
│   └── tests/
│       └── playwright/
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   └── middleware/
│   └── tests/
└── features/
    ├── *.feature
    └── step_definitions/
```

### Rails Structure

```
task-manager/
├── app/
│   ├── controllers/
│   ├── models/
│   ├── views/
│   └── javascript/
├── spec/
│   ├── features/
│   ├── factories/
│   └── support/
└── features/
    ├── *.feature
    └── step_definitions/
```

## Learning Objectives

By completing this project, you will:

1. **Apply Testing Frameworks**
   - Write E2E tests with Cypress
   - Write cross-browser tests with Playwright
   - Implement BDD with Cucumber
   - Create acceptance tests with Capybara

2. **Practice Test Patterns**
   - Page Object Model
   - Custom commands/helpers
   - Fixtures and factories
   - API mocking

3. **Implement CI/CD**
   - GitHub Actions workflows
   - Parallel test execution
   - Test reporting

4. **Handle Real-World Scenarios**
   - Authentication flows
   - CRUD operations
   - Real-time updates
   - Error handling

## Assessment Criteria

Your project will be evaluated on:

| Criteria | Weight |
|----------|--------|
| Test coverage (all features tested) | 25% |
| Code quality (clean, maintainable) | 20% |
| Test organization (page objects, helpers) | 20% |
| CI/CD implementation | 15% |
| Documentation | 10% |
| Edge cases and error handling | 10% |

## Timeline

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| Setup | 1 day | Project scaffolding, dependencies |
| Core Features | 2 days | Authentication, basic CRUD |
| Testing - Cypress | 1 day | E2E tests with Cypress |
| Testing - Playwright | 1 day | Cross-browser tests |
| Testing - Cucumber | 1 day | BDD feature files and steps |
| Testing - Capybara | 1 day | Acceptance tests (if Rails) |
| CI/CD & Documentation | 1 day | GitHub Actions, README |

## Getting Started

1. Choose your technology stack
2. Follow [Setting Up the Test Application](setup.md) to scaffold the project
3. Implement the core features
4. Complete the [Test Exercises](exercises.md)

## Resources

- [Cypress Documentation](https://docs.cypress.io)
- [Playwright Documentation](https://playwright.dev/docs/intro)
- [Cucumber Documentation](https://cucumber.io/docs)
- [Capybara Documentation](https://rubydoc.info/github/teamcapybara/capybara)

## Support

If you get stuck:

1. Review the relevant chapter in this onboarding
2. Check the framework's official documentation
3. Search for similar issues on Stack Overflow
4. Ask for help in the team Slack channel

Good luck! This project will solidify your understanding of QA testing and prepare you for real-world testing challenges.
