# Authentication

When building a front-end web application in React, you will most likely end up needing a mechanism to perform authenticated requests to your Rails application.

To achieve that, usually the best approach is to implement a way to store some credentials in your front-end, either via cookies or local storage.

## The Process

In general, when implementing any type of authentication in your app, you will want to follow these guidelines:

* <b style="color:#ff3f33">(Backend)</b> Set up an endpoint to create/start a new session and to return the session credentials (either via response payload or via headers)
* <b style="color:#00b1ee">(Frontend)</b> Store the returned credentials somewhere in your app (can be browser LocalStorage, Cookies, or something else custom)
* <b style="color:#00b1ee">(Frontend)</b> Implement some call to your backend API and include the stored session credentials, via request headers for example
* <b style="color:#ff3f33">(Backend)</b> Validate the credentials received from the request, and deny/accept the request as necessary.
* <b style="color:#ff3f33">(Backend)</b> If the request had invalid/expired credentials, return some error. Otherwise, execute the requested endpoint
* <b style="color:#00b1ee">(Frontend)</b> Add some error handling for the scenarios where your request may be invalid or if your credentials may have already expired.

## All-in-One Guide

Additionally, we have a more complete guide on building [authentication between Rails and React apps](https://medium.com/runtime-revolution/good-practices-for-communicating-between-your-rails-and-react-apps-2955c995566f){ target=_blank }.

This guide contains recommendations on how to set up a simple authentication mechanism, using JWT tokens and Cookies, to communicate with the back-end Rails app and to store the credentials, respectively.
