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

We do not want to store the image in the database, instead store it on our storage location and store a reference to the image in the database.

This works as a middleware, however, when using it in the route, we'd still need to provide some information, such as how many images to expect, and the name of the form field to expect it from.

This would look something like this:

```js
router.method('/endpoint', upload.single('photo'), moreMiddleWare);
```

This is for a single image. It can become messy when we expect multiple files from various sources.

We can therefore improve this by creating our own middleware:

```js
export const uploadUserPhoto = upload.single('photo');
```

We can now use `uploadUserPhoto` in the middleware stack.

## Setting how many images will be uploaded

### Single Image

```js
upload.single('<field>');
```

### Multiple Image Files

For multiple image file uploads we use `.array()`. Setting up the middleware in a similar way to above:

```js
export const movieImages = upload.array({'field name', maxCount});

// for example, the below allows the user to upload a maximum of 3 images
// and these images will be expected from the form field named 'images'

export const movieImages = upload.array({'images', 3});
```

### Multiple image sources with different image quantities

There's an additional method that can be used if the form contains multiple image upload fields. For example, when adding a movie, we may provide the option to upload a movie cover (1 image) as well as the opportunity to upload multiple images from the movie. For this, we use `.fields()`.

We pass an array of objects into this method which define the rules for each field:

```js
export const movieImages = upload.fields([
  {
    name: 'coverImage',
    maxCount: 1,
  },
  {
    name: 'images',
    maxCount: 5,
  },
]);
```

## Creating storage

One problem with the use of:

```js
const upload = multer({dest: 'public/img/users'});
```

Is that the file name is changed by multer and has no file extension. The image therefore cannot be rendered once uploaded. To remedy this, we can set up `multer Storage`:

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

We can use this middleware in the same way as normal:

```js
router.method('/endpoint', uploadUserPhoto, moreMiddleWare);
```

The middleware will put some information about the file onto the request object. This can be found under `req.file` or for multiple images, `req.files`.

## Persisting the filename to the database

Within the middleware used to store the data to the database, add the following line:

```js
// dataObject in this example, is a the result of a utility function for stripping objects
if (req.file) dataObject.property = req.file.filename;
```

For example, if the middleware we are using uses a function to filter objects for updates:

```js
const filteredBody = filterObject(req.body, ['name', 'email']);
if (req.file) filterBody.photo = req.file.filename;
```

See [filterObject](/utilFunctions.md#filter-objects).

This adds the photo name to the object which can then be passed in when using `findByIdAndUpdate(id, filterBody)`
