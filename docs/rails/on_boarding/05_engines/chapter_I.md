# Chapter I - Webhooks Engine
`(avr. time for this chapter: 3 to 4 days)`

In this chapter, you will build a mountable engine that exposes an API dedicated to receiving webhooks from external services. The engine shares the same database as your ebook store (the MVC application) and processes every webhook asynchronously with Sidekiq. Later in the chapter, the same engine will also send webhooks to sellers.

Continue building upon the application from the previous chapters.

> Reference: [Rails Guides - Getting Started with Engines](https://guides.rubyonrails.org/engines.html)

## Exercise 1: Webhooks API

Simulate a payment provider (e.g., [Stripe](https://docs.stripe.com/webhooks)) that notifies the ebook store about the state of a purchase payment.

### Requirements

- The engine is mountable, API only, lives inside the ebook store repository, and is mounted at `/webhooks`
- The engine uses the same database as the ebook store
- Every received webhook is stored (provider, event type, external id, payload, status, error, processed at)
- The engine does not reference host classes directly (e.g., `Purchase`); they are configured by the host
- Endpoints:
  - `POST /webhooks/payments` - receives payment events
  - `GET /webhooks/health` - health check for the provider
- Events to support:
  - `payment.succeeded` - marks the purchase as paid
  - `payment.failed` - marks the purchase as failed and notifies the buyer
  - `payment.refunded` - marks the purchase as refunded and reverts the seller commission

### Steps to implement:

1. Add a payment status to the `Purchase` model (this change belongs to the host, not to the engine)
2. The controller must only verify, store the event, enqueue a job, and respond `200 OK` as fast as possible
3. Never run business logic inside the request: providers have short timeouts and retry when your application is slow

## Exercise 2: Security and Idempotency

### Requirements

- Every request is signed by the provider with an HMAC SHA256 of the raw body, sent in a header
- Requests with an invalid signature are rejected with `401 Unauthorized`
- The same event can arrive more than once and must never be processed twice (e.g., crediting the seller twice)
- Duplicated events are answered with `200 OK` without being processed again
- Secrets are never committed

### Steps to implement:

1. Use a constant-time comparison to validate signatures
2. Guarantee uniqueness at the database level, not only with validations
3. Create a Rake task that sends signed fake events to your local endpoint, to simulate the provider

## Exercise 3: Processing with Sidekiq

### Requirements

- Jobs live inside the engine but run in the same Sidekiq process as the host
- Events are processed in a dedicated `webhooks` queue
- Each event type has its own handler
- Events end up as `processed` or `failed` (with the error message)
- Purchase updates run inside a database transaction
- Existing host jobs are reused when needed (e.g., `PurchaseNotificationJob` after a successful payment)
- Temporary failures (e.g., database lock) are retried; unknown events or non-existent purchases are discarded and marked as `failed`
- Failed events are re-processed periodically

## Exercise 4: Outgoing Webhooks

Now the ebook store also sends webhooks: sellers can register a URL to be notified in real time when one of their ebooks is sold. Your application becomes the provider, and must offer the same guarantees you expected from the payment provider.

### Requirements

- Sellers can create, enable, disable, and delete their endpoints (URL and subscribed events)
- Each endpoint has a secret, shown to the seller only once
- The engine exposes a single public method for the host to trigger deliveries, e.g.:
  ```ruby
  Webhooks.deliver(owner: seller, event: "ebook.sold", payload: { purchase_id: purchase.id })
  ```
- Deliveries are sent by a Sidekiq job in a low priority queue, outside the purchase transaction
- Requests are signed (HMAC SHA256 with a timestamp) and include a unique delivery id
- Requests have short timeouts (e.g., 5 seconds)
- Every attempt is stored (status, response body, attempt number)
- Failed deliveries are retried with exponential backoff (e.g., 1 min, 5 min, 30 min, 2 h, 12 h)
- After several consecutive failures, the endpoint is disabled and the seller is notified by email

> Note: Discuss with your tutor whether to use Sidekiq retries or manage attempts in the database, and what each option means for visibility and replays.

> Reference: [Standard Webhooks](https://www.standardwebhooks.com)

### Steps to test it manually:

1. Use a request inspection service (e.g., [webhook.site](https://webhook.site)) as the endpoint URL
2. Make a purchase and inspect the received request, headers, and signature
3. Point the endpoint to an invalid URL and observe the retries in the Sidekiq Web UI

## Exercise 5: Webhooks Administration Page

An API engine can also ship views.

### Requirements

- Page at `/webhooks/admin`, only for admin users
- List incoming events and outgoing deliveries, with filters by status, provider, and event type
- Show the full payload, headers, attempts, and errors of each event or delivery
- "Replay" button that re-processes an event or re-sends a delivery

## Exercise 6: Tests

### Requirements

- The engine has its own RSpec suite, running against the dummy application
- Request specs for the API: valid signature, invalid signature, duplicated event
- Job specs for event processing and each handler
- Specs for outgoing deliveries with [WebMock](https://github.com/bblimke/webmock): signature, success, timeouts, `5xx` responses, endpoint disabled after failures
- Integration spec in the host: a `payment.succeeded` webhook marks the purchase as paid
- The engine's suite runs in your CI pipeline

## Checklist

Before moving to the next chapter, ensure you can:

- [ ] Explain the difference between a full engine and a mountable engine
- [ ] Share the host database with an engine
- [ ] Access host models from an engine without hard dependencies
- [ ] Verify webhook signatures with HMAC
- [ ] Explain why webhooks must be idempotent and processed asynchronously
- [ ] Send signed outgoing webhooks with retries and exponential backoff
- [ ] Test an engine in isolation using the dummy application
