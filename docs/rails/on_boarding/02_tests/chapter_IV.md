# Chapter IV - Factory Bot and Faker
`(avr. time for this chapter: 1 day)`

Creating test data is a common need in testing. Instead of manually creating objects or using fixtures, Factory Bot provides a flexible way to build test objects. Combined with Faker, you can generate realistic random data that makes your tests more robust.

In this chapter, you will learn how to set up and use Factory Bot with Faker to create test data efficiently.

***So, let's build factories***

# Factory Bot

## Setup

### Steps to implement:

1. Add `factory_bot_rails` gem to your Gemfile (`:development, :test` group)
2. Add `faker` gem to your Gemfile
3. Run `bundle install`
4. Configure Factory Bot in `spec/rails_helper.rb`:
   - Add `config.include FactoryBot::Syntax::Methods` inside the RSpec.configure block
5. Create `spec/factories/` directory for your factory files

> Reference: https://github.com/thoughtbot/factory_bot_rails

## Basic Factories

### Steps to implement:

1. Create a basic factory for your model:
   ```ruby
   # spec/factories/users.rb
   FactoryBot.define do
     factory :user do
       name { "John Doe" }
       email { "john@example.com" }
     end
   end
   ```
2. Use the factory in tests:
   - `build(:user)` - creates an instance without saving
   - `create(:user)` - creates and saves to database
   - `build_stubbed(:user)` - creates a stub (faster, no DB)
   - `attributes_for(:user)` - returns a hash of attributes

## Using Faker

Faker generates realistic random data. Integrate it with Factory Bot for dynamic test data.

### Steps to implement:

1. Replace static values with Faker:
   ```ruby
   factory :user do
     name { Faker::Name.name }
     email { Faker::Internet.email }
   end
   ```
2. Explore common Faker generators:
   - `Faker::Name` - names, first_name, last_name
   - `Faker::Internet` - email, username, url, password
   - `Faker::Lorem` - sentences, paragraphs, words
   - `Faker::Number` - number, decimal, between
   - `Faker::Date` - birthday, between, forward
   - `Faker::Address` - city, street_address, country

> Reference: https://github.com/faker-ruby/faker

## Sequences

Use sequences to generate unique values.

### Steps to implement:

1. Create a sequence:
   ```ruby
   factory :user do
     sequence(:email) { |n| "user#{n}@example.com" }
   end
   ```
2. Use Faker with sequence for more realistic unique data
3. Understand when to use sequences vs Faker

## Associations

Factory Bot handles model associations elegantly.

### Steps to implement:

1. Define associated factories:
   ```ruby
   factory :ebook do
     title { Faker::Book.title }
     association :seller, factory: :user
   end
   ```
2. Use shorthand when factory name matches association:
   ```ruby
   factory :ebook do
     seller  # automatically uses :seller factory
   end
   ```
3. Override associations when creating:
   ```ruby
   user = create(:user)
   ebook = create(:ebook, seller: user)
   ```

## Traits

Traits allow you to group attributes for reuse.

### Steps to implement:

1. Define traits:
   ```ruby
   factory :ebook do
     title { Faker::Book.title }
     status { :draft }

     trait :published do
       status { :live }
     end

     trait :with_high_price do
       price { 99.99 }
     end
   end
   ```
2. Use traits when building:
   - `create(:ebook, :published)`
   - `create(:ebook, :published, :with_high_price)`
3. Combine multiple traits

## Callbacks

Factory Bot supports callbacks for complex setup.

### Steps to implement:

1. Use `after(:build)` for setup before save
2. Use `after(:create)` for setup after save
3. Use `before(:create)` for modifications before save
4. Create associated records in callbacks

## Transient Attributes

Use transient attributes for conditional logic in factories.

### Steps to implement:

1. Define transient attributes:
   ```ruby
   factory :user do
     transient do
       with_ebooks { false }
       ebook_count { 3 }
     end

     after(:create) do |user, evaluator|
       if evaluator.with_ebooks
         create_list(:ebook, evaluator.ebook_count, seller: user)
       end
     end
   end
   ```
2. Use transient attributes: `create(:user, with_ebooks: true, ebook_count: 5)`

## Exercise

> Apply these concepts to the ebook application

1. Create factories for all your models:
   - `User` factory with traits for `:seller` and `:buyer`
   - `Ebook` factory with traits for each status (`:draft`, `:pending`, `:live`)
   - `Purchase` factory with proper associations

2. Use Faker appropriately:
   - Book titles for ebooks
   - Prices in a reasonable range
   - User names and emails
   - Dates for purchases

3. Create a complex factory:
   - User with multiple ebooks using transient attributes
   - Ebook with purchase history

4. Refactor existing tests:
   - Replace manual object creation with factories
   - Use traits to simplify test setup

