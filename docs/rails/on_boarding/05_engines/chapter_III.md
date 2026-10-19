# Chapter III - Engines in Practice: Installation, Customization and Reuse
`(avr. time for this chapter: 1 to 2 days)`

In the previous chapters, you built two engines that work inside the ebook store. But engines are only truly useful when they can be installed and adapted by other applications without changing the engine's code.

In this chapter, you will make the `webhooks` and `media_library` engines installable with a single command, and allow the host application to customize them. This is how popular engines such as Devise, Sidekiq, and Active Admin work.

Continue building upon the engines from Chapters I and II.

## Install Generator

Until now, installing an engine required several manual steps: adding an initializer, mounting the routes, and copying migrations. A generator automates all of them.

### Steps to implement:

1. Create an install generator inside each engine:
   ```
   engines/media_library/lib/generators/media_library/install/
     install_generator.rb
     templates/
       initializer.rb.tt
   ```
2. The generator should:
   - Create `config/initializers/media_library.rb` with every configuration option documented and set to its default value
   - Mount the engine in the host routes (use the `route` method)
   - Copy the engine migrations to the host
   - Print the next steps (e.g., where to add the Cloudinary credentials)
3. Accept options in the generator (e.g., `--mount-path=/media`)
4. Make the generator idempotent: running it twice must not duplicate routes or initializers
5. Remove the manual setup from the host application and run the generators:
   ```bash
   bin/rails generate webhooks:install
   bin/rails generate media_library:install
   ```
6. Write a generator spec that runs the generator in a temporary folder and checks the created files

> Reference: [Rails Guides - Creating and Customizing Rails Generators](https://guides.rubyonrails.org/generators.html)

## Customization from the Host Application

Each application that installs an engine will need small changes: a different layout, an extra field, a new behavior. The host must be able to make these changes without editing the engine's code.

### Overriding Views

Rails looks for views in the host application before looking in the engine. A host can replace any engine view by creating a file with the same path.

#### Steps to implement:

1. Override the `media_library` images index in the host (`app/views/media_library/images/index.html.erb`) to use the ebook store's style
2. Split the engine views into small partials so the host can override only what it needs (e.g., the image card) instead of the whole page
3. Create a generator `media_library:views` that copies the engine views to the host so they can be customized (similar to `rails generate devise:views`)
4. Make the engine layout configurable, so the host can use its own layout:
   ```ruby
   MediaLibrary.configure do |config|
     config.layout = "application"
   end
   ```

### Extending Engine Classes

Sometimes the host needs to add behavior to an engine's model or controller.

#### Steps to implement:

1. Add a method to `MediaLibrary::Image` from the host using a decorator module loaded in `config.to_prepare`:
   ```ruby
   # config/initializers/media_library.rb
   Rails.application.config.to_prepare do
     MediaLibrary::Image.include(ImageExtensions)
   end
   ```
2. Understand why the decorator must be loaded in `to_prepare` (code reloading in development)
3. Discuss with your tutor the risks of decorators and why configuration options and hooks are usually preferred

> Reference: [Rails Guides - Improving Engine Functionality](https://guides.rubyonrails.org/engines.html#improving-engine-functionality)

### Hooks and Callbacks

A better way to allow customization is for the engine to offer explicit extension points.

#### Steps to implement:

1. Allow the host to register callbacks in the configuration, for example:
   ```ruby
   MediaLibrary.configure do |config|
     config.after_upload = ->(image) { Analytics.track("image.uploaded", image_id: image.id) }
   end

   Webhooks.configure do |config|
     config.handlers["payment.disputed"] = "PaymentDisputedHandler"
   end
   ```
2. Allow the host to add its own handlers for incoming webhook event types without changing the engine
3. Publish engine events with `ActiveSupport::Notifications` (e.g., `media_library.image_uploaded`, `webhooks.event_processed`) so the host can subscribe
4. Use `ActiveSupport.run_load_hooks` to let the host run code when engine classes are loaded:
   ```ruby
   ActiveSupport.on_load(:media_library_image) do
     # runs in the context of MediaLibrary::Image
   end
   ```

> Reference: [Rails Guides - Configuring an Engine](https://guides.rubyonrails.org/engines.html#configuring-an-engine)

## (Optional) Reusing an Engine in Another Application

The best proof that an engine is independent is installing it in a different application.

### Steps to implement:

1. Create a new, minimal Rails application (e.g., a blog)
2. Install the `media_library` engine using the generator, pointing the gem to the ebook store repository with `path:` or `git:`
3. Make it work without changing the engine's code; every change you need reveals a hidden dependency on the ebook store
4. Fix those dependencies in the engine using configuration or hooks
5. Add a `CHANGELOG.md` and a version to each engine, following [Semantic Versioning](https://semver.org)

## (Optional) Engine Assets

Engines with views need their own CSS and JavaScript.

### Steps to implement:

1. Add a stylesheet and a Stimulus controller to the `media_library` engine (e.g., to copy variant URLs to the clipboard)
2. Make the engine assets available to the host, whichever asset pipeline it uses (Propshaft or Sprockets, importmap or jsbundling)
3. Make sure the assets work both in the host application and in the dummy application

> Note: Asset configuration in engines changed a lot between Rails versions. Read the guides for the Rails version you are using.

## Exercise

Apply these concepts to both engines:

1. Create install generators for `webhooks` and `media_library`, and use them to install both engines in the ebook store
2. Allow the host to override the `media_library` views, and create a generator to copy them
3. Make the `media_library` layout configurable
4. Extend an engine model from the host using `config.to_prepare`
5. Add configuration hooks to both engines and allow custom webhook handlers
6. Publish engine events and subscribe to at least one of them in the host
7. Document every configuration option and extension point in each engine's README

## Checklist

Before moving to the next section, ensure you can:

- [ ] Create an install generator for an engine
- [ ] Override engine views from the host application
- [ ] Extend engine classes safely with `config.to_prepare`
- [ ] Design extension points with configuration, callbacks, and events
- [ ] Explain the trade-offs between decorators and explicit hooks
- [ ] Identify hidden dependencies between an engine and its host application
