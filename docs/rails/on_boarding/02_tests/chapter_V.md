# Chapter V - Shared Examples and Shared Contexts
`(avr. time for this chapter: 1 day)`

As your test suite grows, you'll notice patterns of repeated test code. RSpec provides powerful mechanisms for sharing test code: shared examples for reusable test cases and shared contexts for reusable setup. Using these features keeps your tests DRY and maintainable.

In this chapter, you will learn how to identify opportunities for sharing test code and implement shared examples and contexts effectively.

***So, let's share and reuse***

# Shared Examples

## What are Shared Examples?

Shared examples are reusable groups of tests that can be included in multiple spec files. They're perfect for testing common behavior across different classes or contexts.

> Reference: https://rspec.info/features/3-12/rspec-core/example-groups/shared-examples/

## Creating Shared Examples

### Steps to implement:

1. Create a shared examples file in `spec/support/shared_examples/`:
   ```ruby
   # spec/support/shared_examples/statusable.rb
   RSpec.shared_examples "a statusable model" do
     it "has a default status" do
       expect(subject.status).to be_present
     end

     it "can change status" do
       subject.status = :active
       expect(subject.status).to eq("active")
     end
   end
   ```

2. Include shared examples in your specs:
   ```ruby
   describe User do
     it_behaves_like "a statusable model"
   end

   describe Ebook do
     it_behaves_like "a statusable model"
   end
   ```

3. Configure RSpec to load support files:
   - In `spec/rails_helper.rb`, ensure support files are required:
   ```ruby
   Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }
   ```

## Passing Parameters to Shared Examples

### Steps to implement:

1. Pass a block to shared examples:
   ```ruby
   RSpec.shared_examples "requires authentication" do |action|
     it "redirects to login when not authenticated" do
       send(action)
       expect(response).to redirect_to(login_path)
     end
   end
   ```

2. Use the block parameter:
   ```ruby
   describe EbooksController do
     it_behaves_like "requires authentication", :get_index

     def get_index
       get :index
     end
   end
   ```

3. Use `let` definitions from the including context:
   ```ruby
   RSpec.shared_examples "a soft deletable" do
     it "marks as deleted instead of destroying" do
       subject.soft_delete
       expect(subject.deleted_at).to be_present
     end
   end
   ```

## Alternative Inclusion Syntax

### Steps to implement:

1. Use `include_examples` to include without a nested group
2. Use `it_should_behave_like` as an alias for `it_behaves_like`
3. Understand the difference between `include_examples` and `it_behaves_like`:
   - `it_behaves_like` creates a nested context
   - `include_examples` includes directly in current context

# Shared Contexts

## What are Shared Contexts?

Shared contexts are reusable setup blocks. They define `let` declarations, `before` hooks, and helper methods that can be included in multiple specs.

> Reference: https://rspec.info/features/3-12/rspec-core/example-groups/shared-context/

## Creating Shared Contexts

### Steps to implement:

1. Create a shared context file in `spec/support/shared_contexts/`:
   ```ruby
   # spec/support/shared_contexts/authenticated_user.rb
   RSpec.shared_context "authenticated user" do
     let(:current_user) { create(:user) }

     before do
       sign_in(current_user)
     end
   end
   ```

2. Include shared context in specs:
   ```ruby
   describe EbooksController do
     include_context "authenticated user"

     describe "GET #index" do
       it "returns success" do
         get :index
         expect(response).to be_successful
       end
     end
   end
   ```

## Common Shared Context Patterns

### Steps to implement:

1. Create an authentication context:
   - Set up a logged-in user
   - Include helper methods for different user roles

2. Create a time-frozen context:
   ```ruby
   RSpec.shared_context "frozen time" do
     let(:frozen_time) { Time.zone.parse("2024-01-15 10:00:00") }

     before { travel_to(frozen_time) }
     after { travel_back }
   end
   ```

3. Create a database setup context:
   ```ruby
   RSpec.shared_context "with seed data" do
     let!(:admin) { create(:user, :admin) }
     let!(:categories) { create_list(:category, 5) }
   end
   ```

## Metadata-based Inclusion

### Steps to implement:

1. Define shared context with metadata:
   ```ruby
   RSpec.shared_context "authenticated user", :authenticated do
     # ... setup code
   end
   ```

2. Configure automatic inclusion in `rails_helper.rb`:
   ```ruby
   RSpec.configure do |config|
     config.include_context "authenticated user", :authenticated
   end
   ```

3. Use metadata to include context:
   ```ruby
   describe EbooksController, :authenticated do
     # authenticated user context is automatically included
   end
   ```

## Organization Best Practices

### Steps to implement:

1. Organize shared examples by behavior:
   - `spec/support/shared_examples/models/` - model behaviors
   - `spec/support/shared_examples/controllers/` - controller behaviors

2. Organize shared contexts by purpose:
   - `spec/support/shared_contexts/authentication.rb`
   - `spec/support/shared_contexts/time_helpers.rb`

3. Keep shared examples focused:
   - One behavior per shared example group
   - Use descriptive names

4. Document parameters and requirements:
   - Comment what `let` definitions are expected
   - Document required subject setup

## Exercise

> Apply these concepts to the ebook application

1. Create shared examples:
   - `"a model with timestamps"` - test created_at and updated_at
   - `"a soft deletable model"` - if you implemented soft delete
   - `"a model with status"` - for User and Ebook status behavior
   - `"a controller requiring authentication"` - for protected actions

2. Create shared contexts:
   - `"authenticated user"` - logged-in user setup
   - `"authenticated seller"` - user with seller role
   - `"with published ebooks"` - seed data with published ebooks

3. Refactor existing tests:
   - Identify repeated setup code across specs
   - Extract into shared contexts
   - Identify common assertions across models
   - Extract into shared examples

4. Use metadata for automatic inclusion:
   - Tag controller specs that need authentication
   - Configure automatic context inclusion

