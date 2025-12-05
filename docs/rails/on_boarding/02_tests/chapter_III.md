# Chapter III - Mocking and Stubbing
`(avr. time for this chapter: 1 to 2 days)`

When testing, you often need to isolate the code under test from its dependencies. Mocking and stubbing allow you to replace real objects with test doubles, making your tests faster, more reliable, and focused on the specific behavior you're testing.

In this chapter, you will learn how to use RSpec's built-in mocking framework to create test doubles, stubs, and mocks.

***So, let's mock***

# Test Doubles

## What are Test Doubles?

Test doubles are objects that stand in for real objects in your tests. They come in several flavors:

- **Dummy** - objects passed around but never used
- **Stub** - provides canned answers to calls made during the test
- **Mock** - objects pre-programmed with expectations
- **Spy** - records information about how it was called
- **Fake** - working implementations with shortcuts

> Reference: https://rspec.info/features/3-12/rspec-mocks/

## Creating Doubles

### Steps to implement:

1. Create a simple double: `double("name")`
2. Create a double with methods: `double("user", name: "John", email: "john@example.com")`
3. Use `instance_double` for verified doubles (recommended)
4. Use `class_double` for stubbing class methods

## Stubbing Methods

Stubbing replaces method implementations with predetermined responses.

### Steps to implement:

1. Stub a method on a double:
   - `allow(object).to receive(:method_name).and_return(value)`
2. Stub a method on a real object:
   - `allow(User).to receive(:find).and_return(user_double)`
3. Stub with different return values for consecutive calls:
   - `allow(object).to receive(:method).and_return(1, 2, 3)`
4. Stub to raise an exception:
   - `allow(object).to receive(:method).and_raise(SomeError)`
5. Stub with block for dynamic responses:
   - `allow(object).to receive(:method) { |arg| arg.upcase }`

## Mocking with Expectations

Mocks verify that methods are called as expected.

### Steps to implement:

1. Set expectation that a method is called:
   - `expect(object).to receive(:method_name)`
2. Verify method is called with specific arguments:
   - `expect(object).to receive(:method).with(arg1, arg2)`
3. Verify method is called a specific number of times:
   - `expect(object).to receive(:method).exactly(3).times`
4. Use argument matchers:
   - `expect(object).to receive(:method).with(anything)`
   - `expect(object).to receive(:method).with(hash_including(key: value))`

## Using Spies

Spies allow you to verify calls after the fact, which can make tests more readable.

### Steps to implement:

1. Create a spy: `spy("logger")`
2. Call methods on the spy in your test
3. Verify calls were made: `expect(logger).to have_received(:info).with("message")`

## Mocking External Services

One of the most common uses of mocking is to isolate your code from external services (APIs, email services, etc.).

### Steps to implement:

1. Mock HTTP requests using `WebMock` or `VCR` gems
2. Stub mailer deliveries in tests
3. Mock third-party API clients

> Reference: https://github.com/bblimke/webmock

## Best Practices

- **Mock what you don't own** - mock external services, not your own code
- **Don't mock the object under test** - test real behavior
- **Use verified doubles** - `instance_double` catches method name typos
- **Prefer stubs over mocks** - only mock when you need to verify calls
- **Keep mocks simple** - complex mock setups are a code smell

## Exercise

> Apply these concepts to the ebook application

1. Mock the email service:
   - When testing the purchase flow, stub the mailer to avoid sending real emails
   - Verify that the correct mailer method is called with correct arguments

2. Mock external statistics service:
   - If you have an external service for tracking (views, downloads), mock it
   - Test that your code sends the correct data

3. Stub complex queries:
   - In controller tests, stub model queries to return predictable data
   - Use `instance_double` for model instances

4. Test error handling:
   - Stub methods to raise exceptions
   - Verify your code handles errors gracefully

