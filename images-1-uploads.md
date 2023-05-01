# Multer

Multer is an npm package that allows handling of multi-part form data.

```
npm i multer
```

```js
import multer from 'multer';
```

## Creating middleware

This allows us to create a middleware:

```js
const upload = multer({options});
```

## Options

Without any options, the photo will be stored in memory rather than a specifid destination.

So, one option we can set is the destination in which we want our photo to be stored:

```js
const upload = multer({dest: 'public/img/users'});
```

We do not want to store the image in the database, instea store it on our storage location and store a reference to the image in the database.

However, there are better ways to do this.

## Creating storage

One problem with the use of:

```js
const upload = multer({dest: 'public/img/users'});
```

Is that the file name is changed and has no file extension. The image therefore cannot be rendered once uploaded. To remedy this, we can set up `multer Storage`:

```js
const multerStorage = multer.diskStorage({
  // set the storage location
  destination: (req, file, callback) => {
    callback(null, 'public/img/users');
  },
  // set the target file names
  filename: (req, file, callback) => {
    // mimetype will show : image/extension
    const ext = file.mimetype.split('/')[1];
    // create unique file name in storage
    callback(null, `user-${req.user.id}-${Date.now()}.${ext}`);
  },
});
```

However, if you plan to perform any image processing after this middleware, then we should store the image in memory instead:

```js
const multerStorage = multer.memoryStorage();
```

## Create a filter

The filters job is to make sure the uploaded file is an image:

```js
const multerFilter = (req, file, callback) => {
  if (file.mimitype.startsWith('image')) {
    callback(null, true);
  } else {
    callback(err, false);
  }
};
```

`err` can be replaced with any custom error handler, such as `new AppError('message', 400)`.

## Applying the Storage and Filter to upload

```js
const upload = multer({
  storage: multerStorage,
  fileFilter: multerFilter,
});
```

## Using the middleware

`upload` can be used as middleware as it is, but requires us to tell it how many files to expect, and the name of the form field it will find these files in:

```js
router.method('/endpoint', upload.single('photo'), moreMiddleWare);
```

But we can tidy this up further:

```js
export const uploadUserPhoto = upload.single('photo');
```

and use this instead:

```js
router.method('/endpoint', uploadUserPhoto, moreMiddleWare);
```

The middleware will put some information about the file onto the request object. This can be found under `req.file`.

## Persisting the filename to the database

Within the middleware used to store the data to the database, add the following line:

```js
if (req.file) dataObject.property = req.file.filename;
```

For example, if the middleware if we are using a function to filter objects for updates:

```js
const filteredBody = filterObject(req.body, ['name', 'email']);
if (req.file) filterBody.photo = req.file.filename;
```

This adds the photo name to the object which can then be passed in when using `findByIdAndUpdate(id, filterBody)`
