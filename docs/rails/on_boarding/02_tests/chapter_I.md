# Chapter I - Introduction to RSpec
`(avr. time for this chapter: 1 day)`

Testing is a crucial part of software development. In the Ruby on Rails ecosystem, RSpec is one of the most popular testing frameworks. It provides a behavior-driven development (BDD) approach that makes tests readable and maintainable.

In this chapter, the objective is to understand the basics of RSpec, how to set it up in a Rails application, and how to write your first tests.

## RSpec Setup

Before writing tests, you need to set up RSpec in your Rails application.

### Steps to implement:

1. Add the `rspec-rails` gem to your Gemfile (in the `:development` and `:test` groups)
2. Run `bundle install`
3. Initialize RSpec with `rails generate rspec:install`
4. Understand the folder structure created:
   - `spec/` - main folder for all tests
   - `spec/spec_helper.rb` - RSpec configuration
   - `spec/rails_helper.rb` - Rails-specific configuration
   - `.rspec` - command line options

> Reference: [RSpec Rails Documentation](https://rspec.info/documentation/6.0/rspec-rails/)

## Basic Test Structure

Understand the anatomy of an RSpec test file:

- `describe` blocks - group related tests
- `context` blocks - describe different scenarios
- `it` blocks - individual test cases
- `expect` - make assertions

### Steps to implement:

1. Create a simple model (e.g., `Article` with `title` and `body`)
2. Generate the spec file: `rails generate rspec:model Article`
3. Write your first test checking if the model exists
4. Run tests with `bundle exec rspec`

## RSpec Matchers

Matchers are used to define expected outcomes. Learn the most common ones:

### Steps to implement:

1. Practice using equality matchers: `eq`, `eql`, `equal`, `be`
2. Practice using comparison matchers: `be >`, `be <`, `be_between`
3. Practice using truthiness matchers: `be_truthy`, `be_falsy`, `be_nil`
4. Practice using collection matchers: `include`, `match_array`, `contain_exactly`
5. Practice using predicate matchers: `be_empty`, `be_valid`

> Reference: [RSpec Built-in Matchers](https://rspec.info/features/3-12/rspec-expectations/built-in-matchers/)

## Hooks: before and after

Hooks allow you to run code before or after tests.

### Steps to implement:

1. Use `before(:each)` to set up data for each test
2. Use `before(:all)` to set up data once for all tests in a group
3. Understand when to use `after` hooks for cleanup
4. Practice using `let` and `let!` for lazy and eager evaluation

## Exercise

> Apply these concepts to the ebook application from Chapter III of the base onboarding

1. Set up RSpec in your ebook application
2. Write basic model specs for your `Ebook` model:
   - Test that an ebook can be created with valid attributes
   - Test that an ebook without a title is invalid
   - Test the status enum values (Draft, Pending, Live)
3. Write basic model specs for your `User` model:
   - Test that a user can be created
   - Test the enable/disable status functionality
