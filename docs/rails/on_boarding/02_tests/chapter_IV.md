# Chapter IV - Factory Bot and Faker
`(avr. time for this chapter: 1 day)`

Creating test data is a common need in testing. Instead of manually creating Ebook, User, and Purchase objects in every test, Factory Bot provides a flexible way to build test objects. Combined with Faker, you can generate realistic random data that makes your tests more robust.

In this chapter, you will learn how to set up and use Factory Bot with Faker to create test data for your ebook application.

## Setup

### Steps to implement:

1. Add `factory_bot_rails` gem to your Gemfile (`:development, :test` group)
2. Add `faker` gem to your Gemfile
3. Run `bundle install`
4. Configure Factory Bot in `spec/rails_helper.rb`:
   ```ruby
   RSpec.configure do |config|
     config.include FactoryBot::Syntax::Methods
   end
   ```
5. Create `spec/factories/` directory for your factory files

> Reference: [Factory Bot Rails](https://github.com/thoughtbot/factory_bot_rails)

## Basic Factories

Create factories for your ebook application models.

### Steps to implement:

1. Create the User factory:
   ```ruby
   # spec/factories/users.rb
   FactoryBot.define do
     factory :user do
       name { "John Doe" }
       email { "john@example.com" }
       password { "password123" }
       status { :enabled }
     end
   end
   ```
2. Create the Ebook factory:
   ```ruby
   # spec/factories/ebooks.rb
   FactoryBot.define do
     factory :ebook do
       title { "Ruby Programming Guide" }
       price { 19.99 }
       status { :draft }
       association :seller, factory: :user
     end
   end
   ```
3. Use factories in tests:
   - `build(:user)` - creates an instance without saving
   - `create(:user)` - creates and saves to database
   - `build_stubbed(:ebook)` - creates a stub (faster, no DB)
   - `attributes_for(:ebook)` - returns a hash of attributes

## Using Faker

Faker generates realistic random data. This prevents test failures due to uniqueness constraints.

### Steps to implement:

1. Update User factory with Faker:
   ```ruby
   factory :user do
     name { Faker::Name.name }
     email { Faker::Internet.unique.email }
     password { Faker::Internet.password(min_length: 8) }
     status { :enabled }
   end
   ```
2. Update Ebook factory with Faker:
   ```ruby
   factory :ebook do
     title { Faker::Book.title }
     description { Faker::Lorem.paragraph(sentence_count: 3) }
     price { Faker::Commerce.price(range: 9.99..99.99) }
     status { :draft }
     association :seller, factory: :user
   end
   ```
3. Explore useful Faker generators for your ebook app:
   - `Faker::Book.title` - realistic book titles
   - `Faker::Book.author` - author names
   - `Faker::Commerce.price` - prices in range
   - `Faker::Internet.email` - email addresses
   - `Faker::Lorem.paragraph` - descriptions
   - `Faker::Date.backward(days: 30)` - past dates for purchases

> Reference: [Faker Ruby](https://github.com/faker-ruby/faker)

## Sequences

Use sequences to generate unique values when Faker isn't enough.

### Steps to implement:

1. Create a sequence for unique emails:
   ```ruby
   factory :user do
     sequence(:email) { |n| "user#{n}@example.com" }
   end
   ```
2. Create a sequence for ebook titles:
   ```ruby
   factory :ebook do
     sequence(:title) { |n| "Ebook Volume #{n}" }
   end
   ```

## Associations

Factory Bot handles the relationships in your ebook application elegantly.

### Steps to implement:

1. Define Ebook factory with seller association:
   ```ruby
   factory :ebook do
     title { Faker::Book.title }
     price { Faker::Commerce.price(range: 9.99..49.99) }
     association :seller, factory: :user
   end
   ```
2. Define Purchase factory with all associations:
   ```ruby
   factory :purchase do
     association :buyer, factory: :user
     association :ebook
     purchased_at { Time.current }
     amount { ebook.price }
   end
   ```
3. Override associations when creating:
   ```ruby
   seller = create(:user, name: "Seller Joe")
   buyer = create(:user, name: "Buyer Jane")
   ebook = create(:ebook, seller: seller, price: 29.99)
   purchase = create(:purchase, buyer: buyer, ebook: ebook)
   ```

## Traits

Traits allow you to define variations of your factories—perfect for ebook statuses and user roles.

### Steps to implement:

1. Define traits for Ebook statuses:
   ```ruby
   factory :ebook do
     title { Faker::Book.title }
     price { Faker::Commerce.price(range: 9.99..49.99) }
     status { :draft }
     association :seller, factory: :user

     trait :draft do
       status { :draft }
     end

     trait :pending do
       status { :pending }
     end

     trait :published do
       status { :live }
     end

     trait :expensive do
       price { Faker::Commerce.price(range: 79.99..199.99) }
     end
   end
   ```
2. Define traits for User roles:
   ```ruby
   factory :user do
     name { Faker::Name.name }
     email { Faker::Internet.unique.email }

     trait :seller do
       # Add seller-specific attributes if any
     end

     trait :buyer do
       # Add buyer-specific attributes if any
     end

     trait :disabled do
       status { :disabled }
     end
   end
   ```
3. Use traits when building:
   ```ruby
   create(:ebook, :published)
   create(:ebook, :published, :expensive)
   create(:user, :disabled)
   ```

## Callbacks and Transient Attributes

Use these for complex setup scenarios.

### Steps to implement:

1. Create a user with ebooks using transient attributes:
   ```ruby
   factory :user do
     name { Faker::Name.name }
     email { Faker::Internet.unique.email }

     transient do
       ebooks_count { 0 }
     end

     after(:create) do |user, evaluator|
       create_list(:ebook, evaluator.ebooks_count, seller: user)
     end
   end
   ```
2. Use in tests:
   ```ruby
   seller = create(:user, ebooks_count: 5)
   expect(seller.ebooks.count).to eq(5)
   ```
3. Create ebook with purchases:
   ```ruby
   factory :ebook do
     # ... other attributes ...

     transient do
       purchases_count { 0 }
     end

     after(:create) do |ebook, evaluator|
       create_list(:purchase, evaluator.purchases_count, ebook: ebook)
     end
   end
   ```

## Exercise

Apply these concepts to your ebook application:

1. Create factories for all your models:
   - `User` factory with Faker for name and email
   - `Ebook` factory with Faker for title, description, and price
   - `Purchase` factory with proper associations
   - `Tag` factory (if you implemented the tags system)

2. Add traits to your factories:
   - Ebook: `:draft`, `:pending`, `:published`, `:with_pdf`
   - User: `:enabled`, `:disabled`, `:seller_with_ebooks`

3. Create complex factories with transient attributes:
   - User with N ebooks: `create(:user, ebooks_count: 10)`
   - Ebook with N purchases: `create(:ebook, :published, purchases_count: 5)`

4. Refactor existing tests:
   - Replace all manual `User.create` and `Ebook.create` with factories
   - Use traits to simplify test setup
   - Use `build` instead of `create` when you don't need database persistence
