# Node App Setup

Using:

- Express server
- MongoDB
- Mongoose

# Packages and Libraries

## 1) Base Packages

```
npm i express mongoose dotenv nodemailer slugify
```

## 2) Security Packages

```
npm i bcrypt express-rate-limit helmet xss-clean express-mong-sanitize hpp
```

## 3) Libraries

```
crypto
```

# Definitions

- Instance Method - a method attatched to a mongoose schema that will be available on all instances of the model

# Send Emails with nodemailer.

- [Documentation](https://nodemailer.com)

## 1) Install with npm

```
npm i nodemailer
```

## 2) Setup email environment variables

Add your email host, port, username and password to `env` file.

```
EMAIL_HOST=
EMAIL_PORT=
EMAIL_USER=
EMAIL_PASS=
```

## 3) Setup sendEmail function

The transporter object method `sendMail` returns a promise. So use async function.

```js
export const sendEmail = async (options) => {};
```

Create configuration object.

```js
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };
};
```

A transporter is required to send emails. It only needs to be created once. Create the transporter.

```javascript
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };

  const transporter = nodemailer.createTransport(options);
};
```

Setup mail options.

```javascript
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };

  const transporter = nodemailer.createTransport(options);

  const mailOptions = {
    from: '<from email address>',
    to: options.email,
    subject: options.subject,
    text: options.text, // raw text format
    // html: options.html (used to send HTML instead of raw text)
  };
};
```

# Create token with node crypto library

## 1) Import

```js
import crypto from crypto;
```

## 2) Create functions to generate tokens

Generate Token.

```js
export const getToken = () => crypto.randomBytes(20).toString('hex');
```

Encrypt token.

```js
export const encryptToken = (token) => crypto.createHash('sha256').update(token).digest('hex');
```

# JSON Web Tokens

JSON Web Tokens are ....

A JSON Web Token should be stored in a secure HTTP only cookie.

## 1) Install JSONWebToken

```
npm i jsonwebtoken
```

## 2) Create a JWT secret and expiry date in environment

- Secret shouuld ideally be at least 32 characters long.
- Can set expiry using 1s, 1m, 1h, 1d

```
JWT_SECRET=
JWT_EXPIRY=
JWT_COOKIE_EXPIRY=
```

## 3) Create function to generate JSONWebToken storing user ID

```js
const getJWT = (id) => jwt.sign({id}, process.env.JWT_SECRET, {expiresIn: process.env.JWT_EXPIRY});
```

## 4) Create function to verify tokens later

```js
const verifyJWT = async (token) => promisify(jwt.verify)(token, process.env.JWT_SECRET);
```

Note: `promisify` is available from node but needs to be imported first.

```js
import { promisify } from util;
```

See [Node Docs](https://nodejs.org/api/util.html#utilpromisifyoriginal) | [The util.promisify() function](https://masteringjs.io/tutorials/node/promisify)

## 4) Create function to build cookie

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

# Apply a Rate Limiter

## 1) Install Rate Limiter

```
npm i express-rate-limit
```

```js
import rateLimit from express-rate-limit;
```

## 2) Create a Limiter

```js
const limiter = rateLimit({
  max: 100,
  windowMs: 60 * 60 * 1000, // the time limit in milliseconds
  message: 'Too many requests from this IP. Try again in 1 hour',
});
```

## 3) Create the global middleware

```js
app.use('/', limiter);
```

# Apply HTTP Security Header

## 1) Install Helmet

```
npm i helmet
```

```js
import helmet from helmet;
```

## 2) Create the global middleware

```js
app.use(helmet());
```

# Data Sanitisation - noSQL Query Injection

## 1) Install the express-mongo-sanitize package

```
npm i express-mongo-sanitize
```

```js
import mongoSanitise from express-mongo-sanitize;
```

## 2) Add the global middleware

```js
app.use(mongoSanitise());
```

# Data Sanitisation - XSS

## 1) Install the xss-clean package

```
npm i xss-clean
```

```js
import xss from xss-clean;
```

## 2) Add the global middleware

```js
app.use(xss());
```

# Prevent Parameter Polution

## 1) Install the hpp package

```
npm i hpp
```

```js
import xss from xss-clean;
```

## 2) Add the global middleware

```js
app.use(hpp());
```

# 3) Whitelist parameters

White list parameters by passing in an object containing an array of parameters.

```js
app.use(hpp({['paramters','parameters']}));
```
