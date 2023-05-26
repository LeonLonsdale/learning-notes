# Geospatial

# Geospatial Queries - geoWithin

We are able to query documents to find out if they are within a specified distance of a specified point. This requires that a geospatial point is defined in the Schema. A geospatial point can be defined by applying a type of `point`, along with coordinates in the format of `[longitude, latitude]`:

```js
startLocation: {
  type: {
    type: String,
    default: 'Point',
    enum: ['Point'],
  },
  coordinates: [Number],
}
```

In our controller, we need to calculate a `radius` which is our distance divided by the radius of the earth. The radius of the earth:

- In miles is 3963.2
- In kilometers is 6378.1

```js
const radius = unit === 'miles' ? distance / 3963.2 : distance / 6378.1;
```

We can then submit a query string to our `Model.find()` query, using `$geoWithin`. We need to pass in `$centreSphere` into this, which must be an array `[[coordinates], radius]`. For geoJSON, in the `[coordinates]` the format reverses to `longitude, latitude`:

```js
const locations = await Model.find({
  startLocation: {$geoWithin: {$centerSphere: [[longitude, latitude], radius]}},
});
```

If we use a route like this:

```
/get-within/:distance/center/:latlng/unit/:unit
```

The controller should look something like this:

```js
getWithin = async (req, res, next) => {
  const {distance, latlng, unit} = req.params;
  const [lat, lng] = lnglat.split(',');
  const radius = unit === 'mi' ? distance / 3963.2 : distance / 6378.1;

  if (!lng || !lat) return next(new AppError('Please provide a longitude and latitude', 400));
  // if (!lng || !lat) throw new Error('Please provide a longitude and latitude');

  const locations = Model.find({
    startLocation: {
      $geoWithin: {
        $centerSphere: [[lng, lat], radius],
      },
    },
  });

  res.status(200).json({
    status: 'success',
    data: {
      data: locations,
    },
  });
};
```

However, we also need to apply indexing to our Schema to index our startLocations according to a 2dsphere.

```js
Schema.index({startLocation: '2dsphere'});
```

# Geospatial Aggregation

Aggregate pipelines give us a feature that can be used to work out distances from one point to another. This is `$geoNear`. This must be used as the first stage of a pipeline or it will not work - that includes any middleware that executes an aggregate pipeline, that will execute before reaching the aggregate using `geoNear`.

The first property that should be entered into $geoNear is `near` which should be in a geoJSON format. It should be an object containing the type of Point and the coordinates. `near` refers to the start point. We can also pass other fields into $geoNear such as the name of the field we want to display in queries, and a multiplier that should be used to convert the distance returned, which will be in `metres` by default.

For $geoNear to work, the schema must have a geospatial index - as above.

```js
const multiplier = unit === 'mi' ? 0.000621371 : 0.001;

const distances = await Model.aggregate([
  {
    $geoNear: {
      near: {
        type: 'Point',
        coordinates: [lng * 1, lat * 1],
      },
      distanceField: 'distance',
      multiplier: multiplier,
    },
    // limit the fields in the response to:
    {
      $project: {
        distance: 1,
        name: 1,
      }
    }
  }
]);
```

Using a route such as:

```
/distances/:latlng/unit/:unit
```

The controller would look something like this:

```js
getDistances = async (req, res, next) => {
  const { latlng, unit } = req.params;
  const [lat, lng] = latlng.split(',');
  const multiplier = unit === 'mi' ? 0.000621371 : 0.001;

  if (!lat || !lng) next(new AppError('Please provide a latitude and longitude in the format lat,lng', 400));

  const distances = await Model.aggregate([
    {
      $geoNear: {
        near: {
          type: 'Point',
          coordinates: [lng * 1, lat * 1],
        },
        distanceField: 'distance',
        multiplier: multiplier,
      },
      {
        $project: {
          name: 1,
          distance: 1
        },
      },
    },
  ]);

  res.status(200).json({
    status: 'success',
    data: {
      data: distances,
    },
  });
}
```
