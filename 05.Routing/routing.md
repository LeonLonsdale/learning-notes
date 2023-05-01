# Routing

# Contents

- [Nested Routes](#nested-routes)

# Nested Routes

## Basic

Example nexted route:

```
/movies/:movieId/reviews/reviewId
```

Build the route in the route file for the main route. For example, in the moviesRoute:

```js
// remember that url/movies has been mounted.
router.route('/:movieId/reviews/:reviewId').get(reviewsController.getReview);
```

The controller function is in the controller of the nexted route. For example, in the reviewsController.

## With Express using mounting

The basic method can result in duplicate code where you can, for example:

- Get the review with the `/movies/:movieId/reviews/:reviewId` route; AND
- Get the review directly with a `/reviews/:id` route.

To overcome this, we want to `mount` the reviewRouter into the movie router. So need to import the reviewRouter into movieRouter file.

Mount it:

```js
router.use('/:movieId/reviews', reviewRouter);
```

Now, in the reviewsRouter file, where we have declared our router, we use the express feature `mergeParams`

```js
const router = express.Router({mergeParams: true});
```

This gives the reviews router access to the parameters of the movie router its mounted on.
