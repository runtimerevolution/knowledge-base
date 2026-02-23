# Chapter I - Introduction to Capybara
`(avr. time for this chapter: 1 day)`

Capybara is an acceptance testing framework for Ruby web applications. It simulates how a real user would interact with your application, providing a simple and expressive DSL for writing tests.

## Why Capybara?

### Advantages

- **Expressive DSL**: Natural language-like syntax
- **Multiple Drivers**: Supports various browsers and headless options
- **Automatic Waiting**: Built-in synchronization for async operations
- **Session Management**: Easy handling of cookies and sessions
- **Integration**: Works seamlessly with RSpec, Cucumber, and Minitest
- **Rails Integration**: First-class support for Ruby on Rails

### When to Use Capybara

- Testing Ruby on Rails applications
- Integration/acceptance testing
- Feature testing with RSpec
- BDD with Cucumber (Ruby)
- Testing JavaScript-heavy applications (with JS driver)

## Setting Up Capybara

### Installation

Add to your `Gemfile`:

```ruby
group :test do
  gem 'capybara'
  gem 'rspec-rails'
  gem 'selenium-webdriver'
  gem 'webdrivers'  # Auto-manages browser drivers
end
```

Install dependencies:

```bash
bundle install
```

### Configuration with RSpec

Create `spec/support/capybara.rb`:

```ruby
require 'capybara/rspec'
require 'selenium-webdriver'

Capybara.configure do |config|
  config.default_driver = :rack_test
  config.javascript_driver = :selenium_chrome_headless
  config.default_max_wait_time = 5
  config.app_host = 'http://localhost:3000'
  config.server_host = 'localhost'
  config.server_port = 3000
end

# Register Chrome headless driver
Capybara.register_driver :selenium_chrome_headless do |app|
  options = Selenium::WebDriver::Chrome::Options.new
  options.add_argument('--headless')
  options.add_argument('--no-sandbox')
  options.add_argument('--disable-dev-shm-usage')
  options.add_argument('--window-size=1920,1080')
  
  Capybara::Selenium::Driver.new(app, browser: :chrome, options: options)
end

# Register Firefox driver
Capybara.register_driver :selenium_firefox do |app|
  Capybara::Selenium::Driver.new(app, browser: :firefox)
end

RSpec.configure do |config|
  config.include Capybara::DSL
  
  config.before(:each, type: :feature) do
    Capybara.current_driver = Capybara.default_driver
  end
  
  config.before(:each, type: :feature, js: true) do
    Capybara.current_driver = Capybara.javascript_driver
  end
end
```

Update `spec/rails_helper.rb`:

```ruby
require 'spec_helper'
require 'capybara/rspec'

Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }
```

### Project Structure

```
spec/
├── features/
│   ├── authentication_spec.rb
│   ├── products_spec.rb
│   └── cart_spec.rb
├── support/
│   ├── capybara.rb
│   └── helpers/
│       └── authentication_helper.rb
└── rails_helper.rb
```

## Capybara DSL Basics

### Navigation

```ruby
# Visit a URL
visit '/login'
visit root_path
visit user_path(user)

# Navigate back/forward
go_back
go_forward

# Refresh page
refresh

# Get current path
current_path  # => "/login"
current_url   # => "http://localhost:3000/login"
```

### Finding Elements

```ruby
# Find by CSS selector
find('.submit-button')
find('#email-input')
find('[data-testid="login-form"]')

# Find by XPath
find(:xpath, '//button[@type="submit"]')

# Find by content
find('button', text: 'Submit')
find('a', text: 'Home')

# Find by label
find_field('Email')
find_field('Password')

# Find by button text
find_button('Submit')
find_button('Login')

# Find link
find_link('Home')
find_link('Sign Up')

# Find all matching elements
all('.product-card')
all('li.item')
```

### Interacting with Elements

```ruby
# Click
click_button 'Submit'
click_link 'Home'
click_on 'Login'  # Works for both buttons and links

# Fill in forms
fill_in 'Email', with: 'test@example.com'
fill_in 'Password', with: 'password123'

# Select from dropdown
select 'United States', from: 'Country'

# Check/uncheck checkboxes
check 'Remember me'
uncheck 'Subscribe to newsletter'

# Choose radio button
choose 'Credit Card'

# Attach file
attach_file 'Avatar', '/path/to/image.png'

# Execute JavaScript
execute_script("window.scrollTo(0, document.body.scrollHeight)")
evaluate_script("document.title")
```

### Assertions

```ruby
# Page content
expect(page).to have_content('Welcome')
expect(page).to have_text('Login successful')
expect(page).not_to have_content('Error')

# Current path
expect(page).to have_current_path('/dashboard')
expect(page).to have_current_path(dashboard_path)

# Selectors
expect(page).to have_selector('.alert-success')
expect(page).to have_css('#user-menu')
expect(page).to have_xpath('//div[@class="container"]')

# Form elements
expect(page).to have_field('Email')
expect(page).to have_field('Email', with: 'test@example.com')
expect(page).to have_checked_field('Remember me')
expect(page).to have_unchecked_field('Newsletter')

# Buttons and links
expect(page).to have_button('Submit')
expect(page).to have_link('Home')
expect(page).to have_link('Home', href: '/')

# Tables
expect(page).to have_table('users')

# Count
expect(page).to have_selector('.product', count: 5)
expect(page).to have_selector('.item', minimum: 1)
expect(page).to have_selector('.item', maximum: 10)
```

## Your First Feature Test

Create `spec/features/login_spec.rb`:

```ruby
require 'rails_helper'

RSpec.describe 'User Login', type: :feature do
  describe 'login page' do
    before do
      visit '/login'
    end

    it 'displays the login form' do
      expect(page).to have_field('Email')
      expect(page).to have_field('Password')
      expect(page).to have_button('Login')
    end

    it 'has empty inputs initially' do
      expect(page).to have_field('Email', with: '')
      expect(page).to have_field('Password', with: '')
    end
  end

  describe 'successful login' do
    let!(:user) { User.create(email: 'test@example.com', password: 'password123') }

    it 'redirects to dashboard with valid credentials' do
      visit '/login'
      
      fill_in 'Email', with: 'test@example.com'
      fill_in 'Password', with: 'password123'
      click_button 'Login'
      
      expect(page).to have_current_path('/dashboard')
      expect(page).to have_content('Welcome')
    end
  end

  describe 'failed login' do
    it 'shows error with invalid credentials' do
      visit '/login'
      
      fill_in 'Email', with: 'wrong@example.com'
      fill_in 'Password', with: 'wrongpassword'
      click_button 'Login'
      
      expect(page).to have_current_path('/login')
      expect(page).to have_content('Invalid email or password')
    end

    it 'shows error with missing email' do
      visit '/login'
      
      fill_in 'Password', with: 'password123'
      click_button 'Login'
      
      expect(page).to have_content("Email can't be blank")
    end
  end
end
```

### Running Tests

```bash
# Run all feature tests
bundle exec rspec spec/features

# Run specific file
bundle exec rspec spec/features/login_spec.rb

# Run specific test
bundle exec rspec spec/features/login_spec.rb:15

# Run with documentation format
bundle exec rspec spec/features --format documentation

# Run JavaScript tests
bundle exec rspec spec/features --tag js
```

## Working with JavaScript

For tests that require JavaScript execution:

```ruby
RSpec.describe 'Dynamic Content', type: :feature, js: true do
  it 'shows modal when clicking button' do
    visit '/products'
    
    click_button 'Quick View'
    
    expect(page).to have_selector('.modal', visible: true)
    expect(page).to have_content('Product Details')
  end

  it 'updates cart count dynamically' do
    visit '/products/1'
    
    click_button 'Add to Cart'
    
    expect(page).to have_selector('#cart-count', text: '1')
  end

  it 'handles AJAX form submission' do
    visit '/contact'
    
    fill_in 'Message', with: 'Hello!'
    click_button 'Send'
    
    expect(page).to have_content('Message sent successfully')
    expect(page).not_to have_selector('.loading')
  end
end
```

## Waiting and Synchronization

Capybara automatically waits for elements, but you can customize:

```ruby
# Default wait time (set in configuration)
Capybara.default_max_wait_time = 5

# Wait for specific element
expect(page).to have_selector('.loaded', wait: 10)

# Using Capybara's synchronize
page.document.synchronize(10) do
  raise Capybara::ElementNotFound unless page.has_selector?('.dynamic-content')
end

# Using within block (scopes queries)
within('.modal') do
  expect(page).to have_content('Confirm')
  click_button 'OK'
end

# Wait for AJAX
expect(page).to have_no_selector('.loading')
```

## Exercise: Test a Registration Flow

Create feature tests for user registration:

```ruby
# spec/features/registration_spec.rb
require 'rails_helper'

RSpec.describe 'User Registration', type: :feature do
  describe 'registration page' do
    before do
      visit '/register'
    end

    # TODO: Write tests for:
    # 1. Page displays all required fields
    # 2. Successful registration with valid data
    # 3. Error when passwords don't match
    # 4. Error when email is already taken
    # 5. Error when required fields are missing
    # 6. Password strength validation
  end
end
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Install and configure Capybara with RSpec
- [ ] Navigate pages using Capybara DSL
- [ ] Find and interact with elements
- [ ] Write assertions for page content
- [ ] Handle JavaScript-enabled tests
- [ ] Understand Capybara's automatic waiting

## Next Steps

Continue to [Chapter II - Selectors and Interactions](chapter_II.md) to learn advanced element selection and interaction patterns.
