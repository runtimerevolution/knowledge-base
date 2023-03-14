
# Chapter IV

(avr. time for this chapter: 3 days)

You'll find several topics listed below, with very little detail. The details do not exist on purpose, it is in everyone's interest to understand how you can infer, through a practical case, the same thing that would happen in real life with a client. As an engineer you know how to think, and if you have doubts you can always do what you would do in a real project, discuss ideas. You can also design your solution before trying it out.

These topics are still the most basic so you don't need to build or model a very elaborate system, if you notice they are linked to the course you just saw in the previous chapter  

***So, let's code***

# Let's do some basic code

> commit your pull request and in the end merge all the code into (main/master)
> IF you have time deploy it, you can choose the platform (render.com, heroku.com, etc ...)

- Store user image profile
- Store ebook image cover profile
- Implement Authentication  **without devise**. sign up and sign in
- on Successfully Login redirect to Source URL and not the root url
- Force to update password every 6 months
- send an welcome mail when a new user is registered
- Implement a Tags System
	-	Filter Tags on ebooks 
	-	Filter Articles by User (do not show on filter users without articles)        

> when a user has no articles it will not show on filters list
> the emails referenced below do not need to be sent, it is only necessary to create the notification logic, or you can use **mailcatcher.me**