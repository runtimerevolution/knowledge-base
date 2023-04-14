
# Chapter III
`(avr. time for this chapter: 3 to 4 days)`

Moving on from the theoretical part, we move on to the practical part.

You'll find several topics listed below, with very little detail. The details do not exist on purpose, it is in everyone's interest to understand how you can infer, through a practical case, the same thing that would happen in real life with a client. As an engineer you know how to think, and if you have doubts you can always do what you would do in a real project, discuss ideas. You can also design your solution before trying it out.

These topics are still the most basic so you don't need to build or model a very elaborate system, if you notice they are linked to the course you just saw in the previous chapter  

***So, let's code***

# Let's do some basic code

> create a simple rails application MVC
> open a git repository and give access to your tutor
> IF you have time deploy it, you can choose the platform (render.com, heroku.com)

Let's build an application from scratch. The application will have as its theme the sale of ebooks online. A virtual store where you will have sellers and buyers. You won't have to implement the cart's logic, but some functionalities related to some actions that are performed when buying an item.

As previously mentioned, all the points below are generic, it is up to the developer who develops to build an architecture consistent with the proposed objective

 - List item (ebooks)
 - Add navigation menu with ebook and users
 - Create CRUD for the user (seller or a buyer)
 - Update Status of the user (enable or disable) 
 - Create CRUD for Ebooks
 - Update Status for the Ebook (Draft, Pending, Live)
 - Store PDF with the preview draft for download

> the emails referenced below do not need to be sent, it is only necessary to create the notification logic, or you can use **mailcatcher.me**

- buy ebook (**don't apply cart logic**), 
	- the buy button should trigger 2 actions (send mail to the user with the fee he get from the bought ebook 10% from the book price, send an email with all statistics from that ebook (how many times the ebook was bought, how many times the draft pdf has been viewed, how many times ebook has been seen, IP, browser, location, etc ... )) **note: you can find different metrics and not use these**
	- Register the book purchase
