# React Router

![](../../../assets/rails/react/react-router.webp){ width=200 align=right }

React Router is a standard library for routing in React applications. It enables the navigation among views of various components in a React application, allows changing the browser URL, and keeps the UI in sync with the URL. React Router provides a collection of navigational components like `<Router>`, `<Route>`, `<Link>`, and `<Switch>` to build single-page applications with dynamic routing.

Here's an example of how you can create and set up a router for your web application.

1. __Install `react-router-dom` package__:

    ```
    npm install react-router-dom
    ```

2. __Set up the router somewhere in your application code, as follows__:

    ```jsx
    import React from 'react';
    import ReactDOM from 'react-dom';
    import { RouterProvider, createBrowserRouter } from 'react-router-dom';

    const router = createBrowserRouter([
      {
        path: '/',
        element: <BaseAppLayout />,
        children: [
          { path: '/', element: <HomePage /> },
          { path: 'about', element: <AboutPage /> },
          { path: '*', element: <NotFoundPage /> },
        ],
      },
    ]);

    ReactDOM.render(
      <RouterProvider router={router} />,
      document.getElementById('root')
    );
    ```

3. __On your `BaseAppLayout` component, add an `Outlet`__:

    ```jsx
    import React from 'react';
    import { Outlet } from 'react-router-dom';

    function BaseAppLayout() {
      return (
        <div>
          <h1>My App</h1>
          <Outlet />
        </div>
      );
    }

    export default App;
    ```

    This `Outlet` will render all the nested routes in your app, depending on which one is active at a time, of course.

## More information

If you're interested in learning more about React Router, you can follow their [tutorial on creating an Address Book](https://reactrouter.com/tutorials/address-book){ target=_blank }.
