## Aggregation Framework

It is another way to query data in MongoDB that exceeds the capabilities of querying with MQL.

It is a pipeline that is done in the stages that they are provided.

Non-filtering stages do not modify the original data, instead they work with the data in the cursor.

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
