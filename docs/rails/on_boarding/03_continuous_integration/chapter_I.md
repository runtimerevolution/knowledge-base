# Chapter I - Continuous Integration
`(avr. time for this chapter: 1 to 2 days)`

Continuous Integration (CI) is the practice of automating the validation of your code. This process helps reduce the risk of introducing bugs and ensures your project remains in a deployable state at all times.

In this chapter, you will set up an automated pipeline that validates your code on every push to the repository.

## Platform Selection

Choose one of the following CI/CD platforms for this exercise:

- [GitHub Actions](https://docs.github.com/en/actions)
- [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)

## Pipeline Components

Your CI pipeline should include the following stages:

### 1. Project Setup

Configure the environment with the correct tool versions.

#### Steps to implement:

**For Bitbucket Pipelines and GitLab:**

1. Use an official Docker image from [Docker Hub](https://hub.docker.com) with the required Ruby version
2. Alternatively, create and use a custom Docker image

**For GitHub Actions:**

1. Choose the operating system (Linux, Windows, or macOS)
2. Use the official [Ruby setup action](https://github.com/ruby/setup-ruby) to configure the Ruby environment

### 2. Run Tests

Automated tests ensure your application behaves as expected as you add new features.

#### Steps to implement:

1. Configure the pipeline to run your RSpec test suite
2. Ensure all tests pass before proceeding to subsequent stages
3. Configure test output for visibility in the CI dashboard

> Note: At this point in the onboarding, you should have RSpec tests from the testing chapters.

### 3. Code Styling (Linting)

Consistent code styling improves readability and maintainability across the team.

#### Steps to implement:

1. Add [RuboCop](https://docs.rubocop.org/rubocop/1.18/installation.html) to your project
2. Configure RuboCop rules according to your team's standards
3. Add a linting step to your CI pipeline
4. Ensure the pipeline fails if linting errors are detected

### 4. Test Coverage

Test coverage metrics help identify areas of your codebase that need additional testing.

#### Steps to implement:

1. Add [SimpleCov](https://github.com/simplecov-ruby/simplecov) to your project
2. Configure SimpleCov to generate coverage reports
3. (Optional) Use [simplecov-json](https://github.com/vicentllongo/simplecov-json) for machine-readable output
4. Add coverage reporting to your CI pipeline
5. (Optional) Set minimum coverage thresholds

> Note: If parsing JSON output from the command line, consider using [jq](https://jqlang.github.io/jq/).
