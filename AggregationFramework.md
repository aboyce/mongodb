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

### Grouping

The aggregation framework exposes the `$group` operator that takes the incoming stream of data, and siphons it into multiple distinct reservoirs.

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
