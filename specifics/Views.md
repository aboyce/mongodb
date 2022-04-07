## Views

MongoDB enables non-materialised views, i.e. they are computed every time a read operation takes place against that view. From a users perspective they are collections but they are essentially aggregation pipeline outputs.

They allow us to create vertical and horizontal slices of a collection. Where vertical slicing is done through the use of a `$project` stage, i.e. changing the shape of a document without changing the total number of documents being returned. Horizontal slicing is done through the use of `$match` stages, so returning a subset of documents that are appropriate for the view, it will change the number of documents returned but not the shape of the objects.

To create a view you can use `db.createView(<view>, <source>, <pipeline>, <collation>)`.

You can perform all read operations on views, including `db.view.aggregate()`.

They are restrictions though:

- No write operations
- No index operations (will use the source collection indexes though for operation)
- Cannot rename a view (can just recreate it)
- Collation restrictions
- No `mapReduce`, `$text`, `geoNear` or `$geoNear`
- `find()` operations with projection operators are not permitted

You can use `db.system.views.find()` to see the views.
