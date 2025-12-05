# Chapter III - Mocking and Stubbing
`(avr. time for this chapter: 1 to 2 days)`

When testing, you often need to isolate the code under test from its dependencies. Mocking and stubbing allow you to replace real objects with test doubles, making your tests faster, more reliable, and focused on the specific behavior you're testing.

In this chapter, you will learn how to use RSpec's built-in mocking framework to create test doubles, stubs, and mocks—particularly useful for testing your ebook purchase flow and email notifications.

## What are Test Doubles?

Test doubles are objects that stand in for real objects in your tests. They come in several flavors:

- **Dummy** - objects passed around but never used
- **Stub** - provides canned answers to calls made during the test
- **Mock** - objects pre-programmed with expectations
- **Spy** - records information about how it was called
- **Fake** - working implementations with shortcuts

> Reference: [RSpec Mocks Documentation](https://rspec.info/features/3-12/rspec-mocks/)

## Creating Doubles

### Steps to implement:

1. Create a simple double: `double("ebook")`
2. Create a double with methods:
   ```ruby
   user_double = double("user", name: "John", email: "john@example.com")
   ```
3. Use `instance_double` for verified doubles (recommended):
   ```ruby
   ebook_double = instance_double(Ebook, title: "Ruby Guide", price: 19.99)
   ```
4. Use `class_double` for stubbing class methods:
   ```ruby
   ebook_class = class_double(Ebook)
   allow(ebook_class).to receive(:published).and_return([ebook_double])
   ```

## Stubbing Methods

Stubbing replaces method implementations with predetermined responses.

### Steps to implement:

1. Stub a method on a double:
   ```ruby
   allow(ebook).to receive(:price).and_return(29.99)
   ```
2. Stub a method on a real object (useful for User.find in controllers):
   ```ruby
   allow(User).to receive(:find).and_return(user_double)
   allow(Ebook).to receive(:published).and_return([ebook1, ebook2])
   ```
3. Stub with different return values for consecutive calls:
   ```ruby
   allow(ebook).to receive(:view_count).and_return(10, 11, 12)
   ```
4. Stub to raise an exception:
   ```ruby
   allow(ebook).to receive(:publish!).and_raise(InvalidStatusTransition)
   ```
5. Stub with block for dynamic responses:
   ```ruby
   allow(Ebook).to receive(:by_seller) { |user| ebooks.select { |e| e.seller == user } }
   ```

## Mocking with Expectations

Mocks verify that methods are called as expected—essential for testing your email notifications.

### Steps to implement:

1. Set expectation that the mailer is called when purchasing an ebook:
   ```ruby
   expect(PurchaseMailer).to receive(:seller_notification)
   ```
2. Verify method is called with specific arguments:
   ```ruby
   expect(PurchaseMailer).to receive(:seller_notification).with(seller, ebook, purchase)
   ```
3. Verify method is called a specific number of times:
   ```ruby
   expect(StatisticsTracker).to receive(:record_view).exactly(3).times
   ```
4. Use argument matchers:
   ```ruby
   expect(PurchaseMailer).to receive(:buyer_confirmation).with(anything, hash_including(ebook_id: ebook.id))
   ```

## Using Spies

Spies allow you to verify calls after the fact, which can make tests more readable.

### Steps to implement:

1. Create a spy for your mailer:
   ```ruby
   mailer_spy = spy("PurchaseMailer")
   ```
2. Perform the action in your test
3. Verify calls were made:
   ```ruby
   expect(mailer_spy).to have_received(:seller_notification).with(seller, ebook)
   ```

## Mocking External Services

Your ebook application sends emails and tracks statistics. Mock these to avoid side effects in tests.

### Steps to implement:

1. Stub mailer deliveries:
   ```ruby
   allow(PurchaseMailer).to receive(:seller_notification).and_return(double(deliver_later: true))
   allow(PurchaseMailer).to receive(:buyer_confirmation).and_return(double(deliver_later: true))
   ```
2. Mock statistics tracking service:
   ```ruby
   allow(StatisticsService).to receive(:track_view)
   allow(StatisticsService).to receive(:track_download)
   ```
3. Use WebMock for external HTTP requests (if you have external APIs):
   ```ruby
   stub_request(:post, "https://analytics.example.com/events")
     .to_return(status: 200, body: '{"success": true}')
   ```

> Reference: [WebMock](https://github.com/bblimke/webmock)

## Best Practices

- **Mock what you don't own** - mock email services, external APIs, not your own models
- **Don't mock the object under test** - test real behavior of Ebook, User, Purchase
- **Use verified doubles** - `instance_double(Ebook)` catches method name typos
- **Prefer stubs over mocks** - only mock when you need to verify calls
- **Keep mocks simple** - complex mock setups indicate design problems

## Exercise

Apply these concepts to your ebook application:

1. Mock the email service in purchase tests:
   - Stub `PurchaseMailer.seller_notification` to avoid sending real emails
   - Verify that the mailer is called with correct seller and ebook
   - Verify that `buyer_confirmation` is called with purchase details

2. Mock the statistics tracking:
   - Stub `Ebook#record_view` when testing the show action
   - Verify view count is recorded with correct data (IP, browser, etc.)
   - Stub PDF download tracking

3. Stub complex queries in controller tests:
   - Stub `Ebook.published` to return predictable data
   - Stub `User.find` to return a test user
   - Use `instance_double` for ebook instances

4. Test error handling:
   - Stub `ebook.purchase!` to raise an exception
   - Verify your controller handles the error gracefully
   - Test what happens when email delivery fails
