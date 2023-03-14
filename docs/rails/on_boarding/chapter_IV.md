
# Chapter IV
`(avr. time for this chapter: 3 days)`

This will be the last topic that closes onBoarding.
The topics are broader, which will force you to think better before starting to implement. Following the previous exercise, here's your challenge

***So, let's finish this***

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