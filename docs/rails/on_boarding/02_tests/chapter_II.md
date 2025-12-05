# Chapter II - Model and Validation Specs
`(avr. time for this chapter: 1 day)`

Model specs are the foundation of your test suite. They test the business logic, validations, associations, and methods of your models. These are unit tests—they test small, isolated pieces of code.

In this chapter, you will learn how to properly test your Ebook, User, and Purchase models, including validations, associations, callbacks, and custom methods.

## Testing Validations

Rails validations ensure data integrity. Your tests should verify that these validations work as expected.

### Steps to implement:

1. Create tests for presence validations on your Ebook model:
   - Test that an ebook is invalid without a title
   - Test that an ebook is invalid without a price
   - Test the error messages are correct
2. Create tests for uniqueness validations on your User model:
   - Test that duplicate emails are rejected
3. Create tests for format validations:
   - Test valid and invalid email formats for User
4. Create tests for numericality validations:
   - Test that ebook price must be greater than zero
   - Test that price cannot be negative
5. Create tests for inclusion validations:
   - Test that ebook status must be one of: draft, pending, live

> Reference: [Rails Testing Guide](https://guides.rubyonrails.org/testing.html#testing-your-models)

## Testing Associations

Your ebook application has relationships between models. Test that these associations are properly defined.

### Steps to implement:

1. Test `belongs_to` associations:
   - Ebook belongs to User (seller)
   - Purchase belongs to User (buyer)
   - Purchase belongs to Ebook
2. Test `has_many` associations:
   - User has many Ebooks (as seller)
   - User has many Purchases (as buyer)
   - Ebook has many Purchases
3. Test `has_many :through` associations (if applicable):
   - User has many purchased ebooks through Purchases
4. Verify dependent destroy behavior:
   - When a user is deleted, what happens to their ebooks?

> Tip: You can use the `shoulda-matchers` gem to simplify association tests

## Testing Scopes

Your ebook application should have scopes for filtering ebooks by status.

### Steps to implement:

1. Test the `published` scope (ebooks with status 'live'):
   ```ruby
   describe '.published' do
     it 'returns only live ebooks' do
       draft_ebook = Ebook.create(title: "Draft", status: :draft, ...)
       live_ebook = Ebook.create(title: "Live", status: :live, ...)

       expect(Ebook.published).to include(live_ebook)
       expect(Ebook.published).not_to include(draft_ebook)
     end
   end
   ```
2. Test the `by_seller` scope:
   - Create ebooks for different users
   - Verify the scope returns only ebooks from a specific seller
3. Test scope chaining:
   - Example: `Ebook.published.by_seller(user)`

## Testing Instance Methods

Your models should have custom methods. Test them thoroughly.

### Steps to implement:

1. Test User status methods:
   - `user.enable!` should set status to enabled
   - `user.disable!` should set status to disabled
   - `user.enabled?` should return true/false
2. Test Ebook status transition methods:
   - `ebook.publish!` should change status from pending to live
   - `ebook.submit_for_review!` should change status from draft to pending
3. Test ebook statistics methods (if implemented):
   - `ebook.view_count`
   - `ebook.purchase_count`

## Testing Callbacks

Your ebook application likely has callbacks for sending emails and tracking statistics.

### Steps to implement:

1. Test `after_create` callback on Purchase:
   - Verify that purchase creation triggers the notification logic
2. Test `before_save` callbacks:
   - If you normalize data (e.g., downcase email), test it
3. Test callbacks that update statistics:
   - When a purchase is created, ebook statistics should update

> Warning: Avoid over-testing callbacks. Test the behavior, not the implementation.

## Exercise

Apply these concepts to your ebook application:

1. Write comprehensive model specs for `Ebook`:
   - Test all validations (title presence, status inclusion, price numericality)
   - Test the association with User (seller)
   - Test status transition methods
   - Test scopes: `published`, `by_seller`, `draft`, `pending`

2. Write comprehensive model specs for `User`:
   - Test email format validation
   - Test the enable/disable status methods
   - Test associations with ebooks (as seller)
   - Test associations with purchases (as buyer)

3. Write specs for the `Purchase` model:
   - Test the association between buyer, seller, and ebook
   - Test that a purchase records the correct price
   - Test any callbacks that trigger notifications
