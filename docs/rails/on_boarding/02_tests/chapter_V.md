# Chapter V - Shared Examples and Shared Contexts
`(avr. time for this chapter: 1 day)`

As your test suite grows, you'll notice patterns of repeated test code. Both User and Ebook have status functionality. Both need authentication in controller tests. RSpec provides powerful mechanisms for sharing test code: shared examples for reusable test cases and shared contexts for reusable setup.

In this chapter, you will learn how to identify opportunities for sharing test code and implement shared examples and contexts for your ebook application.

## What are Shared Examples?

Shared examples are reusable groups of tests that can be included in multiple spec files. They're perfect for testing common behavior—like the status functionality shared by User and Ebook models.

> Reference: [RSpec Shared Examples](https://rspec.info/features/3-12/rspec-core/example-groups/shared-examples/)

## Creating Shared Examples

### Steps to implement:

1. Create a shared example for status behavior (shared by User and Ebook):
   ```ruby
   # spec/support/shared_examples/statusable.rb
   RSpec.shared_examples "a model with status" do
     it "has a status attribute" do
       expect(subject).to respond_to(:status)
     end

     it "has a default status" do
       expect(subject.status).to be_present
     end
   end
   ```

2. Include shared examples in your model specs:
   ```ruby
   # spec/models/user_spec.rb
   describe User do
     subject { create(:user) }
     it_behaves_like "a model with status"
   end

   # spec/models/ebook_spec.rb
   describe Ebook do
     subject { create(:ebook) }
     it_behaves_like "a model with status"
   end
   ```

3. Configure RSpec to load support files in `spec/rails_helper.rb`:
   ```ruby
   Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }
   ```

## Shared Examples for Ebook Status Transitions

### Steps to implement:

1. Create shared examples for publishable models:
   ```ruby
   # spec/support/shared_examples/publishable.rb
   RSpec.shared_examples "a publishable resource" do
     describe "status transitions" do
       context "when draft" do
         before { subject.status = :draft }

         it "can be submitted for review" do
           subject.submit_for_review!
           expect(subject.status).to eq("pending")
         end
       end

       context "when pending" do
         before { subject.status = :pending }

         it "can be published" do
           subject.publish!
           expect(subject.status).to eq("live")
         end
       end
     end
   end
   ```

2. Use in Ebook spec:
   ```ruby
   describe Ebook do
     subject { create(:ebook, :draft) }
     it_behaves_like "a publishable resource"
   end
   ```

## Shared Examples for Authentication

### Steps to implement:

1. Create shared examples for protected controller actions:
   ```ruby
   # spec/support/shared_examples/requires_authentication.rb
   RSpec.shared_examples "requires authentication" do
     it "redirects to login when not authenticated" do
       expect(response).to redirect_to(login_path)
     end

     it "returns unauthorized status for API requests" do
       # If you have API endpoints
     end
   end
   ```

2. Use in controller specs:
   ```ruby
   describe EbooksController do
     describe "GET #new" do
       before { get :new }
       it_behaves_like "requires authentication"
     end

     describe "POST #create" do
       before { post :create, params: { ebook: attributes_for(:ebook) } }
       it_behaves_like "requires authentication"
     end
   end
   ```

## Shared Contexts

Shared contexts are reusable setup blocks. They define `let` declarations, `before` hooks, and helper methods.

> Reference: [RSpec Shared Context](https://rspec.info/features/3-12/rspec-core/example-groups/shared-context/)

## Creating Shared Contexts

### Steps to implement:

1. Create an authenticated user context:
   ```ruby
   # spec/support/shared_contexts/authentication.rb
   RSpec.shared_context "authenticated user" do
     let(:current_user) { create(:user) }

     before do
       sign_in(current_user)
     end
   end

   RSpec.shared_context "authenticated seller" do
     let(:current_user) { create(:user, :seller) }
     let!(:seller_ebooks) { create_list(:ebook, 3, seller: current_user) }

     before do
       sign_in(current_user)
     end
   end
   ```

2. Include shared context in controller specs:
   ```ruby
   describe EbooksController do
     include_context "authenticated seller"

     describe "GET #index" do
       it "returns seller's ebooks" do
         get :index
         expect(assigns(:ebooks)).to match_array(seller_ebooks)
       end
     end
   end
   ```

## Common Shared Contexts for Ebook Application

### Steps to implement:

1. Create a context with published ebooks:
   ```ruby
   # spec/support/shared_contexts/ebook_data.rb
   RSpec.shared_context "with published ebooks" do
     let(:seller) { create(:user) }
     let!(:published_ebooks) { create_list(:ebook, 5, :published, seller: seller) }
     let!(:draft_ebooks) { create_list(:ebook, 2, :draft, seller: seller) }
   end
   ```

2. Create a context for purchase testing:
   ```ruby
   RSpec.shared_context "with purchase setup" do
     let(:seller) { create(:user) }
     let(:buyer) { create(:user) }
     let(:ebook) { create(:ebook, :published, seller: seller, price: 29.99) }

     before do
       sign_in(buyer)
     end
   end
   ```

3. Create a frozen time context for testing time-sensitive features:
   ```ruby
   RSpec.shared_context "frozen time" do
     let(:frozen_time) { Time.zone.parse("2024-06-15 10:00:00") }

     before { travel_to(frozen_time) }
     after { travel_back }
   end
   ```

## Metadata-based Inclusion

Automatically include contexts using RSpec metadata tags.

### Steps to implement:

1. Define shared context with metadata:
   ```ruby
   RSpec.shared_context "authenticated user" do
     let(:current_user) { create(:user) }
     before { sign_in(current_user) }
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

     describe "GET #new" do
       it "returns success" do
         get :new
         expect(response).to be_successful
       end
     end
   end
   ```

## Organization Best Practices

### Steps to implement:

1. Organize shared examples by domain:
   ```
   spec/support/shared_examples/
   ├── models/
   │   ├── statusable.rb
   │   └── publishable.rb
   └── controllers/
       └── requires_authentication.rb
   ```

2. Organize shared contexts by purpose:
   ```
   spec/support/shared_contexts/
   ├── authentication.rb
   ├── ebook_data.rb
   └── time_helpers.rb
   ```

3. Document expected setup:
   ```ruby
   # Requires subject to be a model with status attribute
   # subject { create(:ebook) }
   RSpec.shared_examples "a model with status" do
     # ...
   end
   ```

## Exercise

Apply these concepts to your ebook application:

1. Create shared examples for your models:
   - `"a model with status"` - test status attribute for User and Ebook
   - `"a publishable resource"` - test status transitions for Ebook
   - `"a model with timestamps"` - test created_at and updated_at

2. Create shared examples for controllers:
   - `"requires authentication"` - test redirect for unauthenticated users
   - `"requires seller ownership"` - test that users can only edit their own ebooks

3. Create shared contexts:
   - `"authenticated user"` - logged-in user setup
   - `"authenticated seller"` - seller with existing ebooks
   - `"with published ebooks"` - seed data for listing tests
   - `"with purchase setup"` - buyer, seller, and ebook ready for purchase tests

4. Refactor existing tests:
   - Identify repeated `before` blocks across specs
   - Extract into shared contexts
   - Identify common test patterns (status, authentication)
   - Extract into shared examples

5. Use metadata for cleaner specs:
   - Tag controller specs with `:authenticated`
   - Configure automatic context inclusion
