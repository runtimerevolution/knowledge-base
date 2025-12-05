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
