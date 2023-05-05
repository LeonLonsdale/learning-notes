# Enforce HTTPS

## Install express-sslify

```
npm i express-sslify
```

## Include into app file

```js
import {HTTPS} from 'express-sslify';
```

## Apply as first middleware

In the event that we use a service that proxy redirects, we will need to tell express to trust these proxy headers before we use this https enforcement:

```js
app.enable('trust proxy');
```

Note that it is important to only use this middleware when in production. If we fail to check for production, this will apply to `http` requests on localhost which the browser will cache. It then becomes a minor problem to fix, and uninstalling the package will not suffice.

```js
if (process.env.NODE_ENV === 'production') app.use(HTTPS());
```
