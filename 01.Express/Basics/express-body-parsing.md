# Body Parsing

## JSON

Express comes with a built in function for parsing body data.

Returns middleware that only parses JSON and only looks at requests where the Content-Type header matches the type option. This parser accepts any Unicode encoding of the body and supports automatic inflation of gzip and deflate encodings.

A new body object containing the parsed data is populated on the request object after the middleware (i.e. req.body), or an empty object ({}) if there was no body to parse, the Content-Type was not matched, or an error occurred.

View the [Express Documentation](https://expressjs.com/en/api.html#express.json)

Set this up in the app file.

```js
app.use(express.json());
```

## Raw

Express also comes with a function that parses incoming data into a buffer in raw format.

Returns middleware that parses all bodies as a Buffer and only looks at requests where the Content-Type header matches the type option. This parser accepts any Unicode encoding of the body and supports automatic inflation of gzip and deflate encodings.

A new body Buffer containing the parsed data is populated on the request object after the middleware (i.e. req.body), or an empty object ({}) if there was no body to parse, the Content-Type was not matched, or an error occurred.

View the [Express Documentation](https://expressjs.com/en/api.html#express.raw)

Set this up in the app file.

```js
app.use(express.raw());
```
