# Data Modelling

## Types of data relationships

- one to one
- one to many
  - one to few
  - one to many
  - one to ton
- many to many

### One to one

A movie will have 1 name.

### One to Many

- A movie may win a FEW awards
- A movie may have MANY actors
- A movie in a database may have one to a TON of logs

### Many to Many

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

# Mongoose Schemas

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

When a request is made to the route for the movie, within the controller, use `populate('field')` to retrieve the actors data from the Actor collection.

```js
// in controller function.
const movie = await Movies.findById(req.params.id).populate('actors');

// can also choose which actor fields you don't want to include.
const movie = await Movies.findById(req.params.id).populate({path: 'actors', select: '-__v ...'});

// can also be done with query middleware
movieSchema.pre(/^find/, function (next) {
  this.populate({path: 'actors', select: '-__v ...'});
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
