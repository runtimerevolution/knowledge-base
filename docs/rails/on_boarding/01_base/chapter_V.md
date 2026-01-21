# Chapter V - Practical Exercise: Ebook Store (Part 2)
`(avr. time for this chapter: 3 days)`

This chapter concludes the base onboarding with advanced features for your ebook store application. The topics are broader and will require more careful planning before implementation.

Continue building upon the application from Chapter III.

## Project Finalization

### Steps to implement:

1. Create pull requests for your features
2. Merge all completed code into the main branch
3. (Optional) Deploy the final application to a platform of your choice (e.g., [Render](https://render.com), [Heroku](https://heroku.com))

## Advanced Features

### Image Management

- Implement user profile image upload and storage
- Implement ebook cover image upload and storage

### Authentication System

Implement a custom authentication system **without using Devise**.

#### Steps to implement:

1. Create user registration (sign up) functionality
2. Create user login (sign in) functionality
3. Implement `sign_in` and `sign_out` helper methods
4. On successful login, redirect users to their previous location (not the root URL)

### Password Management

- Create a Rake task that forces password updates every 6 months
- Send a welcome email when a new user registers

> Note: For email functionality, create the notification logic only, or use [Mailcatcher](https://mailcatcher.me) for local testing.

### Tags System

#### Steps to implement:

1. Implement a tagging system for ebooks
2. Add filtering functionality:
   - Filter ebooks by tags
   - Filter ebooks by user (seller)

> Note: Users with no ebooks should not appear in the filter list.
