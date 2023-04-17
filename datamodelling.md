# Data Modelling

# Definitions in this section

- `Schema`
- `Model`
- `Validation`
- `Referencing (Normalised)`
- `Embedding (Demormalised)`

# Contents

- [Types of data relationships](#types-of-data-relationships)
- [Referencing (Normalised) vs Embedding (Denomralised)](#referencing-normalised-vs-embedding-denormalised)
- [Deciding whether to reference or embed](#deciding-whether-to-reference-or-embed)
- [Types of referencing](#types-of-referencing)
- [Principles](#principles)
- [Creating Schemas with Mongoose](#creating-schemas-with-mongoose)
- [Schemas and Validation](#schemas-and-validation)
- [Virtual Properties](#virtual-properties)
- [Withholding Fields from Responses](#witholding-fields-from-results-directly-from-schema)
- [Referencing / Embedding in a Schema](#referencing--embedding-in-a-schema)
- [Populate](#populate)
- [Virtual Populate](#virtual-populate)
- [Calculating & Persisting Data from related collection](#calculating-and-persisting-statistics-of-related-collections)
- [Models](#models)

# Types of data relationships

- one to one
- one to many
  - one to few
  - one to many
  - one to ton
- many to many

## One to one

A movie will have 1 name.

## One to Many

- A movie may win a FEW awards
- A movie may have MANY actors
- A movie in a database may have one to a TON of logs

## Many to Many

Many actors may be in many movies, which have many actors.

# Referencing (Normalised) vs Embedding (Denormalised)

### Referenced Database

- Data is separated, with references to related objects.
- Known as 'Child Referencing' as the parent references its children.

```json
{
    "_id": ObjectId('123456),
    "title": "Interstellar",
    "releaseYear": 2014,
    "actors": [
        ObjectId('323423'),
        ObjectId('384957')
    ]
}
```

### Embedded Database

Related data is embedded within the same document.

```json
{
    "_id": ObjectId('123456),
    "title": "Interstellar",
    "releaseYear": 2014,
    "actors": [
        {
            "name": "Matthew McConaughey",
            "age": 50,
            "born": "Uvalde, USA",
        },
        {
            "name": "Anne Hathaway",
            "age": 37,
            "born": "NYC, USA",
        }
    ]
}
```

# Deciding whether to reference or embed

Consider:

- The relationship type.
  - One to Few, One to Many best suit embedding.
  - One to Many, One to Ton, Many to Many best suit referencing.
- How it will be accessed.
  - Embed if: data is not often updated, mostly read, rarely changed.
  - Reference if: data is updated a lot, and often changes.
- How close the data is.
  - Embed: If the data can only exist together
  - Reference: If we need to query each data separately.

# Types of Referencing

## Child Referencing

The parent document references its children. Best suited for One to Many.
One to Ton can result in the array containing references becoming too big.

```json
// parent
{
    "_id": ObjectId('123456),
    "app": "Movie Database",
    "errorLogs": [
        ObjectId('323423'),
        ObjectId('384957')
    ]
}

// children
{
    "_id": ObjectId('323423'),
    "type": "error",
    "timestamp": 132342342,
}
{
    "_id": ObjectId('384957'),
    "type": "error",
    "timestamp": 2342423424,
}
```

## Parent Referencing

The children reference the parent. The parent does not know about the children. The array cannot get too big this way.

```json
// parent
{
    "_id": ObjectId('123456),
    "app": "Movie Database",
}

// children
{
    "_id": ObjectId('323423'),
    "app": ObjectId('123456')
    "type": "error",
    "timestamp": 132342342,
}
{
    "_id": ObjectId('384957'),
    "app": ObjectId('123456'),
    "type": "error",
    "timestamp": 2342423424,
}
```

## 2 Way Referencing

Both parent and children reference each other.

```json
{
    "_id": ObjectId('123456),
    "title": "Interstellar",
    "releaseYear": 2014,
    "actors": [
        ObjectId('323423'),
        ObjectId('384957')
    ]
}

{
    "_id": ObjectId('323423'),
    "name": "Matthew McConaughey",
    "age": 50,
    "born": "Uvalde, USA",
    "movies": [ObjectId('123456), ObjectId(...)],
}

{
    "_id": ObjectId('384957'),
    "name": "Anne Hathaway",
    "age": 37,
    "born": "NYC, USA",
    "movies": [ObjectId('123456), ObjectId(...)],
}

```

# Principles

- Structure data to match the ways the application needs to query and update data.
- Always favour embedding unless there's a good reason not to, for one to few or one to many relationships.
- Always favour referencing when data is updated a lot and each piece of data needs to be accessisble on its own.
- Use embedding when the data is only read, rarely updated and the items belong intrinsically together.
- Don't allow arrays to grow indefinitely.
- Use two-way referencing for many to many relationships.

# Creating Schemas with Mongoose

A `Schema` is a means of designing your data model for a particular collection. You build the Schema first and then create the model from it. The model uses the rules you put into a scheme and applies it to your Mongo collections. As a standard feature of Schemas, you define the `types` that apply to each field.

```js
const movieSchema = new mongoose.Schema({
  name: String,
  releaseYear: Number,
  releaseDate: Date,
});
```

# Schemas and Validation

You can also apply some validation requirements to your Schema:

```js
const movieSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Movie title is required'],
  },
  releaseYear: Number,
  releaseDate: Date,
});
```

## Available only on strings

- `maxlength: [x, 'error message']` will give an error if the length of the data entered into this field exceeds `x`.
- `minlength: [x, 'error message']` will give an error if the length of the data entered into this field is less than`x`.
- `enum: { values: ['option1','option2','option3], message: 'error message'` will give an error if the data is not one of the options.

## Available on numbers and dates

- `max: [x, 'error message']` will give an error if data entered into this field exceeds `x`.
- `min: [x, 'error message']` will give an error if data entered into this field is less than`x`.

## Available on All

- `trim: true` removes whitespace before and after the entry.

## Custom Validators

A custom validator must return `true` or `false`. If it returns false, it should return an error, and if it returns true, it should be accepted.

We do this within the schema, within the schema type options using a `validate` property with a `callback function`.

```js
const newSchema = new mongoose.Schema({
	priceDiscount: {
		type: Number;
		validate: {
			validator: function(val) {
				return val < this.price;
			},
			message: 'Discount price must be less than the standard price',
		},
	},
});
```

When creating a custom validator, the `this` keyword within the clalback is only available when creating a new document, and will not work when updating a document.

There is a package called `validator` available on npm that provides a number of validators - only for strings. Many of the validations exist inherently in mongoose, but some are useful, such as the `isAlpha` method to check if all characters in a string are letters.

This can be installed via npm:

```bash
npm i validator
```

and included in your app:

```js
const validator = require('validator');
```

You can then use its methods within the Schema:

```js
const newSchema = new mongoose.Schema({
  name: {
    type: String,
    minlength: [3, 'Name must have at least 3 characters'],
    maxlength: [15, 'Name cannot have more than 15 characters'],
    trim: true,
    unique: true,
    validate: [validator.isAlpha, 'Name can only contain letters of the alphabet'],
  },
});
```

# Virtual Properties

Virtual properties are things that we do not want to save from the database - they can be derrived through calculations based on the data in the database. For example, converting kilometers to miles.

They cannot be used in a query.

```js
// we use a regular function here as we will be referring to the document properties
// this means that we need access to 'this'
userSchema.virtual('age').get(function () {
  // some code
  return result;
});

// or, for example:

userSchema.virtual('fullName').get(function () {
  return `${this.firstName} ${this.surname}`;
});
```

Assuming we have the date of birth in the userSchema, we can calculate the age based on the current date.

In order to include virtual properties in responses, we need to provide options in our Schema:

```js
const newSchema = new mongoose.Schema({}, {options});
```

The option we need to enter is

```js
toJSON: { virtuals: true },  // include virtuals when outputting as JSON
toObject: { virtuals: true }, // include virtuals when outputting as an Object
```

Without these options, we wont see virtuals in outputs.

# Witholding Fields from results directly from Schema

This is useful for hiding sensitive data that may be present in the database, such as passwords.

We can do this by including `select: false` in the schema:

```js
const userSchema = new mongoose.Schema({
  password: {
    type: String,
    select: false,
  },
});
```

# Referencing / Embedding in a Schema

## Referencing - by ID

```js
const movieSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Movie name is required'],
  },
  actors: [
    {
      type: mongoose.Schema.ObjectId,
      ref: 'Actor',
    },
  ],
});
```

## Directly embedding sub documents

```js
const schema = new mongoose.Schema({
  field: [
    {
      type: String,
      property: setting,
    },
  ],
});
```

Each entry in this case gets its own ObjectId in like a sub-document.

## Embedding - from another collection, using Id

```js
const schema = new mongoose.Schema({
  field: Array,
});

Schema.pre('save', async function (next) {
  const promises = this.field.map(async (id) => await Model.findById(id));
  this.field = await Promise.all(promises);
  next();
});
```

This method uses a document middleware to grab the documents from the other collection, using the ID, and then embeds them.

This can be problematic if there's a chance the original records in the other collection can be changed, as they will not automatically be changed in these embeds.

# Populate

When referencing by ID, we can use mongoose's `.populate()` to populate the information about each reference, without persisting that information to the parent.

Using the movie referencing actors example above:

```js
// in controller function.
const movie = await Movies.findById(req.params.id).populate('actors');

// can also choose which actor fields you don't want to include.
const movie = await Movies.findById(req.params.id).populate({path: 'actors', select: '-__v ...'});

// can also be done with query middleware
// this particular example will run for all queries involving find*
// not a good idea for many to many+
movieSchema.pre(/^find/, function (next) {
  this.populate({path: 'actors', select: '-__v ...'});
});
```

# Virtual Populate

With parent referencing, the parent does not know about the child.

Use mongoose feature `virtual populate' to access all the children of a parent without persisting the link to the database.

NOTE `By default, any virtual created in this way that has a nullish value will not be displayed. They will all be nullish until populate is called, so if the field is missing from responses, this is why`

On the parent schema:

```js
Schema.virtual('virtualField', {
  ref: 'Model',
  foreignField: 'reference to field in other model where current model is stored.',
  localField: 'reference to field in current model that is stored in the foreignField',
});
```

What if our Movies do not reference the actors - only the actors reference the movie:

```json
{
    "_id": ObjectId('123456),
    "title": "Interstellar",
    "releaseYear": 2014,
}

{
    "_id": ObjectId('323423'),
    "name": "Matthew McConaughey",
    "age": 50,
    "born": "Uvalde, USA",
    "movies": [ObjectId('123456), ObjectId(...)],
}

{
    "_id": ObjectId('384957'),
    "name": "Anne Hathaway",
    "age": 37,
    "born": "NYC, USA",
    "movies": [ObjectId('123456), ObjectId(...)],
}
```

```js
// in Movie Model

movieSchema.virtual('actors', {
  ref: 'Actor',
  foreignField: 'movies',
  localField: '_id',
});
```

So, when we query a movie and want to see the actors, this virtual populate will:

- Create a virtual field called `actors` which is not persisted to the database.
- Look into the `movies` field for each actor in the `Actor` collection and check for an id that matches the movie `_id`.

Now, with the virtual field in place, we can populate as normal. See [Populate](#populate).

Can cause a populate chain if the item being populated also populates it's own fields. Choose the right method to populate for this.

# Calculating and Persisting statistics of related collections.

If our API relates to movies, where users can leave a review, we will have a `one to many` relationship - potentially `one to ton`. We are likely, therefore, to use parent referencing. However, our movie Schema may still need to store information relating to how many reviews have been left, and what the average rating is.

To achieve this, we need to write some code in the `Review Schema` so that when a review is posted, the statistics are recalculated and persisted to the tour document:

- A static function - which will use aggregate pipelines to calculate the stats.
- A post-save middleware to call that static function after the review has been saved.

```js
// static function

reviewSchema.static.calcReviewStats = async function (movieId) {
  const stats = await this.aggregate([
    {
      $match: {tour: movieId}, // match reviews with the movieId in the movie field
    },
    {
      $group: {
        _id: '$tour', // group by movie field
        numRating: {$sum: 1}, // add 1 for each review that matches
        avgRating: {$avg: '$rating'}, // get the average of the matched ratings
      },
    },
  ]);
  // persist to the Movie document
  // the aggregate promise resolves to an array of objects.
  // typically 1 object per grouping, in this case only ever 1 group.
  await Movie.findByIdAndUpdate(movieId, {
    numRating: stats[0].numRating,
    avgRating: stats[0].avgRating,
  });
};
```

```js
// post-save middleware

reviewSchema.post('save', function () {
  // this.constructor gives access to methods on the model.
  // used because the model should be created at the end of the file.
  this.constructor.calcReviewStats(this.movie);
});
```

# Models

## Creating a Model from a Schema

Once the schema is ready, we can create a model from it. As a standard the name of the model is capitalised.

```js
const User = mongoose.model('User', userSchema);
```

We can then create an instance of thise model:

```js
const exampleUser = new User({
  name: 'Leon',
  email: 'some@email.com',
  alias: '',
  password: 'hashedstuff',
  posts: 5,
  createdAt: {
    type: Date,
    default: Date.now,
  },
});
```

As an instance of the model, `exampleUser` gains access to some methods availabe on models. So we can save this to our database using the method `.save()`.

```js
exampleUser.save();
```

This also returns a promise that we can consume:

```js
exampleUser
  .save()
  .then((doc) => console.log(doc))
  .catch((err) => console.log(err.message));
```

However a better way of doing this is using `<model>.create()`. As this returns a promise, it enables us to use `async/await` when creating our controllers:

```js
const newUser = User.create(data);
```

However, there may be some instances where we do not want to save everything. For example, we would not want a user to be able to pass in data that suggests they have the admin role. For this, we can specify which fields from the request body to save:

```js
const newUser = await User.create({
  name: req.body.name,
  email: req.body.email,
  password: req.body.password,
  passwordConfirmation: req.body.passwordConfirmation,
});
```

This way, a user will only be created using the specified fields above, even if they pass in:

```json
{
  "name": "some name",
  "email": "email@email.com",
  "password": "pass1234",
  "passwordConfirmation": "pass1234",
  "role": "admin"
}
```
