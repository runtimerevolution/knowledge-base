# Chapter II - Rails Engines: Image Management with Cloudinary
`(avr. time for this chapter: 2 to 3 days)`

In the previous chapter, you built an API-only engine. Engines can also ship their own views, layouts, and assets, which makes them ideal for internal tools such as admin panels or media libraries.

In this chapter, you will build a mountable engine to manage images stored in [Cloudinary](https://cloudinary.com). The engine will provide both:

- **Views** - a visual interface to list, upload, preview, edit, and delete images
- **API endpoints** - a JSON API to perform the same CRUD operations programmatically

Continue building upon the application from Chapter III. This chapter assumes you have completed Chapter I of this section and the image management features from Chapter IV (user profile images and ebook covers).

## Cloudinary Setup

Cloudinary is a cloud service to store, transform, and deliver images and videos. Images are uploaded once and can be resized, cropped, or optimized on the fly through the URL.

### Steps to implement:

1. Create a free Cloudinary account
2. Store the Cloudinary credentials (cloud name, API key, API secret) in Rails credentials
3. Explore the Cloudinary Media Library and upload an image manually
4. Experiment with URL transformations (e.g., width, height, crop, format, quality)

> Reference: [Cloudinary Ruby on Rails SDK](https://cloudinary.com/documentation/rails_integration)

> Note: Never commit your Cloudinary credentials. Use Rails credentials or environment variables.

## Engine Setup

Unlike the webhooks engine, this engine is a full-stack mountable engine, with its own layout, views, and assets.

### Steps to implement:

1. Generate a mountable engine inside your ebook store:
   ```bash
   rails plugin new engines/media_library --mountable --skip-git
   ```
2. Add the `cloudinary` gem as a dependency in `media_library.gemspec` (not in the host `Gemfile`)
3. Add the engine to the host `Gemfile` and mount it:
   ```ruby
   mount MediaLibrary::Engine, at: "/media"
   ```
4. Configure the Cloudinary client in an initializer of the engine, reading the credentials from the host application
5. Create the engine's own layout (`app/views/layouts/media_library/application.html.erb`) with a navigation link back to the ebook store

## Database Model

Cloudinary stores the files, but the application should keep track of them. The engine shares the same database as the host application.

### Steps to implement:

1. Create a `MediaLibrary::Image` model with:
   - `public_id` - the Cloudinary identifier (unique)
   - `title` and `description`
   - `folder` - to organize images (e.g., `covers`, `avatars`)
   - `format`, `width`, `height`, and `bytes` - metadata returned by Cloudinary
   - `secure_url` - the delivery URL
2. Run the engine migrations against the host database (use the same strategy you chose in Chapter I)
3. Add validations for the required fields and allowed formats (e.g., `jpg`, `png`, `webp`)
4. Keep Cloudinary and the database consistent:
   - If the upload to Cloudinary fails, the record must not be created
   - If the record fails to be saved, the uploaded image must be removed from Cloudinary

> Note: Database transactions do not cover external services. Think about how to handle failures that happen between the external call and the database write, and discuss your approach with your tutor.

## Cloudinary Service Layer

Both the views and the API need to talk to Cloudinary. Instead of calling the Cloudinary SDK from controllers, encapsulate it in a service.

### Steps to implement:

1. Create a `MediaLibrary::CloudinaryService` with the following methods:
   - `upload(file, folder:)` - uploads an image and returns its metadata
   - `update(public_id, attributes)` - updates tags or context, or renames the image
   - `destroy(public_id)` - deletes the image from Cloudinary
   - `url_for(public_id, transformation)` - builds a transformed URL (e.g., thumbnails)
2. Handle Cloudinary errors and raise a custom error class (e.g., `MediaLibrary::UploadError`)
3. Use this service in both the HTML and the API controllers

## Image Transformations and Variants

Cloudinary can transform images on the fly (resize, crop, change format, apply effects) just by changing the delivery URL. Instead of letting every part of the application build its own transformations, the engine should offer a catalog of ready-made **variants**. Users browse an image already prepared in every variant and choose the one they want to use.

### Suggested Variants

- `thumbnail` - small square crop for lists (e.g., 150x150, `crop: fill`, `gravity: auto`)
- `avatar` - circular crop focused on the face (e.g., 200x200, `gravity: face`, `radius: max`)
- `cover` - ebook cover proportion (e.g., 600x900, `crop: fill`)
- `banner` - wide image for headers (e.g., 1200x400, `crop: fill`)
- `grayscale` - any of the above with the `grayscale` effect

> Note: You may define different variants based on your ebook store needs. Discuss them with your tutor.

### Steps to implement:

1. Define the variants in a single place in the engine (e.g., a `MediaLibrary::Variants` registry or a YAML file), each with a name, a description, and its Cloudinary transformation
2. Allow the host application to add or override variants through the engine configuration:
   ```ruby
   MediaLibrary.configure do |config|
     config.variants[:cover] = { width: 600, height: 900, crop: :fill, gravity: :auto }
   end
   ```
3. Add a `MediaLibrary::Image#variant_url(name)` method that uses `CloudinaryService#url_for` and raises an error for unknown variants
4. Always deliver variants with automatic format and quality (`fetch_format: :auto`, `quality: :auto`)
5. (Optional) Create the variants as [named transformations](https://cloudinary.com/documentation/image_transformations#named_transformations) in Cloudinary and reference them by name
6. (Optional) Generate the variants at upload time with [eager transformations](https://cloudinary.com/documentation/eager_and_incoming_transformations), using a Sidekiq job, so the first visitor does not wait for them to be generated

> Reference: [Cloudinary Image Transformations](https://cloudinary.com/documentation/image_transformations)

### Choosing a Variant

#### Steps to implement:

1. On the image show page, display a gallery with the image rendered in every available variant, including its name, dimensions, and URL
2. Add a "Use this variant" action that lets the user pick a variant for a specific purpose (e.g., ebook cover, profile image)
3. Store the choice in the shared database (e.g., a `MediaLibrary::ImageUsage` model with the image, the variant name, and a polymorphic reference to the record using it, such as an `Ebook` or a `User`)
4. Add a "Copy URL" button next to each variant so it can be used outside the application
5. (Optional) Allow users to build a custom variant with a form (width, height, crop mode, effect), preview it live, and save it as a new variant

## Views: Visual Image Management

The views give non-technical users a way to manage the images visually.

### Pages to Implement

- **Index** - a grid of thumbnails with title, folder, and dimensions
- **Show** - the image in full size, its metadata, and the gallery of variants to choose from
- **New** - an upload form with title, description, and folder
- **Edit** - a form to update title, description, folder, or replace the file
- **Delete** - a button with confirmation that removes the image from Cloudinary and the database

### Steps to implement:

1. Create `MediaLibrary::ImagesController` with the standard CRUD actions
2. Use Cloudinary URL transformations to display thumbnails in the index (never load the original image in a list)
3. Add pagination to the index page
4. Add filters to the index page:
   - Filter by folder
   - Search by title
5. Display flash messages for successful and failed operations
6. Use partials to keep the views DRY (see Chapter II of the base onboarding)
7. (Optional) Upload directly from the browser to Cloudinary using a signed upload, so the file does not pass through your server

### Restricting Access

The media library should not be public.

#### Steps to implement:

1. Reuse the authentication system from Chapter IV of the base onboarding
2. Make the engine's `ApplicationController` delegate authentication to the host, without hard-coding the host's classes (use configuration, as in Chapter I):
   ```ruby
   MediaLibrary.configure do |config|
     config.parent_controller = "::ApplicationController"
   end
   ```
3. Only signed-in sellers should be able to access the media library

## API Endpoints

The API allows other parts of the application (or external clients) to manage images without using the views.

### Endpoints

- `GET /media/api/images` - list images (with pagination and filters)
- `GET /media/api/images/:id` - show an image and its metadata
- `POST /media/api/images` - upload a new image
- `PATCH /media/api/images/:id` - update an image
- `DELETE /media/api/images/:id` - delete an image
- `GET /media/api/variants` - list the available variants and their transformations
- `GET /media/api/images/:id/variants` - list the URLs of an image in every variant
- `POST /media/api/images/:id/usages` - choose a variant of an image for a record (e.g., an ebook cover)

### Steps to implement:

1. Create a namespaced `MediaLibrary::Api::ImagesController` that inherits from `ActionController::API`
2. Define the routes inside an `api` namespace in the engine's `config/routes.rb`
3. Return consistent JSON responses:
   - Serialize images with the same set of attributes in every endpoint
   - Return validation errors with status `422`
   - Return `404` when an image does not exist
4. Protect the API with token authentication (e.g., an API token per user stored in the host database)
5. Accept file uploads with `multipart/form-data`
6. Document the API endpoints in the engine's README with request and response examples

> Note: The API controllers and the HTML controllers must share the same service. If you find yourself duplicating logic between them, move it to the service or to the model.

## Background Processing

Uploads and deletions in Cloudinary can be slow. Reuse what you learned in Chapter V of the base onboarding.

### Steps to implement:

1. Create a `MediaLibrary::DestroyImageJob` that removes the image from Cloudinary in the background after the record is deleted
2. Create a `MediaLibrary::SyncImagesJob` that compares the database with Cloudinary and reports images that exist only in one of them
3. Schedule `SyncImagesJob` to run daily
4. (Optional) Receive Cloudinary upload notifications through the webhooks engine from Chapter I

## Integration with the Ebook Store

The media library becomes useful when the host application starts using it.

### Steps to implement:

1. Allow sellers to pick an ebook cover from the media library, choosing both the image and the variant (e.g., `cover`), instead of uploading a new file
2. Allow users to pick their profile image the same way, using the `avatar` variant
3. Display ebook covers and user profile images using the chosen variants
4. Prevent deleting images that are being used as ebook covers or profile images

## Testing the Engine

### Steps to implement:

1. Configure RSpec inside the engine and use the dummy application
2. Never call Cloudinary in tests: stub `MediaLibrary::CloudinaryService` or use [WebMock](https://github.com/bblimke/webmock) (see Chapter III of the tests onboarding)
3. Write model specs for `MediaLibrary::Image` validations and `variant_url` (including unknown variants)
4. Write specs that verify variants overridden by the host configuration are used
5. Write service specs for `MediaLibrary::CloudinaryService`, including failure scenarios
6. Write request specs for every API endpoint, including authentication failures
7. Write system or request specs for the HTML pages (index, upload, edit, delete, choose variant)
8. Write job specs for `DestroyImageJob` and `SyncImagesJob`
9. Add the engine's test suite to your CI pipeline

## Exercise

Apply these concepts to your ebook store:

1. Create the `media_library` mountable engine and mount it at `/media`
2. Store image metadata in the shared database and the files in Cloudinary
3. Build the visual interface to list, upload, preview, edit, and delete images
4. Build the JSON API with the same CRUD operations
5. Share the Cloudinary logic between views and API through a service
6. Offer a catalog of image variants, show every image prepared in each variant, and let users choose which one to use
7. Restrict access to signed-in sellers in the views and to token holders in the API
8. Move slow Cloudinary operations to Sidekiq jobs
9. Use the media library and its variants for ebook covers and user profile images

## Checklist

Before finishing this chapter, ensure you can:

- [ ] Create a mountable engine with its own views, layout, and assets
- [ ] Expose both HTML and JSON endpoints from the same engine
- [ ] Integrate an external storage service (Cloudinary) through a service layer
- [ ] Keep an external service and the database consistent when failures happen
- [ ] Use image transformations to deliver optimized images
- [ ] Define reusable image variants and let the host application configure them
- [ ] Reuse the host application's authentication inside an engine
- [ ] Test an engine without calling external services
