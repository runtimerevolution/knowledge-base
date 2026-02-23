# Chapter III - Advanced Testing
`(avr. time for this chapter: 2 days)`

This chapter covers advanced Capybara testing patterns including test organization, factories, database management, and CI/CD integration.

## Test Organization

### Shared Examples

```ruby
# spec/support/shared_examples/authentication.rb
RSpec.shared_examples 'requires authentication' do
  it 'redirects to login when not authenticated' do
    visit path
    expect(page).to have_current_path('/login')
  end
end

RSpec.shared_examples 'admin only' do
  context 'as regular user' do
    before { login_as(regular_user) }

    it 'shows access denied' do
      visit path
      expect(page).to have_content('Access Denied')
    end
  end

  context 'as admin' do
    before { login_as(admin_user) }

    it 'allows access' do
      visit path
      expect(page).to have_current_path(path)
    end
  end
end

# Usage in specs
RSpec.describe 'Admin Dashboard', type: :feature do
  let(:path) { '/admin/dashboard' }
  let(:regular_user) { create(:user) }
  let(:admin_user) { create(:user, :admin) }

  it_behaves_like 'requires authentication'
  it_behaves_like 'admin only'
end
```

### Shared Contexts

```ruby
# spec/support/shared_contexts/authenticated_user.rb
RSpec.shared_context 'authenticated user' do
  let(:user) { create(:user) }

  before do
    login_as(user)
  end
end

RSpec.shared_context 'with products' do
  let!(:products) { create_list(:product, 5) }
end

RSpec.shared_context 'with cart items' do
  include_context 'authenticated user'
  include_context 'with products'

  before do
    products.first(3).each do |product|
      visit product_path(product)
      click_button 'Add to Cart'
    end
  end
end

# Usage
RSpec.describe 'Checkout', type: :feature do
  include_context 'with cart items'

  it 'shows cart items' do
    visit cart_path
    expect(page).to have_selector('.cart-item', count: 3)
  end
end
```

### Helper Methods

```ruby
# spec/support/helpers/authentication_helper.rb
module AuthenticationHelper
  def login_as(user)
    visit '/login'
    fill_in 'Email', with: user.email
    fill_in 'Password', with: 'password123'
    click_button 'Login'
    expect(page).to have_current_path('/dashboard')
  end

  def logout
    click_button 'Logout'
    expect(page).to have_current_path('/login')
  end

  def login_via_api(user)
    page.set_rack_session(user_id: user.id)
  end
end

# spec/support/helpers/cart_helper.rb
module CartHelper
  def add_to_cart(product, quantity: 1)
    visit product_path(product)
    fill_in 'Quantity', with: quantity if quantity > 1
    click_button 'Add to Cart'
  end

  def cart_total
    find('#cart-total').text.gsub(/[^0-9.]/, '').to_f
  end
end

# Include in RSpec
RSpec.configure do |config|
  config.include AuthenticationHelper, type: :feature
  config.include CartHelper, type: :feature
end
```

## Using Factories

### FactoryBot Setup

```ruby
# Gemfile
group :test do
  gem 'factory_bot_rails'
end

# spec/support/factory_bot.rb
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
end
```

### Defining Factories

```ruby
# spec/factories/users.rb
FactoryBot.define do
  factory :user do
    sequence(:email) { |n| "user#{n}@example.com" }
    password { 'password123' }
    name { Faker::Name.name }

    trait :admin do
      role { 'admin' }
    end

    trait :with_avatar do
      after(:create) do |user|
        user.avatar.attach(
          io: File.open(Rails.root.join('spec/fixtures/avatar.png')),
          filename: 'avatar.png'
        )
      end
    end

    factory :admin_user, traits: [:admin]
  end
end

# spec/factories/products.rb
FactoryBot.define do
  factory :product do
    sequence(:name) { |n| "Product #{n}" }
    price { Faker::Commerce.price(range: 10..1000) }
    description { Faker::Lorem.paragraph }
    stock { 100 }

    trait :out_of_stock do
      stock { 0 }
    end

    trait :on_sale do
      sale_price { price * 0.8 }
    end

    trait :with_images do
      after(:create) do |product|
        3.times do |i|
          product.images.attach(
            io: File.open(Rails.root.join("spec/fixtures/product#{i + 1}.png")),
            filename: "product#{i + 1}.png"
          )
        end
      end
    end
  end
end

# spec/factories/orders.rb
FactoryBot.define do
  factory :order do
    user
    status { 'pending' }
    total { 0 }

    trait :with_items do
      transient do
        items_count { 3 }
      end

      after(:create) do |order, evaluator|
        create_list(:order_item, evaluator.items_count, order: order)
        order.update(total: order.order_items.sum(&:subtotal))
      end
    end

    trait :completed do
      status { 'completed' }
      completed_at { Time.current }
    end
  end
end
```

### Using Factories in Tests

```ruby
RSpec.describe 'Product Listing', type: :feature do
  let!(:products) { create_list(:product, 10) }
  let!(:sale_product) { create(:product, :on_sale, name: 'Sale Item') }
  let!(:out_of_stock) { create(:product, :out_of_stock, name: 'Unavailable') }

  it 'displays all available products' do
    visit products_path
    expect(page).to have_selector('.product-card', count: 11)
  end

  it 'shows sale badge on discounted items' do
    visit products_path
    within('.product-card', text: 'Sale Item') do
      expect(page).to have_selector('.sale-badge')
    end
  end

  it 'shows out of stock label' do
    visit products_path
    within('.product-card', text: 'Unavailable') do
      expect(page).to have_content('Out of Stock')
    end
  end
end
```

## Database Management

### Database Cleaner

```ruby
# Gemfile
group :test do
  gem 'database_cleaner-active_record'
end

# spec/support/database_cleaner.rb
RSpec.configure do |config|
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, js: true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
```

### Test Data Seeding

```ruby
# spec/support/test_data.rb
module TestData
  def seed_categories
    %w[Electronics Clothing Books].each do |name|
      Category.find_or_create_by(name: name)
    end
  end

  def seed_products
    seed_categories
    Category.all.each do |category|
      5.times do
        create(:product, category: category)
      end
    end
  end
end

RSpec.configure do |config|
  config.include TestData
end

# Usage
RSpec.describe 'Product Search', type: :feature do
  before do
    seed_products
  end

  it 'filters by category' do
    visit products_path
    select 'Electronics', from: 'Category'
    click_button 'Filter'
    expect(page).to have_selector('.product-card', count: 5)
  end
end
```

## API Testing with Capybara

### Testing API Endpoints

```ruby
# spec/features/api/users_api_spec.rb
require 'rails_helper'

RSpec.describe 'Users API', type: :feature do
  let(:user) { create(:user) }
  let(:auth_headers) { { 'Authorization' => "Bearer #{user.auth_token}" } }

  describe 'GET /api/users' do
    let!(:users) { create_list(:user, 5) }

    it 'returns list of users' do
      page.driver.get '/api/users', {}, auth_headers

      expect(page.status_code).to eq(200)
      
      json = JSON.parse(page.body)
      expect(json['users'].length).to eq(6) # 5 + authenticated user
    end
  end

  describe 'POST /api/users' do
    it 'creates a new user' do
      page.driver.post '/api/users', {
        user: {
          email: 'new@example.com',
          password: 'password123',
          name: 'New User'
        }
      }.to_json, { 'Content-Type' => 'application/json' }

      expect(page.status_code).to eq(201)
      
      json = JSON.parse(page.body)
      expect(json['user']['email']).to eq('new@example.com')
    end
  end
end
```

## Screenshot and Debugging

### Taking Screenshots

```ruby
# Manual screenshot
save_screenshot('debug.png')
save_screenshot('screenshots/login_page.png', full: true)

# Screenshot on failure (automatic with config)
RSpec.configure do |config|
  config.after(:each, type: :feature) do |example|
    if example.exception
      save_screenshot("tmp/screenshots/#{example.full_description.parameterize}.png")
    end
  end
end

# Save page HTML
save_page('debug.html')
```

### Debugging Tools

```ruby
# Print page content
puts page.body
puts page.html

# Print current URL
puts current_url
puts current_path

# Print all visible text
puts page.text

# Pause execution (with pry)
binding.pry

# Save and open page in browser
save_and_open_page

# Save and open screenshot
save_and_open_screenshot
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/capybara.yml
name: Feature Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.2
          bundler-cache: true
      
      - name: Install Chrome
        uses: browser-actions/setup-chrome@latest
      
      - name: Setup database
        env:
          RAILS_ENV: test
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
        run: |
          bundle exec rails db:create
          bundle exec rails db:schema:load
      
      - name: Run feature tests
        env:
          RAILS_ENV: test
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
        run: bundle exec rspec spec/features --format documentation
      
      - name: Upload screenshots
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: screenshots
          path: tmp/screenshots/
```

### Parallel Testing

```ruby
# Gemfile
gem 'parallel_tests'

# Run parallel tests
bundle exec parallel_rspec spec/features

# With specific count
bundle exec parallel_rspec spec/features -n 4
```

## Exercise: Complete Test Suite

Create a comprehensive test suite for an e-commerce application.

### Requirements

1. **Authentication Tests**
   - Login/logout
   - Registration
   - Password reset
   - Remember me

2. **Product Tests**
   - List products
   - Filter and sort
   - Search
   - Product details

3. **Cart Tests**
   - Add/remove items
   - Update quantities
   - Apply coupons
   - Cart persistence

4. **Checkout Tests**
   - Guest checkout
   - Logged-in checkout
   - Multiple payment methods
   - Order confirmation

5. **Admin Tests**
   - Product management
   - Order management
   - User management

### Deliverables

```
spec/
├── features/
│   ├── authentication/
│   │   ├── login_spec.rb
│   │   ├── registration_spec.rb
│   │   └── password_reset_spec.rb
│   ├── products/
│   │   ├── listing_spec.rb
│   │   ├── search_spec.rb
│   │   └── details_spec.rb
│   ├── cart/
│   │   └── cart_spec.rb
│   ├── checkout/
│   │   └── checkout_spec.rb
│   └── admin/
│       ├── products_spec.rb
│       └── orders_spec.rb
├── support/
│   ├── pages/
│   │   ├── login_page.rb
│   │   ├── product_page.rb
│   │   └── cart_page.rb
│   ├── helpers/
│   │   └── authentication_helper.rb
│   └── shared_examples/
│       └── authentication.rb
└── factories/
    ├── users.rb
    ├── products.rb
    └── orders.rb
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Organize tests with shared examples and contexts
- [ ] Create and use factories effectively
- [ ] Manage test database state
- [ ] Debug tests with screenshots and page inspection
- [ ] Set up CI/CD for Capybara tests
- [ ] Run tests in parallel

## Next Steps

You've completed the Capybara section! Continue to [Practice Project Overview](../06_practice_project/overview.md) to apply everything you've learned.
