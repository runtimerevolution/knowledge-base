# Chapter III - Practical Exercise: Ebook Store (Part 1)
`(avr. time for this chapter: 3 to 4 days)`

Moving from theory to practice, this chapter presents a hands-on exercise where you will build a Rails application from scratch.

The requirements listed below are intentionally generic. This approach mirrors real-world scenarios where client requirements are often open to interpretation. As an engineer, you are expected to analyze, design, and implement solutions based on the given specifications. If you have questions or need clarification, discuss ideas with your tutor—just as you would in a professional environment.

These topics align with what you learned in the previous chapter, so the implementation should feel familiar.

## Project Setup

### Steps to implement:

1. Create a new Rails application using MVC architecture
2. Initialize a Git repository and grant access to your tutor
3. (Optional) Deploy the application to a platform of your choice (e.g., [Render](https://render.com), [Heroku](https://heroku.com))

## Application Requirements

Build an online ebook store application. The platform will have sellers and buyers. You will not implement shopping cart logic, but you will implement functionalities related to purchasing actions.

### Core Features

#### User Management

- Create CRUD operations for Users (can be either seller or buyer)
- Implement user status management (enable/disable)

#### Ebook Management

- Create CRUD operations for Ebooks
- Implement status workflow for Ebooks (Draft → Pending → Live)
- Store PDF preview drafts available for download

#### Navigation

- Add a navigation menu with links to Ebooks and Users sections

### Purchase Functionality

> Note: For email functionality, you do not need to send actual emails. Create the notification logic only, or use [Mailcatcher](https://mailcatcher.me) for local testing.

#### Steps to implement:

1. Implement an ebook purchase action (without cart logic)
2. The purchase button should trigger the following actions:
   - Send an email to the seller with their commission (10% of the book price)
   - Send an email with ebook statistics (examples below)
   - Register the purchase in the database

#### Suggested Statistics to Track:

- Number of times the ebook was purchased
- Number of times the preview PDF was viewed
- Number of times the ebook page was visited
- Visitor information (IP, browser, location, etc.)

> Note: You may define different metrics based on your implementation approach.

### Database Transactions (ACID Properties)

When processing a purchase, multiple database operations must succeed or fail together. This is where database transactions become essential to maintain data integrity.

#### Understanding ACID Properties:

- **Atomicity** - All operations succeed or all fail (no partial updates)
- **Consistency** - Database moves from one valid state to another
- **Isolation** - Concurrent transactions don't interfere with each other
- **Durability** - Committed changes persist even after system failure

#### Steps to implement:

1. Wrap the purchase flow in a transaction:
   - Create the Purchase record
   - Update the ebook's purchase count
   - Update the seller's earnings/balance
   - All operations must succeed, or none should persist

2. Handle transaction failures:
   - Use `ActiveRecord::Base.transaction` block
   - Raise exceptions to trigger rollback
   - Implement proper error handling and user feedback

3. Test the transaction behavior:
   - Verify that if one operation fails, all changes are rolled back
   - Simulate failures (e.g., validation errors) and confirm no partial data exists

> Reference: [Rails Active Record Transactions](https://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html)

#### Scenario to implement:

When a buyer purchases an ebook, the following must happen atomically:

1. Validate the buyer has sufficient balance (if implementing wallet)
2. Create a `Purchase` record with buyer, ebook, and amount
3. Increment the ebook's `purchase_count`
4. Credit the seller's balance with the commission (10%)
5. Send notification emails (outside the transaction)

If any step fails (e.g., ebook becomes unavailable, validation error), the entire operation should roll back, and the user should see an appropriate error message.
