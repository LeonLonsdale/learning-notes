# Rate Limiting

View [package documentation](https://www.npmjs.com/package/express-rate-limit).

## Installation

```
npm i express-rate-limiter
```

## Installation

```js
import rateLimit from 'express-rate-limit';
```

## Usage

All requests for an API:

```js
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per `window` (here, per 15 minutes)
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false, // Disable the `X-RateLimit-*` headers
});

// Apply the rate limiting middleware to all requests
app.use(limiter);
```

To use it in a 'regular' web server (e.g. anything that uses express.static()), where the rate-limiter should only apply to certain requests:

```js
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per `window` (here, per 15 minutes)
  standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
  legacyHeaders: false, // Disable the `X-RateLimit-*` headers
});

// Apply the rate limiting middleware to API calls only
app.use('/api', apiLimiter);
```
