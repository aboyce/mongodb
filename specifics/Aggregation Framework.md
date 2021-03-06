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

### Lookup Stage

The `$lookup` stage lets you combine information from two collections, effectively a left outer join in the SQL world. This is a combination of all document entries on the left with matching documents from the right.

The collection you specify in the `from` field cannot be sharded and must exist in the same database.

The fields `localField` and `foreignField` can bot resolve to either a single value or an array. They are matched on equality.

Any matches will appear as a nested array field on the source collection, if there are no matches it will be an empty array. It is common to follow this stage with a match stage to filter out documents with no matches.

It will retrieve the entire document that matched, not just the `foreignField`.

### Redact State

The redact stage help to protect information from unauthorised access. It is not for restricting access to a collection, only the documents within. To perform this redaction stage the user will already have read access, so should not be used for implementing access control.

If comparing to a field in the document, the field must be present at every level if using descend or the redaction is likely to error.

The document can be made up of sub documents, if deemed to be redacted, it will apply to that document level and those beneath it. It will traverse the document and make the decision at each level on the `$cond`, it will keep going until it hits a prune and to the user this document will not exist at all.

It takes an expression that resolves to one of three values;

#### `$$DESCEND`

Retains the field at the current document level being evaluated except for sub documents and arrays of documents and progresses to the next level down to evaluate.

#### `$$PRUNE`

Remove all fields at the current document level without further inspection (including sub documents), the inverse of `$$KEEP`.

#### `$$KEEP`

Retain all fields at the current document level without further inspection (including sub documents), the inverse of `$$PRUNE`.

### Out Stage

The `$out` stage must be the last stage in the pipeline, because of this it cannot be used within a `$facet` stage.

It will create a collection with the name specified if it does not exist or will overwrite an existing collection if it does. If it does overwrite a collection, any existing indexes will stay in place, this means the output must honour index restrictions.

You can only create the new collection within the same database.

Will not output to the collection if the pipeline errors.

### Merge Stage

This builds on the `$out` stage by adding more flexibility, you can use it to merge output into a collection, that collection can be in a different database, and it can also be sharded.

You can specify the fields on which to match the documents using the `on` argument. If not specified, it will use the `_id` field as the merging field when unsharded, if it is sharded, it will use a combination of the `_id` and the shard key. If you specify your own merging key, there must be a unique index on it.

As part of the merge process there are two fields, `whenNotMatched` and `whenMatched` to specify how the merge should take place. These are optional. By default, no match will result in an insert, a match will result in a merge (adding top level fields, preserving existing).

The options for `whenNotMatched` are:

- `insert` - insert into the collection
- `discard` - discard it (it may not be something that is wanted)
- `fail` - fail the stage (you may be expecting a document to previous exist and a merge to take place)

The options for `whenMatched` are:

- `merge` - merge with the existing document
- `replace` - replace the full document with the incoming document
- `keepExisting` - if it is already there keep the existing document
- `fail` - fail the stage (you may be expecting a document to not have existed)
- You can also provide a custom pipeline to create the new document to be written to the collection. This is limited to stages that do single document transformations, they are applied to the existing document in the target collection. Any `$` variables refer to the existing document, but you can use `$$new.value` to refer to values in the incoming document. For example, `{ $set: { total: { $sum: [ "$total", "$$new.total" ] } } }` will add the existing total with the new total and set that to the merged document. You can use `$replaceWith` and `$mergeObjects` to manually create merging logic with custom overrides.

### Graph Lookup Stage

Some more complex data structures can be graph or tree hierarchies, an example could be an organisational chart, and these require specific lookups.

The `$graphLookup` stage allow the combination of flexible datasets with graph or graph-like operations.

In SQL this type of lookup is via recursive common table expressions.

A graph lookup allows looking up recursively a set of documents with a defined relationship to a starting document, for example an employee to look at their management chain.

It will default to a `maxDepth` of 0, i.e. the first layer of matches. If you are doing the lookup across collections you will have to increase the `maxDepth` by 1.

You can use `depthField` to show how many levels away the matched document is to the original document.

You can limit the recursive matches with `restrictSearchWithMatch` to avoid matching on everything.

The steps of this stage are;

1. Input documents flow into this stage
2. It targets the search to the collection designated by the `from` parameter
3. For each input document, the search begins with the `startWith` value
4. It matches the `startWith` value against the field designated by `connectToField` from the `from` collection.
5. For each matching document, it takes the value of the `connectedFromField` and checks every document in the `from` collection for a matching `connectToField` value. For each of these matches, it adds the matching document in the `from` collection to an array field specified via the `as` value.
   This happens recursively until no more matches are found, or until the `maxDepth` value is reached.
6. The array field is them added to the input document.

Due to its recursive nature, you will have to consider the memory resources required. You can use `$allowDiskUse` to get around the default memory limitations. This though may not be enough with some complex queries.

Having the `connectToField` indexed should help with performance.

The `connectFromField` **cannot** be sharded.

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

### Facets

Facet navigation is often used for browsing data catalogs and grouping data into analytic use cases. They can be seen as filter categories, i.e. you could have people with a certain job title and then add a location facet to further limit though results.

#### `$sortByCount`

You can use `$sortByCount` to display the categories and the count of each of those. An example of this could be `[..., { $sortByCount: '$category' }]` to get a count of each of the categories.

It works just like a `$group` stage followed immediately by a `$sort` in descending direction.

#### Buckets

Buckets are groups specified by a range of values, as opposed to individual values. For example:

```
{
  $bucket: {
    groupBy: '$employee_count',
    boundaries: [0, 50, 100, 150, Infinity]
  }
}
```

The lower values are inclusive for their bucket, so for above it would be 0-49, 50-99, 100-149, 150-Infinity.

If you have values that fall outside of the boundaries you specify, or have different data types to categorise, you will get an error. To avoid this, you can add a `default` value so that these can fall into this bucket.

All values that define the boundaries need to have the same data type and you must always specify at least **2** values.

If the default output of the `$bucket` stage is too minimal, you can use `output` to provide a structure of the results. This will remove the `count` that is provided by default.

To avoid having to manually specifying the buckets, you can get MongoDB to automatically create the buckets for you with `$bucketAuto`. For example:

```
{
  $bucketAuto: {
    groupBy: '$employee_count',
    buckets: 10
  }
}
```

For this the `_id` will be an object containing the `min` and `max` for each of the buckets, along with the `count` for each of them. MongoDB will try to evenly balance the number in each bucket.

#### Multiple Buckets

To combine multiple buckets, you can use the `$facet` stage. This allows you to provide a bucket for each object value. You provide a sub-pipeline for each facet/bucket, they will receive the same number as documents as the `$facet` stage itself but run independently of one another, the output of one sub-pipeline cannot be used by the following ones. For example:

```
{
  $facet: {
    firstBucketAbove: [
      {
        $match: { location: 'Here' }
      },
      {
        $bucket: {
          groupBy: '$employee_count',
          boundaries: [0, 50, 100, 150, Infinity]
        }
      }
    ],
    secondBucketAbove: [
      {
        $match: { name: 'Value' }
      },
      {
        $bucketAuto: {
          groupBy: '$employee_count',
          buckets: 10
        }
      }
    ]
  }
}
```
