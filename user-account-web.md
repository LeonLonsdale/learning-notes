# HTTP Requests
For logging in and logging out on the web, using a `HTTP request`, we use a package called `axios`. Axios returns a promise, so we can use `async await` and will also throw errors received from the API:

```
npm i axios
```


# Logging In (Web)


```js
export const login = async (email, password) => {
  try {
    const result = await axios({
      method: 'POST',
      url: '{{URL}}/api/v1/users/login',
      data: {
        email,
        password,
      }
    });

    if (result.data.status === 'success') {
      // send a success alert
      window.setTimeout(() => location.assign('/'), 1500);
    }
  } catch err {
    // send a failure alert
  }
}
```

# Logging Out

The cookie that we send when logging in via our API login endpoint cannot be manipulated or deleted. 

The only option then is to overwrite it with a new cookie that does not contain a token. This would typically cause an error in the `isLoggedIn` middleware, however, the `try catch` block will pick this up and return next().

```js
export const logout = (req, res) => {
  // create new cookie with invalid token to expire after 10s
  res.cookie('jwt', 'loggedout', {
    expires: new Date(Date.now() * 10 * 1000),
    httpOnly: true,
  });

  res.status(200).json({
    status: 'success',
  });
};
```
