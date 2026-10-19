# Chapter I - Rails Engines: Webhooks API
`(avr. time for this chapter: 3 to 4 days)`

As an application grows, some features become large enough to deserve their own boundaries, but not large enough to justify a separate service. Rails Engines solve this problem: an engine is a miniature Rails application that lives inside your main application, with its own models, controllers, routes, jobs, and tests.

In this chapter, you will build a mountable engine that exposes an API dedicated to receiving webhooks from external services. The engine shares the same database as your ebook store (the MVC application) and processes every webhook asynchronously with Sidekiq. Later in the chapter, the same engine will also send webhooks to sellers.

Continue building upon the application from Chapter III. This chapter assumes you have completed the Sidekiq setup from Chapter V and the testing chapters.

## Introduction to Rails Engines

Rails itself is built on engines. Gems like Devise, Sidekiq Web, and Active Storage are all engines mounted inside the host application.

### Types of Engines

- **Full engine** - shares the host application's namespace; its classes are mixed with the host's
- **Mountable engine** - isolated namespace (`isolate_namespace`), its own routes, mounted at a path in the host

For this onboarding, you will always use **mountable engines**.

### Why Use Engines?

- **Modularity** - clear boundaries between domains of the application
- **Reusability** - the same engine can be mounted in different applications
- **Ownership** - a team can own an engine without touching the rest of the codebase
- **Testability** - each engine has its own test suite running against a dummy application

> Reference: [Rails Guides - Getting Started with Engines](https://guides.rubyonrails.org/engines.html)

### Steps to implement:

1. Read the Rails Engines guide up to the "Testing an Engine" section
2. Explore the structure of an existing engine (e.g., the `sidekiq` gem's web UI or `active_storage`)
3. Discuss with your tutor when an engine is a better choice than a separate microservice

## Engine Setup

Engines usually live inside the host application, in an `engines/` (or `components/`) folder, and are loaded through the `Gemfile` as a local gem.

### Steps to implement:

1. Generate a mountable engine inside your ebook store:
   ```bash
   rails plugin new engines/webhooks --mountable --api --skip-git
   ```
2. Review the generated structure and identify:
   - `lib/webhooks/engine.rb` and the `isolate_namespace Webhooks` call
   - `webhooks.gemspec` and its dependencies
   - `config/routes.rb` (the engine's own routes)
   - `test/dummy` (the dummy application used for tests)
3. Add the engine to the host application's `Gemfile`:
   ```ruby
   gem "webhooks", path: "engines/webhooks"
   ```
4. Mount the engine in the host application's routes:
   ```ruby
   mount Webhooks::Engine, at: "/webhooks"
   ```
5. Run `bin/rails routes` in the host and confirm the engine routes are listed

> Note: The `--api` flag makes the engine's `ApplicationController` inherit from `ActionController::API`. This engine does not need views, sessions, or cookies.

## Sharing the Database

The engine does not have its own database. Its tables live in the same database as the ebook store, so the engine can read and update host models such as `Purchase` and `Ebook`.

### Steps to implement:

1. Generate the engine's models from inside the engine folder:
   - `WebhookEvent` - stores every received webhook (provider, event type, external id, payload, status, error message, processed at)
2. Notice that the engine prefixes its tables with the namespace (`webhooks_webhook_events`)
3. Choose one of the strategies to run engine migrations in the host, and discuss the trade-offs with your tutor:
   - Copy migrations to the host with `bin/rails webhooks:install:migrations`
   - Append the engine's migration paths to the host in an initializer of the engine
4. Run the migrations in the host and verify the new table in `db/schema.rb`
5. Add a unique index on `[provider, external_id]` - this will be important for idempotency

### Accessing Host Models

The engine should not depend directly on the host's classes, otherwise it can no longer be reused or tested in isolation.

#### Steps to implement:

1. Make the host classes the engine depends on configurable:
   ```ruby
   Webhooks.configure do |config|
     config.purchase_class = "Purchase"
   end
   ```
2. Create the configuration module in `lib/webhooks.rb` and an initializer in the host (`config/initializers/webhooks.rb`)
3. Inside the engine, always resolve the class through the configuration (e.g., `Webhooks.purchase_class.constantize`)

> Reference: [Rails Guides - Configuring an Engine](https://guides.rubyonrails.org/engines.html#configuring-an-engine)

## Webhooks API

Webhooks are HTTP requests sent by an external service to notify your application that something happened (a payment succeeded, a refund was issued, a file finished processing).

For this exercise, simulate a payment provider (e.g., [Stripe](https://docs.stripe.com/webhooks)) that notifies the ebook store about the state of a purchase payment.

### Endpoints

- `POST /webhooks/payments` - receives payment events
- `GET /webhooks/health` - simple health check for the provider

### Events to Support

- `payment.succeeded` - marks the purchase as paid
- `payment.failed` - marks the purchase as failed and notifies the buyer
- `payment.refunded` - marks the purchase as refunded and reverts the seller commission

> Note: You will need to add a payment status to the `Purchase` model of the host application. This change belongs to the host, not to the engine.

### Steps to implement:

1. Create the `Webhooks::PaymentsController` with a `create` action
2. The controller must only:
   - Verify the request signature
   - Persist the raw event as a `WebhookEvent` with status `pending`
   - Enqueue a job to process the event
   - Respond with `200 OK` as fast as possible
3. Never process business logic inside the request: providers have short timeouts and will retry if your application is slow

### Signature Verification

Anyone can send a request to a public endpoint. Providers sign their requests so you can confirm they are authentic.

#### Steps to implement:

1. Store a shared secret in Rails credentials (never commit secrets)
2. Compute an HMAC SHA256 of the raw request body using the secret
3. Compare it with the value of a request header (e.g., `X-Signature`) using `ActiveSupport::SecurityUtils.secure_compare`
4. Respond with `401 Unauthorized` when the signature is invalid
5. Create a Rake task or script that sends signed fake events to your local endpoint, to simulate the provider

> Reference: [OpenSSL::HMAC](https://ruby-doc.org/3.3/exts/openssl/OpenSSL/HMAC.html)

### Idempotency

Providers deliver webhooks **at least once**. The same event can arrive more than once, and your application must not process it twice (e.g., crediting the seller twice).

#### Steps to implement:

1. Use the unique index on `[provider, external_id]` to reject duplicated events
2. When a duplicated event arrives, respond with `200 OK` without enqueueing a new job
3. Write a test that sends the same event twice and verifies the purchase is only updated once

## Processing Webhooks with Sidekiq

The engine owns its jobs. They live inside the engine (`engines/webhooks/app/jobs/webhooks/`) but are processed by the same Sidekiq process as the host application.

### Steps to implement:

1. Create a `Webhooks::ProcessEventJob` that receives the `WebhookEvent` id
2. Inside the job:
   - Load the event and skip it if it was already processed
   - Route the event to a handler based on its type (e.g., `Webhooks::Handlers::PaymentSucceeded`)
   - Mark the event as `processed` (with `processed_at`) or `failed` (with the error message)
3. Wrap the purchase updates in a database transaction (see Chapter III)
4. Enqueue the existing host jobs from the handlers when needed (e.g., `PurchaseNotificationJob` after a successful payment)
5. Assign the job to a dedicated `webhooks` queue and add it to your Sidekiq configuration with the appropriate priority

### Error Handling and Retries

#### Steps to implement:

1. Configure retries for temporary failures (e.g., database lock, external timeout)
2. Discard events with unknown types or that reference a non-existent purchase, marking them as `failed`
3. Create a `Webhooks::RetryFailedEventsJob` that re-enqueues failed events, scheduled to run periodically
4. Observe the retries and failures in the Sidekiq Web UI

## Outgoing Webhooks

So far, the engine only receives webhooks. Now the ebook store will also send them: sellers can register a URL to be notified in real time when one of their ebooks is sold. Your application becomes the provider, and must offer the same guarantees you expected from the payment provider.

### Steps to implement:

1. Create the models in the engine (shared database):
   - `Webhooks::Endpoint` - the owner (configurable host class, e.g., `User`), URL, secret, subscribed events, enabled flag
   - `Webhooks::Delivery` - the endpoint, event name, payload, attempt count, response status, response body, next retry at, delivered at
2. Generate a secret for each endpoint and show it to the seller only once
3. Create a page in the host application where sellers can create, enable, disable, and delete their endpoints
4. Expose a public method in the engine for the host to trigger deliveries:
   ```ruby
   Webhooks.deliver(owner: seller, event: "ebook.sold", payload: { purchase_id: purchase.id, ... })
   ```
5. Call it after a successful purchase (outside the database transaction)

### Delivering with Sidekiq

#### Steps to implement:

1. Create a `Webhooks::DeliverJob` that sends the request for a `Webhooks::Delivery`
2. Sign each request with HMAC SHA256 using the endpoint secret, and send the signature and a timestamp in headers (the same approach you used to verify incoming webhooks)
3. Include a unique delivery id in the headers so the receiver can guarantee idempotency
4. Use short timeouts (e.g., 5 seconds); a slow receiver must not block your workers
5. Consider any `2xx` response a success; store the response status and body for every attempt
6. Retry failed deliveries with exponential backoff (e.g., 1 min, 5 min, 30 min, 2 h, 12 h)
7. After the last attempt, mark the delivery as `failed` and disable the endpoint after several consecutive failures, notifying the seller by email
8. Use a dedicated `webhooks_outgoing` queue with low priority

> Note: Sidekiq already retries failed jobs with backoff. Discuss with your tutor whether to use Sidekiq retries or manage attempts in the `Webhooks::Delivery` record, and what each option means for visibility and replays.

> Reference: [Standard Webhooks](https://www.standardwebhooks.com)

### Steps to test it manually:

1. Use a request inspection service (e.g., [webhook.site](https://webhook.site)) as the endpoint URL
2. Make a purchase and inspect the received request, headers, and signature
3. Point the endpoint to an invalid URL and observe the retries in the Sidekiq Web UI

## (Optional) Webhooks Administration Page

An API engine can also ship views. Add an internal page to inspect webhook traffic.

### Steps to implement:

1. Add a small set of views to the engine (with its own layout) mounted at `/webhooks/admin`
2. List incoming events and outgoing deliveries with filters by status, provider, and event type
3. Show the full payload, headers, attempts, and errors of each event or delivery
4. Add a "Replay" button that re-enqueues the processing of an event or the delivery of a webhook
5. Restrict access to admin users of the host application

> Note: The engine was generated with `--api`. Think about what you need to change so that only these controllers render views.

## Testing the Engine

The engine must be tested in isolation, using the dummy application, and also in the host application through integration tests.

### Steps to implement:

1. Configure RSpec inside the engine (replace the default Minitest setup)
2. Create factories for `WebhookEvent` inside the engine
3. Write request specs for the webhooks API:
   - Valid signature returns `200` and creates a `WebhookEvent`
   - Invalid signature returns `401` and creates nothing
   - Duplicated event returns `200` and does not enqueue a job
4. Write job specs for `Webhooks::ProcessEventJob` and each handler
5. Use `have_enqueued_job` to verify jobs are enqueued without running Sidekiq
6. Write specs for `Webhooks::DeliverJob` using [WebMock](https://github.com/bblimke/webmock):
   - The request is signed correctly
   - A `2xx` response marks the delivery as delivered
   - Timeouts and `5xx` responses schedule a retry
   - The endpoint is disabled after consecutive failures
7. In the host application, write an integration spec that sends a `payment.succeeded` webhook and verifies the purchase is marked as paid

> Reference: [RSpec Rails - Job Specs](https://rspec.info/features/7-0/rspec-rails/job-specs/job-spec/)

## Exercise

Apply these concepts to your ebook store:

1. Create the `webhooks` mountable API engine and mount it at `/webhooks`
2. Store every received event in the shared database
3. Verify the signature of every request
4. Process events asynchronously with Sidekiq, in a dedicated queue
5. Guarantee that duplicated events are never processed twice
6. Update purchases in the host application through the configured class
7. Send signed outgoing webhooks to sellers when their ebooks are sold, with retries and exponential backoff
8. Add the engine's test suite to your CI pipeline (see Continuous Integration chapter)
9. Document in the README how to send a fake signed webhook to your local environment

## Checklist

Before moving to the next chapter, ensure you can:

- [ ] Explain the difference between a full engine and a mountable engine
- [ ] Create and mount an engine in a host application
- [ ] Run engine migrations against the host database
- [ ] Access host models from an engine without hard dependencies
- [ ] Verify webhook signatures with HMAC
- [ ] Explain why webhooks must be idempotent and processed asynchronously
- [ ] Send signed outgoing webhooks with retries and exponential backoff
- [ ] Create and test Sidekiq jobs inside an engine
- [ ] Test an engine in isolation using the dummy application
