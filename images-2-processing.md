# Image Processing

Requires a package named `Sharp`

```
npm i sharp
```

## Resizing

First we pass the image into `sharp`. If using multer, this image should be in the request under `req.file.buffer`:

```js
sharp(req.file.buffer);
```

This returns an object onto which we can chain methods.

The method for resizing is `resize` into which we can pass our required dimensions:

```js
sharp(req.file.buffer).resize(width, height);
```

## Converting to a File Type

Another method we can chain is `toFormat()` which converts the file type to a format we choose:

```js
sharp(req.file.buffer).resize(500, 500).toFormat('jpeg');
```

## Compressing

We will likely want to compress uploaded images for performance and disk management. We can do that with methods specific to your chosen filetype. We pass an object of options into this method. The `quality` option allows us to reduce image quality, and therefore compress the file:

```js
sharp(req.file.buffer).resize(500, 500).toFormat('jpeg').jpeg({quality: 90});
```

## Saving the output

Finally, we can use the `.toFile()` method to save the file. We pass in the full url of the save destination, along with the file name formatting we desire:

```js
// this line is important so that the filename is available
// in the final middleware that stores the data in the database
req.file.filename = `user-${req.user.id}-${Date.now()}.jpeg`;

sharp(req.file.buffer)
  .resize(500, 500)
  .toFormat('jpeg')
  .jpeg({quality: 90})
  .toFile(`your/location/for/images/${req.file.filename}`);
```

## Create the middleware

```js
export const resizeUserPhoto = (req, res, next) => {
  if (!req.file) return next();

  req.file.filename = `user-${req.user.id}-${Date.now()}.jpeg`;

  sharp(req.file.buffer)
    .resize(500, 500)
    .toFormat('jpeg')
    .jpeg({quality: 90})
    .toFile(`your/location/for/images/${req.file.filename}`);

  next();
};
```
