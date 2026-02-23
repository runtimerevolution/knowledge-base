# Chapter II - Selectors and Interactions
`(avr. time for this chapter: 1 day)`

This chapter covers advanced element selection strategies, complex interactions, and working with different page components in Capybara.

## Advanced Selectors

### CSS Selectors

```ruby
# By ID
find('#user-profile')

# By class
find('.btn-primary')
find('.alert.alert-success')  # Multiple classes

# By attribute
find('[data-testid="submit-button"]')
find('[type="email"]')
find('[name="user[email]"]')

# By attribute with value
find('input[placeholder="Enter email"]')
find('button[disabled]')

# Descendant selector
find('.form-container input[type="text"]')

# Direct child
find('.menu > li')

# Sibling
find('.label + input')

# Nth child
find('ul li:nth-child(2)')
find('table tr:first-child')
find('table tr:last-child')

# Contains text (CSS4)
find('button:contains("Submit")')
```

### XPath Selectors

```ruby
# Basic XPath
find(:xpath, '//button')
find(:xpath, '//input[@type="email"]')

# By text content
find(:xpath, '//button[text()="Submit"]')
find(:xpath, '//button[contains(text(), "Submit")]')

# By partial attribute
find(:xpath, '//input[contains(@class, "form")]')

# Parent/ancestor
find(:xpath, '//span[@class="error"]/parent::div')
find(:xpath, '//input[@id="email"]/ancestor::form')

# Following sibling
find(:xpath, '//label[text()="Email"]/following-sibling::input')

# Preceding sibling
find(:xpath, '//input[@id="password"]/preceding-sibling::label')

# Multiple conditions
find(:xpath, '//input[@type="text" and @required]')
find(:xpath, '//button[@type="submit" or @class="submit"]')
```

### Custom Selectors

```ruby
# Register custom selector
Capybara.add_selector(:data_testid) do
  xpath { |id| XPath.descendant[XPath.attr(:'data-testid') == id] }
end

# Usage
find(:data_testid, 'login-button')
```

## Scoping with Within

```ruby
# Scope to a specific container
within('.login-form') do
  fill_in 'Email', with: 'test@example.com'
  fill_in 'Password', with: 'password123'
  click_button 'Login'
end

# Scope to table row
within('table tbody tr', text: 'John Doe') do
  click_link 'Edit'
end

# Scope to modal
within('.modal') do
  expect(page).to have_content('Confirm Action')
  click_button 'Confirm'
end

# Nested scoping
within('.sidebar') do
  within('.user-menu') do
    click_link 'Settings'
  end
end

# Within frame
within_frame('payment-iframe') do
  fill_in 'Card Number', with: '4242424242424242'
end

# Within window
within_window(windows.last) do
  expect(page).to have_content('New Window')
end
```

## Working with Forms

### Text Inputs

```ruby
# Fill in by label
fill_in 'Email', with: 'test@example.com'

# Fill in by name
fill_in 'user[email]', with: 'test@example.com'

# Fill in by placeholder
fill_in placeholder: 'Enter your email', with: 'test@example.com'

# Fill in by ID
fill_in 'email-input', with: 'test@example.com'

# Clear and fill
find('#email').set('')
find('#email').set('new@example.com')

# Type character by character (for autocomplete)
find('#search').send_keys('test', :enter)

# Native input methods
find('#email').native.send_keys('test@example.com')
```

### Select Dropdowns

```ruby
# Select by visible text
select 'United States', from: 'Country'

# Select by value
select 'us', from: 'Country'

# Select multiple (for multi-select)
select ['Red', 'Blue', 'Green'], from: 'Colors'

# Unselect
unselect 'Red', from: 'Colors'

# Get selected value
find('#country').value  # => 'us'

# Check if option exists
expect(page).to have_select('Country', with_options: ['USA', 'Canada', 'Mexico'])
expect(page).to have_select('Country', selected: 'United States')
```

### Checkboxes and Radio Buttons

```ruby
# Check checkbox
check 'Remember me'
check 'terms_accepted'

# Uncheck checkbox
uncheck 'Subscribe to newsletter'

# Choose radio button
choose 'Credit Card'
choose 'payment_method_credit_card'

# Verify state
expect(page).to have_checked_field('Remember me')
expect(page).to have_unchecked_field('Newsletter')

# Check specific checkbox in a group
within('.permissions') do
  check 'Read'
  check 'Write'
  uncheck 'Delete'
end
```

### File Uploads

```ruby
# Simple file upload
attach_file 'Avatar', '/path/to/image.png'

# Upload by input name
attach_file 'user[avatar]', Rails.root.join('spec/fixtures/avatar.png')

# Multiple files
attach_file 'Documents', [
  '/path/to/doc1.pdf',
  '/path/to/doc2.pdf'
]

# Hidden file input (make visible first)
attach_file 'file-input', '/path/to/file.pdf', make_visible: true

# Drag and drop (requires JS driver)
find('#dropzone').drop(Rails.root.join('spec/fixtures/file.pdf'))
```

## Working with Tables

```ruby
# Find table
expect(page).to have_table('users')

# Find row by content
within('table#users') do
  within('tr', text: 'John Doe') do
    expect(page).to have_content('john@example.com')
    click_link 'Edit'
  end
end

# Check table has specific data
expect(page).to have_table('users', with_rows: [
  { 'Name' => 'John Doe', 'Email' => 'john@example.com' },
  { 'Name' => 'Jane Doe', 'Email' => 'jane@example.com' }
])

# Get all rows
all('table#users tbody tr').each do |row|
  puts row.text
end

# Get specific cell
find('table#users tbody tr:first-child td:nth-child(2)').text
```

## Mouse and Keyboard Actions

### Mouse Actions

```ruby
# Hover
find('.dropdown-trigger').hover

# Double click
find('.item').double_click

# Right click
find('.item').right_click

# Drag and drop
find('#draggable').drag_to(find('#droppable'))

# Click at specific position
find('#canvas').click(x: 100, y: 50)
```

### Keyboard Actions

```ruby
# Press key
find('#search').send_keys(:enter)
find('#input').send_keys(:tab)

# Key combinations
find('#editor').send_keys([:control, 'a'])  # Select all
find('#editor').send_keys([:control, 'c'])  # Copy
find('#editor').send_keys([:control, 'v'])  # Paste

# Special keys
find('#input').send_keys(:backspace)
find('#input').send_keys(:delete)
find('#input').send_keys(:escape)
find('#input').send_keys(:arrow_down)

# Type text with special keys
find('#search').send_keys('test query', :enter)
```

## Working with JavaScript

### Execute JavaScript

```ruby
# Execute script (no return value)
execute_script("window.scrollTo(0, document.body.scrollHeight)")
execute_script("document.querySelector('.modal').classList.add('show')")

# Evaluate script (with return value)
title = evaluate_script("document.title")
count = evaluate_script("document.querySelectorAll('.item').length")

# Pass arguments to script
execute_script("arguments[0].click()", find('#hidden-button'))

# Scroll element into view
element = find('#target')
execute_script("arguments[0].scrollIntoView()", element)
```

### Handling Alerts

```ruby
# Accept alert
accept_alert do
  click_button 'Delete'
end

# Dismiss alert
dismiss_alert do
  click_button 'Delete'
end

# Accept confirm dialog
accept_confirm do
  click_button 'Submit'
end

# Dismiss confirm dialog
dismiss_confirm do
  click_button 'Submit'
end

# Handle prompt
accept_prompt(with: 'My answer') do
  click_button 'Ask Question'
end

# Get alert message
message = accept_alert do
  click_button 'Show Alert'
end
expect(message).to eq('Are you sure?')
```

### Working with Windows

```ruby
# Open new window
new_window = window_opened_by do
  click_link 'Open in new window'
end

# Switch to window
within_window(new_window) do
  expect(page).to have_content('New Window Content')
end

# Switch by name/title
within_window('popup') do
  # ...
end

# Get all windows
windows.count  # => 2

# Close window
new_window.close
```

## Page Object Pattern

Create `spec/support/pages/login_page.rb`:

```ruby
class LoginPage
  include Capybara::DSL

  def visit_page
    visit '/login'
    self
  end

  def fill_email(email)
    fill_in 'Email', with: email
    self
  end

  def fill_password(password)
    fill_in 'Password', with: password
    self
  end

  def submit
    click_button 'Login'
    self
  end

  def login(email, password)
    fill_email(email)
    fill_password(password)
    submit
  end

  def error_message
    find('.alert-danger').text
  end

  def has_error?(message)
    has_content?(message)
  end

  def logged_in?
    has_selector?('#user-menu')
  end
end
```

### Using Page Objects

```ruby
RSpec.describe 'Login', type: :feature do
  let(:login_page) { LoginPage.new }

  it 'logs in successfully' do
    login_page
      .visit_page
      .login('test@example.com', 'password123')

    expect(login_page).to be_logged_in
  end

  it 'shows error for invalid credentials' do
    login_page
      .visit_page
      .login('wrong@example.com', 'wrong')

    expect(login_page).to have_error('Invalid credentials')
  end
end
```

## Exercise: Complex Form Testing

Create tests for a multi-step form:

```ruby
# spec/features/checkout_spec.rb
require 'rails_helper'

RSpec.describe 'Checkout Process', type: :feature, js: true do
  let!(:product) { Product.create(name: 'Laptop', price: 999.99) }
  let!(:user) { User.create(email: 'test@example.com', password: 'password123') }

  before do
    # Login and add item to cart
    login_as(user)
    visit product_path(product)
    click_button 'Add to Cart'
    visit checkout_path
  end

  describe 'shipping information step' do
    # TODO: Test filling shipping form
    # TODO: Test form validation
    # TODO: Test navigation to next step
  end

  describe 'payment information step' do
    # TODO: Test credit card form
    # TODO: Test different payment methods
    # TODO: Test form validation
  end

  describe 'order review step' do
    # TODO: Test order summary display
    # TODO: Test applying discount code
    # TODO: Test completing order
  end

  describe 'order confirmation' do
    # TODO: Test confirmation page
    # TODO: Test confirmation email
  end
end
```

## Self-Assessment

After completing this chapter, you should be able to:

- [ ] Use CSS and XPath selectors effectively
- [ ] Scope tests with `within` blocks
- [ ] Handle all form element types
- [ ] Perform mouse and keyboard actions
- [ ] Work with JavaScript dialogs and windows
- [ ] Implement the Page Object pattern

## Next Steps

Continue to [Chapter III - Advanced Testing](chapter_III.md) to learn about test organization, factories, and CI/CD integration.
