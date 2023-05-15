# Mongoose - Populate

- [About Populate](#about-populate)
- [Populate 1 Field](#populating-a-field)
- [Populate Multiple Fields](#populating-multiple-fields)
- [Multi-level populate](#multi-level-populate)
- [Virtual Populate](#virtual-populate)
- [Virtual in Json & Object Response](#displaying-our-virtual-population-in-json-and-console-log-responses)

## About Populate

The `populate()` method that comes with mongoose is similar to the `$lookup` aggregation operator offered by Mongo, but more powerful.

Populate allows us to automatically replace specified paths in our document with documents from another collection.

## Populating a Field

```js
import {Schema, model} from 'mongoose';

const reviewSchema = new Schema({
  createdAt: {
    type: Date,
    default: Date.now(),
  },
  body: {
    type: String,
    required: [true, 'A review must have review content'],
  },
  rating: {
    type: Number,
    required: [true, 'A review must have a rating'],
    min: 1,
    max: 5,
  },
  user: {
    type: Schema.Types.ObjectId,
    ref: 'User',
  },
});

export const Review = model('Review', reviewSchema);
```

Here we have a basic schema for reviews. It generates the creation date on save, and accepts the review content and a rating. It also records the User ID of the user that created the review. A review ID will automatically be generated when a review is created.

When we want to display a review, after searching for it by ID, by default we will receive:

```js
const review = await Review.findById(reviewId);

// returns
review = {
  createdAt: aDateString,
  body: 'Great product, recommend!',
  rating: 4,
  user: ObjectId(dsa90usda0das0dna0sd), // not a real object ID
};
```

But we may wish to display the name of the user that left the review. This is where populate comes in. We can use populate to look up the user ID and get the user details at the time we locate the review, by chaining the populate method onto the query:

```js
const review = await Review.findById(reviewId).populate('user');
```

Now our output would be:

```js
review = {
  _id: asdbas98dha0sdna0sd,
  createdAt: aDateString,
  body: 'Great product, recommend!',
  rating: 4,
  user: {
    _id: d0s9ahd08n0na0sidna,
    username: 'Best Reviewer',
    lastLogin: aDateString,
  },
};
```

## Populating Multiple Fields

But what if our review model also records the ID of the product the review is about?

```js
product: {
  type: Schema.Types.ObjectId,
  ref: 'Product'
},
```

In this case, we may want to display both the user and the product. To do this, we simply chain populates:

```js
const review = await Review.findById(reviewId).populate('user').populate('product');
```

## Multi-level populate

So we've got the user, and we've got the product. But what if the product references the seller:

```js
const productSchema = new Schema({
  seller: {
    type: Schema.Types.ObjectId,
    ref: 'Seller',
  },
  // more fields
});
```

Perhaps when we display the review, we want to provide a link to the seller of the product as well as the review author and the product. Now we will need to use multi-level population. To do this, in the populate used for the product, we pass in an object instead of just the name of the field to populate. We still define the field to populate in this object, but under the key `path`. We can then provide a key of `populate` which will reference the field within that document we also want to populate:

```js
.populate({
  path: 'product',
  // populate the seller of the product
  populate: { path: 'seller' },
})
```

## Virtual Populate

The above examples adhere to the `Principle of Least Cardinality`. A product could have an indefinite amount of reviews, and a seller may sell thousands of products. So user documents would become very large, if they store a reference to their reviews, and a seller document would become very large if they store a reference to every product.

It is best practice to store the reference in the Many.

```js
import {Schema, model} from 'mongoose';

const userSchema = new Schema({
  username: {
    type: String,
    required: [true, 'You must provide a username'],
  },
});

const reviewSchema = new Schema({
  createdAt: {
    type: Date,
    default: Date.now(),
  },
  body: {
    type: String,
    required: [true, 'A review must have review content'],
  },
  rating: {
    type: Number,
    required: [true, 'A review must have a rating'],
    min: 1,
    max: 5,
  },
  user: {
    type: Schema.Types.ObjectId,
    ref: 'User',
  },
  product: {
    type: Schema.Types.ObjectId,
    ref: 'Product',
  },
});

export const User = model('User', userSchema);
export const Review = model('Review', reviewSchema);
```

With the above example, we can populate users on the reviews, but we can't use populate to get reviews for a user. For this we use `virtual populate`.

We do this by adding a `virtual` method to the `userSchema`. This adds a virual field to the model. The virtual requires that we pass in a name for the virtual field, and then an object that provides the reference to the review Model, the field within the user model that is stored in the foreign document (review document), and the field in the review document that stores this information:

```js
userSchema.virtual('reviews', {
  ref: 'Review',
  localField: '_id',
  foreignField: 'user',
  // if we only want the total number of reviews:
  count: true,
});

const User = model.('User', userSchema);
```

In this example we tell the virtual to create a virtual field called '`reviews`' on the User model. We then tell it to look in the '`Review`' collection and find any document where '`User._id`' is stored under the field '`user'`.

We can now use populate to get the reviews posted by the a user:

```js
const user = User.findById(id).populate('reviews');
```

We can also use projections, using `select`. A projection allows us to choose which fields we want to include when populating. Say we don't want to get the entire review object when we populate the virtual property 'reviews' for our user, instead we only want to display the ratings:

```js
const user = User.findById(id).populate({
  path: 'reviews',
  select: 'ratings user',
});
```

This will only work if we include the foreign field `user` in the projection select.

## Displaying our virtual population in Json and console log responses

By default, our virtual will not be populated and displayed when responding in json, or consol logs. TO make thi work we have to add some additional settings as a second arguament when creating the scheme:

```js
import {Schema, model} from 'mongoose';

const userSchema = new Schema(
  {
    username: {
      type: String,
      required: [true, 'You must provide a username'],
    },
  },
  {
    toJSON: {virtuals: true},
    toObject: {virtuals: true},
  }
);
```
