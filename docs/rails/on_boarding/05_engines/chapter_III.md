# Chapter III - Installable and Customizable Engines
`(avr. time for this chapter: 1 to 2 days)`

Engines are only truly useful when they can be installed and adapted by other applications without changing the engine's code. This is how popular engines such as Devise, Sidekiq, and Active Admin work.

In this chapter, you will make the `webhooks` and `media_library` engines installable with a single command, and customizable by the host application.

Continue building upon the engines from Chapters I and II.

> Reference: [Rails Guides - Improving Engine Functionality](https://guides.rubyonrails.org/engines.html#improving-engine-functionality)

## Exercise 1: Install Generators

### Requirements

- Each engine has an install generator:
  ```bash
  bin/rails generate webhooks:install
  bin/rails generate media_library:install --mount-path=/media
  ```
- The generator:
  - Creates an initializer with every configuration option documented and set to its default value
  - Mounts the engine in the host routes
  - Copies the engine migrations to the host
  - Prints the next steps (e.g., where to add credentials)
- Running the generator twice does not duplicate routes or initializers
- The manual setup is removed from the ebook store and replaced by the generators
- Each generator has a spec

> Reference: [Rails Guides - Creating and Customizing Rails Generators](https://guides.rubyonrails.org/generators.html)

## Exercise 2: Customizing Views

### Requirements

- The ebook store overrides the `media_library` images index to use its own style, without changing the engine
- Engine views are split into small partials, so the host can override only what it needs (e.g., the image card)
- A `media_library:views` generator copies the engine views to the host (similar to `rails generate devise:views`)
- The engine layout is configurable (e.g., the host can use its own `application` layout)

## Exercise 3: Extension Points

### Requirements

- The host adds a method to an engine model using a decorator loaded in `config.to_prepare`
- The host can register callbacks in the configuration (e.g., `after_upload`)
- The host can register its own handlers for new incoming webhook event types (e.g., `payment.disputed`) without changing the engine
- The engines publish events with `ActiveSupport::Notifications` (e.g., `media_library.image_uploaded`, `webhooks.event_processed`), and the host subscribes to at least one
- The engines run load hooks (`ActiveSupport.run_load_hooks`) so the host can use `ActiveSupport.on_load`
- Every configuration option and extension point is documented in the engine README

> Note: Discuss with your tutor why `to_prepare` is needed, and why explicit hooks are usually preferred over decorators.

## Exercise 4: Audit Log Engine

Build a third engine from scratch, applying everything from this section. The engine records who changed what in any model of the host application.

### Requirements

- The host enables auditing per model, e.g.:
  ```ruby
  class Ebook < ApplicationRecord
    include AuditLog::Auditable
    audited only: %i[title price status]
  end
  ```
- Each change stores the model, record id, action (create, update, destroy), changed attributes (before and after), the user who made the change, IP, and timestamp
- The current user and IP are captured without the models knowing about controllers
- Audit records are written in a Sidekiq job, so they do not slow down requests
- Audit records cannot be edited or deleted
- Views at `/audit`, only for admins:
  - List of changes with filters by model, user, action, and date
  - History of a single record (e.g., all changes to an ebook)
- API endpoint `GET /audit/api/records/:model/:id` returning the history of a record
- Audit purchases, ebooks, users, and images from the media library
- Install generator and documented configuration
- A Rake task removes audit records older than a configurable period

## Exercise 5 (Optional): Reuse in Another Application

### Requirements

- Create a new, minimal Rails application (e.g., a blog)
- Install `media_library` and `audit_log` with the generators, without changing the engines' code
- Every change you need reveals a hidden dependency on the ebook store: fix it in the engine with configuration or hooks
- Each engine has a version and a `CHANGELOG.md` following [Semantic Versioning](https://semver.org)

## Exercise 6 (Optional): Engine Assets

### Requirements

- The `media_library` engine has its own stylesheet and a Stimulus controller (e.g., to copy variant URLs)
- The assets work in the host application and in the dummy application

## Checklist

Before moving to the next section, ensure you can:

- [ ] Create an install generator for an engine
- [ ] Override engine views from the host application
- [ ] Extend engine classes safely
- [ ] Design extension points with configuration, callbacks, and events
- [ ] Build a reusable engine that works with any host model
- [ ] Identify hidden dependencies between an engine and its host application
