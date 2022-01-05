## Aggregation Framework

It is another way to query data in MongoDB that exceeds the capabilities of querying with MQL.

It is a pipeline that is done in the stages that they are provided.

Non-filtering stages do not modify the original data, instead they work with the data in the cursor.

Pipelines are always an array of one of more stages, stages are composed of one or more aggregation operators or expressions, expressions may take a single argument or array of arguments, this is expression dependant.

### Filtering Documents with `$match`

It should come as early as possible and you are free to use as many match stages as necessary.

Consider it to be more of a `filter` than a `find`, but uses the same query syntax as `find()`.

If it is the first stage of the pipeline it can take advantage of our indexes which will increase the performance of our aggregations.

A `$match` stage may contain an `$text` query operator but it must be the first stage in a pipeline.

You cannot use a `$where` in a `$match`.

### Shaping Documents with `$project`

You can use `$project` to selectively retain and remove fields, you can reassign existing field values and derive entirely new fields. It can be used as many times as required within a pipeline.

Where `$match` is like a `.filter()`, `$project` is like a `.map()`.

You can remove with a `0` and retain with a `1`, when you specify a value to retain you must then specify each value you wish to retain. The `_id` field is the only field that you must explicitly remove.

### Using `$addFields` alongside `$project`

It is very similar to `$project` with one key difference, it adds fields to a document. With `$project` we can selectively remove and retain fields, `$addFields` only allows you to modify the incoming pipeline documents with new computed fields or to modify existing fields.

It can be a bit tedious to use `$project` to constantly declare which fields you want to include.

### Working with GeoJSON

To perform geo queries within the aggregation pipeline you can use `$geoNear`. It must be the first stage within a pipeline and have only one geo index.

`$geoNear` can be used on sharded collections where `$near` cannot.

An example of using `$geoNear` could be:

```javascript
$geoNear: {
  near: { type: "Point", coordinates: [-67.89, 45.67] },
  distanceField: "distanceFromQuery",
  spherical: true
}
```

You can add optional arguments to refine the query:

- `minDistance` (in meters)
- `maxDistance` (in meters)
- `query` general query against documents

### Group Stage

The aggregation framework exposes the `$group` operator that takes the incoming stream of data, and siphons it into multiple distinct reservoirs.

The required parameter is the `_id` field which is the criteria the group stage uses to categorise and bundle documents together.

Grouping on just one expression is functionally equivalent to using the distinct command.

You can use all accumulator expressions within `$group`.

It can be used multiple times within a pipeline.

It may be necessary to sanitise incoming data to ensure missing or differently typed values do not mess up the calculations.

Each time group categorises a document the sum express gets called, we can use this to find out how many documents belong to each group. For each match, it will add the value we provide to `$sum`, for example:

```
{
  $group: {
    _id: '$year',
    count_per_year: { $sum: 1 }
  }
}
```

### Accumulator Stages

Accumulator expressions in `$project` operate over an array in the current document, they do not carry values over all documents, they have no memory between documents.

May still have to use `$reduce` or `$map` for more complex calculations.

- `$sum`
- `$avg`
- `$max`
- `$min`
- `$stdDevPop` (all documents)
- `$stdDevSamp` (sample of documents)

### Unwind Stage

The `$unwind` stage lists unwind in an array field, creating a new document for every entry where the field value is now each entry.

As an example, if you had a document with fields `one`, `two`, and an array field of `three`, after the unwind you would have documents with `one`, `two`, and `three` where `three` is a single field from the array with all of the original fields.

Using unwind on large collections and big documents can exceed memory usage, use matching and projection to get this down and only work on the data we need to, you can use the disk if required though.

### Cursor Methods

Some cursor methods are:

- `pretty()`
- `count()`
- `sort()`
- `limit()`

They are applied on the result set from the query and not the data stored in the database.

MongoDB will always assume that you want to `sort()` before you `limit()` to avoid missing data so will apply them in the correct order for you.

Example syntax for these are:

```
db.collection.find().sort({ "field_1": 1, "field_2": -1 }).limit(10)
```

### Cursor-like Stages

These are also available as an pipeline stage, they are:

- sort - `{ $sort: { fieldOne: 1, fieldTwo: -1 } }`
- skip - `{ $skip: 10 }`
- limit - `{ $limit: 100 }`
- count - `{ $count: "nameOfCountField" }`

If a `sort` stage is before an `project`, `unwind`, or `group` it can take advantages of our indexes, otherwise it will be in-memory. By default, sort operations are limited to 100MB of RAM, to allow larger datasets you can provide `{ allowDiskUse: true }` as an aggregation argument.

### `$sample`

It will select a random set of documents from a collection, for example: `{ $sample: { size: 100 } }`
