# Passport

## Passport Local

```
npm i passport passport-local
```

If using with Mongoose:

```
npm i passport-local-mongoose
```

## app setup

```js
import passport from 'passport';
import localStrategy from 'passport-local';
```

To set up in the app:

1. Initialise passport
2. Initialise passports session. This must come AFTER express session.
3. Create a new local strategy passing in User.authenticate
4. Tell passport how to serialise the user
5. Tell passport how to deserialise the user

The `user.authenticate` method is added to the user model by `passport-local-mongoose`

Serialising tells passport how to store data in the session.

```js
app.use(passport.initialize());
app.use(passport.session());
passport.use(new localStrategy(User.authenticate()));
passport.serializeUser(User.serializeUser());
passport.deserializeUser(User.deserializeUser());
```

## Create our user model

```js
import {Schema, model} from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';

const userSchema = new Schema({
  email: {
    type: String,
    required: true,
    unique: true,
  },
});

userSchema.plugin(passportLocalMongoose);

export const User = model('User', userSchema);
```

`passportLocalMongoose` will add fields to our model, as well as some methods.

In particular, it will add the following `fields`:

- Salt
- Hash

And the following `methods`:

- register()
- authenticate()

## User Registration

To register a user, we make use of the new `register()` method added to our user model by `passport-local-mongoose`. We do this in our `createUser` controller.

We do this by first creating the user, with our model, by passing in all submitted registration information _apart from_ the password.

```js
export const createUser = catchAsync(async (req, res) => {
  const {email, username, password} = req.body;
  const newUser = new User({email, username});
});
```

We then use the `regsiter()` method and pass in the `newUser` and the `password`. Behind the scenes, this register method will then create a salt and add that to our user, and hash the password using the salt, and add that hash to the user, then persist the user to the database.

```js
export const createUser = catchAsync(async (req, res) => {
  const {email, username, password} = req.body;
  const newUser = new User({email, username});
  const user = await User.register(newUser, password);
  res.redirect('/');
});
```

If we are using flash messages for notifications, we can wrap this in a try-catch and redirect back to the register page in case the user tries to register using duplicate details:

```js
export const createUser = async (req, res) => {
  try {
    const {email, username, password} = req.body;
    const newUser = new User({email, username});
    const user = await User.register(newUser, password);
    res.redirect('/');
  } catch (err) {
    req.flash('error', err.message);
    res.redirect(req.url);
  }
};
```

So, this creates the user account, but it doesn't log them in. To log the user in simultaneously, we can use a method added to `req` by passport: `login()`. This accepts the new `user` and a callback function to handle errors:

```js
export const createUser = async (req, res) => {
  try {
    const {email, username, password} = req.body;
    const newUser = new User({email, username});
    const user = await User.register(newUser, password);
    req.login(user, (err) => {
      if (err) return next(err);
    });
    res.redirect('/');
  } catch (err) {
    req.flash('error', err.message);
    res.redirect(req.url);
  }
};
```

## Logging userrs in

Passport comes with built in authentication found on `passport.authenticate()`. We set the authentication strategy earlier in the document when we set `passport.use(new localStrategy(User.authenticate()));` in the app file.

We use this as a middleware in our route:

```js
router.route('/route').post(passport.authenticate('local'));
```

We can then set some additional options, such as `failureFlash` to send a flash message automatically, or `failureRedirect` to redirect the user to another page if the authentication fails.

```js
router.route('/route').post(
  passport.autehnticate('local', {
    failureFlash: true,
    failureRedirect: '/url',
  })
);
```

By default, if authentication is successful, this will add the user to `req.user` and a login session is established. It calls next() so we can have additional middleware.

```js
const login = (req, res) => {
  req.flash('success', 'Successfully logged in');
  res.redirect('/home');
};

router.route('/route').post(
  passport.autehnticate('local', {
    failureFlash: true,
    failureRedirect: '/url',
  }),
  login
);
```

## Check if user is logged in (authorisation)

Passport adds an `isAuthenticated()` method to requests, accessed using `req.isAuthenticated()`. We can use this to create a middleware that we can then apply on any applicable routes:

```js
export const isLoggedIn = (req, res) => {
  if (!req.isAuthenticated()) {
    req.flash('error', 'You are not logged in');
    res.redirect('/login');
  }
  next();
};
```

## Logging users out

Passport also adds `logout()` to the `req` object. This accepts a callback function for error handling.

```js
export const logout = (req, res, next) => {
  req.logout((err) => {
    if (err) return next(err);
  });
  req.flash('succes', `You've been logged out, come back soon`);
  res.redirect('/home');
};
```

## Returning user to original page

When a user is not logged in when they attempt to access an area of the website restricted to logged in users, we redirect the user to the login page. It is a good user experience to send the user back to the page they originally tried to access, after they log in.

First, we need to store the original URL in the session. We do this in our `isLoggedIn` middleware by adding:

```js
req.session.returnTo = req.originalUrl;
```

Then we create a new middleware to store this into our locals, since passport.authenticate() will clear the session:

```js
export const storeReturnTo = (req, res, next) => {
  if (req.session.returnTo) res.locals.returnTo = req.session.returnTo;
  next();
};
```

Now we call this middleware before authenticating in our login route. We need to do this before passport.authenticate so that the url is stored in locals before passport clears the session.

```js
router.route('/login').post(storeReturnTo, passport.authenticate(...), login);
```

Now that we have the original url stored in locals, we can use this within our login controller to to redirect, by adding:

```js
const redirectUrl = res.locals.returnTo || '/home';
res.redirect(redirectUrl);
```
