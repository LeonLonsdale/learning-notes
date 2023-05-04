# Pre-setup

## Install Node

### With Homebrew

```
brew install node
```

### Download

[Visit the official node website](https://nodejs.org/en)

## Initiate npm

```
npm init -y
```

## Install Nodemon Globally

```
npm i -g nodemon
```

## Create npm nodemon start script

In `package.json`:

```json
"scripts": {
  "start": "nodemon server.js"
}
```

## Install Express

```
npm i express
```

## Install TypeScript and types (if necessary)

```
npm i -D typescript @types/node @types/express
```

## Create app and server files

```bash
touch app.js server.js
# OR
touch app.ts server.ts
```

# Express Setup in app.js/ts

## Include express

### ECMA

```js
import express from 'express';
```

### CommonJS

```js
const express = require('express');
```

### TypeScript

```ts
import express, {Express, Request, Response} from 'express';
```

## Initiation

### ECMA

```js
export const app = express();
```

### CommonJS

```js
const app = express();
// ...
module.exports = app;
```

### TypeScript

```ts
export const app: Express = express();
```

# Express Server Connection in server.js/ts

## Import app

### ECMA

```js
import {app} from './app';
```

### CommonJS

```js
const app = require('./app.js');
```

### TypeScript

```ts
import {app} from './app.js';
```

## Setup Port Variable

### ECMA & CommonJS

```js
const port = process.env.PORT || <portNum>;
```

### TypeScript

```js
const port: Number = Number(process.env.PORT) || <portNum>;
```

Note: process.env.PORT will not work yet.

## Start Server

### ECMA, CommonJS, and TypeScript

```js
const server = app.listen(port, () => console.log(`Server running on http://localhost:${port}`));
```

# Create Test Route in app.js/ts

## ECMA

```js
app.get('/', (req, res) => res.send('Server is working'));
```

## CommonJS

```js
// express import

app.get('/', (req, res) => res.send('Server is working'));

// start server
```

## TypeScript

```ts
app.get('/', (req: Request, res: Response) => res.send('Server is working'));
```

# Run Server to Test

```
npm start
```

Open `http://localhost:<port>` in the browser.
