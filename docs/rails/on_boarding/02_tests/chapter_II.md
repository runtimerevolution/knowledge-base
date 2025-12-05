# Chapter II - Model and Validation Specs
`(avr. time for this chapter: 1 day)`

Model specs are the foundation of your test suite. They test the business logic, validations, associations, and methods of your models. These are unit tests - they test small, isolated pieces of code.

In this chapter, you will learn how to properly test your Rails models, including validations, associations, callbacks, and custom methods.

***So, let's test our models***

# Model Specs

## Testing Validations

Rails validations ensure data integrity. Your tests should verify that these validations work as expected.

### Steps to implement:

1. Create tests for presence validations
   - Test that a record is invalid without required fields
   - Test the error messages are correct
2. Create tests for uniqueness validations
   - Test that duplicate values are rejected
3. Create tests for format validations
   - Test valid and invalid email formats
4. Create tests for numericality validations
   - Test boundaries (minimum, maximum values)
5. Create tests for custom validations

> Reference: https://guides.rubyonrails.org/testing.html#testing-your-models

## Testing Associations

Models often have relationships with other models. Test that these associations are properly defined.

### Steps to implement:

1. Test `belongs_to` associations
2. Test `has_many` associations
3. Test `has_one` associations
4. Test `has_many :through` associations
5. Verify dependent destroy/nullify behavior

> Tip: You can use the `shoulda-matchers` gem to simplify association tests

## Testing Scopes

Scopes are named queries that you can use on your models. They should be tested to ensure they return the expected records.

### Steps to implement:

1. Create a scope in your model (e.g., `scope :published, -> { where(status: 'live') }`)
2. Write tests that:
   - Create test data with different statuses
   - Call the scope
   - Verify only the expected records are returned
3. Test scope chaining

## Testing Instance Methods

Custom instance methods contain business logic that should be thoroughly tested.

### Steps to implement:

1. Identify custom methods in your models
2. Write tests covering:
   - The happy path (expected behavior)
   - Edge cases
   - Error conditions
3. Test methods that modify state
4. Test methods that return values

## Testing Callbacks

Callbacks are hooks into the lifecycle of an Active Record object. They should be tested carefully.

### Steps to implement:

1. Test `before_validation` callbacks
2. Test `after_create` callbacks
3. Test `before_save` callbacks
4. Verify that callbacks trigger the expected behavior

> Warning: Avoid over-testing callbacks. Test the behavior, not the implementation.

## Exercise

> Apply these concepts to the ebook application

1. Write comprehensive model specs for `Ebook`:
   - Test all validations (title presence, status inclusion, price numericality)
   - Test the association with `User` (seller)
   - Test status transition methods
   - Test any scopes (e.g., `published`, `by_seller`)

2. Write comprehensive model specs for `User`:
   - Test the enable/disable status methods
   - Test associations with ebooks (as seller and buyer)
   - Test any custom methods for statistics

3. Write specs for the `Purchase` model (if you have one):
   - Test the association between buyer, seller, and ebook
   - Test any callbacks that send notifications

