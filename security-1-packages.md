# Security

# Packages and Libraries

## Base Packages

```
npm i bcrypt express-rate-limit helmet xss-clean express-mong-sanitize hpp
```

## Libraries

```
crypto, util.promisify
```

# Create token with node crypto library

## Import

```js
import crypto from crypto;
```

## Create functions to generate tokens

Generate Token.

```js
export const getToken = () => crypto.randomBytes(20).toString('hex');
```

Encrypt token.

```js
export const encryptToken = (token) => crypto.createHash('sha256').update(token).digest('hex');
```

# Apply a Rate Limiter

## Install Rate Limiter

```
npm i express-rate-limit
```

```js
import rateLimit from express-rate-limit;
```

## Create a Limiter

```js
const limiter = rateLimit({
  max: 100,
  windowMs: 60 * 60 * 1000, // the time limit in milliseconds
  message: 'Too many requests from this IP. Try again in 1 hour',
});
```

## Create the global middleware

```js
app.use('/', limiter);
```

# Apply HTTP Security Header

## Install Helmet

```
npm i helmet
```

```js
import helmet from helmet;
```

## Create the global middleware

```js
app.use(helmet());
```

# Data Sanitisation - noSQL Query Injection

## Install the express-mongo-sanitize package

```
npm i express-mongo-sanitize
```

```js
import mongoSanitise from express-mongo-sanitize;
```

## Add the global middleware

```js
app.use(mongoSanitise());
```

# Data Sanitisation - XSS

## Install the xss-clean package

```
npm i xss-clean
```

```js
import xss from xss-clean;
```

## Add the global middleware

```js
app.use(xss());
```

# Prevent Parameter Polution

## Install the hpp package

```
npm i hpp
```

```js
import hpp from hpp;
```

## Add the global middleware

```js
app.use(hpp());
```

## Whitelist parameters

White list parameters by passing in an object containing an array of parameters.

```js
app.use(hpp({['paramters','parameters']}));
```

Paremeters are added to a whitelist.

# JSON Web Tokens

JSON Web Tokens are ....

A JSON Web Token should be stored in a secure HTTP only cookie.

## Install JSONWebToken

```
npm i jsonwebtoken
```

## Create a JWT secret and expiry date in environment

- Secret shouuld ideally be at least 32 characters long.
- Can set expiry using 1s, 1m, 1h, 1d

```
JWT_SECRET=
JWT_EXPIRY=
JWT_COOKIE_EXPIRY=
```

## Create function to generate JSONWebToken storing user ID

```js
const getJWT = (id) => jwt.sign({id}, process.env.JWT_SECRET, {expiresIn: process.env.JWT_EXPIRY});
```

## Create function to verify tokens later

```js
const verifyJWT = async (token) => promisify(jwt.verify)(token, process.env.JWT_SECRET);
```

Note: `promisify` is available from node but needs to be imported first.

```js
import { promisify } from util;
```

See [Node Docs](https://nodejs.org/api/util.html#utilpromisifyoriginal) | [The util.promisify() function](https://masteringjs.io/tutorials/node/promisify)

## Create function to build cookie containing JWT

Cookie syntax:

```js
res.cookie = ('cookieName', dataToSend, cookieOptions);
```

- See [Express Docs - res.cookie (API v4x)](https://expressjs.com/en/4x/api.html#res.cookie) || [Express Docs - res.cookie (API v5x)](https://expressjs.com/en/5x/api.html#res.cookie)

```js
const sendJWT = (user, statusCode, res) => {
  const token = getToken(user._id);
  const cookieOptions = {
    expires: new Date(Date.now() + process.env.JWT_COOKIE_EXPIRY * 24 * 60 * 60 * 1000),
    httpOnly: true,
  };

  if (process.env.NODE_ENV === 'production') cookieOptions.secure = true;

  res.cookie('jwt', token, cookieOptions);

  user.password = undefined;

  res.status(statusCode).json({
    status: 'success',
    token,
    data: {
      user,
    },
  });
};
```

# Cookie Parsing

To parse the cookie we created, we install cookie-parser

```
npm i cookie-parser
```

an include it in our app.js file:

```js
import cookie-parser as cookieParser from 'cookie-parser';
```

Now we use it similar to our body parser:

```js
app.use(cookieParser());
```
