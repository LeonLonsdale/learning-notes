# Endpoint Protection

## Create Endpoint Protection Middleware (API)

Used to prevent access to certain endpoints based on login status.

This will throw errors if:

- No token is found.
- The token is not valid
- The user no longer exists
- The user changed password after the token was issued

Otherwise it adds the user object to the response, making it available to the next middleware.

```js
export const protect = catchAsync(async (req, res, next) => {
  let token;
  const auth = req.headers.authorisation;

  if (auth && auth.startsWith('Bearer')) {
    token = auth.split(' ')[1];
  } else if (req.cookies.jwt) {
    token = req.cookies.jwt;
  }

  if (!token) return next(new AppError('You are not logged in', 401));

  // validate the token
  const decoded = verifyToken(token);

  // check if the user still has an account
  const user = await User.findById(decoded.id);
  if (!user) return next(new AppError('User account no longer exists', 401));

  // check if the user password changed since the token was issued
  if (user.changedPasswordAfter(decoded.iat))
    return next(new AppError('Password has changed, login again', 401));

  // grant access
  req.user = user;
  next();
});
```

## Create Endpoint Protection Middleware (Web)

Some parts of a rendered webpage may be conditional on whether or not the user is logged in. We can use a similar middleware to the one above to provide some information to the template.

This middleware does a few things:

- Move on to the next middleware if
  - There is no cookie called JWT
  - The JWT verification is invalid
  - The user does not exist
  - The user password changed since the token was issued

By moving on to the next middleware, the user is not passed into locals, so is unavailable to views.

It will also move onto next if the token is malformed which wil be the case immediately after the user logs out (see [User Account Web](./user-account-web.md)).

```js
exports.isLoggedIn = async (req, res, next) => {
  if (req.cookies.jwt) {
    try {
      const token = req.cookies.jwt;
      // 1) Validate cookie
      const decoded = await verifyToken(token);
      // 2) Check if the user still exists
      const user = await User.findById(decoded.id);
      if (!user) return next();
      // 3) Check if the user changed password since the token was issued
      if (user.changedPasswordAfter(decoded.iat)) return next();
      // 4) grant access
      res.locals.user = user; // make the user available to views
      return next();
    } catch (err) {
      return next();
    }
  }
  next();
};
```

`res.locals` is available to the views.
