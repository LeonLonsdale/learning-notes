# CORS

Cross-Origin Resource Sharing (CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.

[View on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

## CORS Package

```
npm i cors
```

## Usage

Import into app document:

```js
import cors from 'cors';
```

Apply CORS to all endpoints:

```js
app.use(cors());
```

Apply CORS to a specific endpoint:

```js
app.get('/endpoint', cors(), moreMiddleware);
```

These work for `simple requests`. A simple request is a `get` or `post` request. `Non-simple` requests are `put`, `patch`, and `delete` requests.

For `non-simple` requests, the browser will perform an `options` request (another type of HTTP request) to make sure the original (non-simple) request is safe.

When we get an `option` request, we need to respond to it by sending back the same `access control allow origin` header so that the browser knows it's safe to perform the original request. Express provides for the options request into which we can pass cors.

```js
app.options('*', cors());
```
