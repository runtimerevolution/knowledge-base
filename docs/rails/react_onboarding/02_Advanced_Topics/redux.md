# Redux (Legacy)

![](../../../assets/rails/react/redux.webp){ width=200 align=right style="margin-top:-50px;" }

Redux is a global state management library. It has an official integration with React, called [React-Redux](https://react-redux.js.org/){ target=_blank }.

## :warning: Read before using

Before the advent of functional programming and the existence of hooks, Redux was a frequently used tool, when you needed to make data available across your entire React app.
This "overall state" approach is, however, not without its drawbacks. Much like the `ContextProviders` and `useContext` hooks offered by React, which should be used sparingly, Redux suffers from the same issue.
If you start accessing the global "store" in multiple places of your app, you may unintentionally be decreasing its performance, because you might be passing too much information to each component, all the time.
This is because whenever any part of this state is changed, usually all your components accessing it will have to re-render. <sup>[1]</sup>

Nowadays, Redux also offers a hook for fetching data from the global store, `useSelector`, where you can pass an arrow function to select just the value(s) you need from the store.
If you need to build more complex state management, usually Redux is coupled with the `reselect` package, that allows creating custom selectors with better optimizations.

In a nutshell, if you are starting a new React application, we advise you to stick to React hooks and internal state management per component, with the exception here and there for sharing contexts.
You should only opt for Redux if you have a strict restriction that forces you to use it, or if you're maintaining an ongoing project that uses it!

<sup>[1]</sup> *There are tools within Redux that can help mitigate this issue, namely by implementing [memoized selectors](https://redux.js.org/usage/deriving-data-selectors#writing-memoized-selectors-with-reselect).*

## Getting started

You can learn more about Redux and how to use it in your web application by watching this [free course and tutorial](https://egghead.io/courses/fundamentals-of-redux-course-from-dan-abramov-bd5cc867){ target=_blank }, made by Dan Abramov himself, the creator of React-Redux.

Ideally, with this tutorial, you will be able to:

* Learn how to add proper state management to your React app
* Dive through reducers and how to manipulate state changes
* Learn to propagate the changes to the components
* Build a "to-do" list app using Redux
