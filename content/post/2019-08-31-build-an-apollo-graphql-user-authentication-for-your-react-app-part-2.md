---
title: "Build an Apollo Graphql user authentication for your React app - part 2"
slug: build-an-apollo-graphql-user-authentication-for-your-react-app-part-2
date: 2019-08-29T19:34:07+02:00
categories:
 - JavaScript development
tags:
 - apollo
 - graphql
 - authentication
 - react
 - json web token
 - directive
 - authorization
images:
 - /images/metal network.jpg
---

In my [last post](/2019/08/27/build-an-apollo-graphql-user-authentication-for-your-react-app-part-1) we built a Graphql API that handles user authentication and authorization. In particular we added a `loginUser` query that returns a JWT token. This token can be used to access restricted resources.
In this post I will show what the implementation looks like on Reacts side.
<!--more-->

Walking through this guide requires basic React knowledge. We will use [React Router](https://reacttraining.com/react-router/) for routing, [React Hooks](https://reactjs.org/docs/hooks-intro.html) for executing Graphql queries and [Matieral-UI](https://material-ui.com/) for the component framework.

## Files

Let's get started with an overview of the project files.

* `Apollo.js`: Apollo client configuration
* `HeaderLoginButton.js`: Links to login page and handles logout
* `Login.js`: Login page
* `Profile.js`: View for user profile
* `queries.js`: Exports all Graphql queries and mutations
* `Routes.js`: React router component
* `hooks.js`: Export React hooks

*2019-09-18 Edit: Added React hook file* 

So obviously we are not going to build an React app from scratch. This guide assumes you already have one and intent to add authentication and authorization functionality.

## Routes

`Routes` is a [higher-order component](https://reactjs.org/docs/higher-order-components.html) (HOC) that connect our view components with specific routes.

**Routes.js**

```js
import React from 'react'
import { Route } from 'react-router-dom'
import Home from './Home'
import Login from './Login'
import Profile from './Profile'

const Routes = () => (
  <>
    <Route exact path='/' component={Home} />
    <Route exact path='/login' component={Login} />
    <Route exact path='/profile' component={Profile} />
  </>
)

export default Routes
```

So if the user opens or gets redirected to `/login` the `Login` component will be shown.

## Login

So what does the `Login` component looks like?

**Login.js**

```js
import React from 'react'
import { useLazyQuery } from '@apollo/react-hooks'
import { Redirect } from 'react-router'
import Paper from '@material-ui/core/Paper'
import Typography from '@material-ui/core/Typography'
import TextField from '@material-ui/core/TextField'
import Button from '@material-ui/core/Button'
import { LOGIN_USER } from './queries'
import Loading from './Loading'
import Error from './Error'
import { useForm } from './hooks'

const Login = () => {

  // Use form state
  const { values, handleChange, handleSubmit } = useForm((credentials) => loginUser(), {
    email: '',
    password: ''
  })

  // Lazy query for login user method
  const [loginUser, { called, loading, data, error }] = useLazyQuery(LOGIN_USER, { variables: values })

  // Wait for lazy query
  if (called && loading) return <Loading />

  // Show error message if lazy query fails
  if (error) return <Error message={error.message} />

  // Store token if login is successful
  if (data) {
    window.localStorage.setItem('token', data.loginUser.token)

    // Redirect to home page
    return <Redirect to='/' />
  }

  return (
    <Paper className={classes.paper}>
      <Typography className={classes.title} variant='h3' component='h1'>
        Login
      </Typography>
      <form onSubmit={handleSubmit}
      >
        <TextField
          variant='outlined'
          margin='normal'
          required
          fullWidth
          id='email'
          name='email'
          label='Email Address'
          type='email'
          value={values.email}
          onChange={handleChange}
          autoFocus
        />
        <TextField
          variant='outlined'
          margin='normal'
          required
          fullWidth
          id='password'
          name='password'
          label='Password'
          type='password'
          value={values.password}
          onChange={handleChange}
        />
        <Button
          type='submit'
          fullWidth
          variant='contained'
          color='primary'
          className={classes.submit}
        >
          Sign in
        </Button>
      </form>
    </Paper>
  )
}

export default Login
```

The component presents a login form. When credentials are submitted a React hook is used to run the lazy Graphql query and retrieve the JWT token. If the login is successful the token is stored in the local storage of the browser.

The form state is managed by a React hook as well. See the `hooks.js` for the simple `useForm` React hook example that can be reused for other form.

As you can see the query is imported from the `queries.js` file. This is externalized because we want to share queries among component.

**queries.js**

```js
import gql from 'graphql-tag'

const GET_CURRENT_USER = gql`
{
    currentUser {
        firstname
        lastname
    }
}
`

const LOGIN_USER = gql`
query loginUser($email: String!, $password: String!) {
    loginUser(email: $email, password: $password) {
        token
    }
}
`

export {
  GET_CURRENT_USER,
  LOGIN_USER
}

```

And here is the `useForm` React hook.

**hooks.js**

```js
import { useState } from 'react'

const useForm = (callback, data) => {
  const [values, setValues] = useState(data)

  const handleChange = (event) => {
    event.persist()
    setValues(values => ({
      ...values,
      [event.target.name]: event.target.value
    }))
  }

  const handleSubmit = (event, onSubmit) => {
    event.preventDefault()
    callback(values)
  }

  return {
    handleChange,
    handleSubmit,
    values
  }
}

export { useForm }
```

*2019-09-18 Edit: Added useForm React hook* 

## Apollo

We saw that the token is being stored in the local storage. But how is passed to Apollo server? This is done in the Apollo client.

**Apollo.js**

```js
import ApolloClient from 'apollo-boost'
import React from 'react'
import PropTypes from 'prop-types'
import { ApolloProvider } from '@apollo/react-hooks'

// Initialize Apollo client
const client = new ApolloClient({
  uri: process.env.REACT_APP_APOLLO_URL || 'http://localhost:3000/api',
  request: async operation => {
    // Get JWT token from local storage
    const token = window.localStorage.getItem('token')

    // Pass token to headers
    operation.setContext({
      headers: {
        Authorization: token ? `Bearer ${token}` : ''
      }
    })
  }
})

// Define Apollo component
const Apollo = ({ children }) => (
  <ApolloProvider client={client}>
    {children}
  </ApolloProvider>
)

Apollo.propTypes = {
  children: PropTypes.object.isRequired
}

export default Apollo
``` 

This HOC initializes the Apollo client and connects to the API. For every request it check if a token is available and if so forwards it in the Bearer Authorization header.

## Logout

Assuming the user has logged in, we can run queries that requires authorization from every other component.

**Profile.js**

```js
import React from 'react'
import { useQuery } from '@apollo/react-hooks'
import Paper from '@material-ui/core/Paper'
import Typography from '@material-ui/core/Typography'
import { GET_CURRENT_USER } from './queries'
import Loading from './Loading'
import Error from './Error'

const Profile = () => {

  const { loading, error, data } = useQuery(GET_CURRENT_USER)

  if (loading) return <Loading />
  if (error) return <Error message={error.message} />

  return (
    <Paper>
      <Typography variant='h3' component='h1'>
            Profil
      </Typography>
      <Typography component='p'>
        {`${data.currentUser.firstname} ${data.currentUser.lastname}`}
      </Typography>
    </Paper>
  )
}

export default Profile
```

The `getCurrentUser` query requires an authentication. Data is only shown if the user has logged in.

Now we got to the final step. Usually you'll find a login/logout button on the top right of the nav bar. This button is called `HeaderLoginButton` in our case and handles the logout.

**HeaderLoginButton.js**

```js
import React from 'react'
import { useQuery } from '@apollo/react-hooks'
import { Link } from 'react-router-dom'
import Button from '@material-ui/core/Button'
import { GET_CURRENT_USER } from './queries'

const HeaderLoginButton = () => {

  const { client, data } = useQuery(GET_CURRENT_USER)

  // Reset Apollo and local store on logout
  const logout = () => {
    window.localStorage.clear()
    client.resetStore()
  }

  if (data && data.currentUser) {
    return (
      <Button onClick={logout} color='inherit'>Logout</Button>
    )
  }
  return (
    <Link to='/login'>
      <Button color='inherit'>Login</Button>
    </Link>
  )
}

export default HeaderLoginButton
```

If the user is not logged in the button if clicked will redirect to the login page. Once the user has logged in the button will logout the user if clicked. The logout process simply gets rid of the token in the local storage and resets the Apollo client. 

Here you can also observe the power React hooks. We don't have to pass the Apollo client object down the hierarchy. Simply retrieve it from the `useQuery` hook!

That's it! 🎉

## Next

In the next and final post we are going to improve our solution regarding security. Cliff hanger questions: What if the user copies the token and paste it after logout? How can we ensure that a token cannot be used indefinitely 😕?

Navigate to [part 1](/2019/08/27/build-an-apollo-graphql-user-authentication-for-your-react-app-part-1) or [part 3](2019/09/26/build-an-apollo-graphql-user-authentication-for-your-react-app-part-3).

## Additions

As already mentioned in my last post I focus on the modifications you have to make to the app in order get a user authentication. Proper implementation of course requires various other features. So here are some I can think of:

* Password reset component
* User profile page
* Proper error and success notification
* Redirect to error page if user is unauthorized
* User signup process

## Edits

2019-09-18: Added useForm React hook.