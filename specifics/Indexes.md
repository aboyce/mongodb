## Indexes

A special data structure that stores a small portion of the collection's data set in an easy to traverse form.

You should add indexes if you often query on a specific field to support the queries. Both finding and sorting by a field would be assisted with an index.

A single field index is only on a single field, for example:

```
db.collection.createIndex({ "field_1": 1 })
```

A compound index is on multiple fields, for example:

```
db.collection.createIndex({ "field_1": 1, "field_2": -1 })
```
