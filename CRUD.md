## CRUD

All documents must have a unique identifier/`_id` field, all other fields could be identical, `_id` does not have to have the type `ObjectId()` though, as long as it is unique. `Object()` is the default type.

One:

- `deleteOne()`
- `updateOne()`
- `findOne()`

Many:

- `deleteMany()`
- `updateMany()`
- `find()`

### Creating/Inserting

To insert you can use `db.collection.insertOne()` passing the document you want to add. If you provide the `_id` yourself it must be unique. To insert an array you can pass an array. If inserting multiple and one fails, it will stop inserting documents at that point, to get around this you can include the `{ "ordered": false }` option to continue inserting all those that will.

Collections and databases are created when they are being used.

### Reading

You can use `find` and `findOne`.

### Updating

You can use update operators to make life easier.

#### Fields

- `$currentDate` sets the value to the current date, either as a Date or Timestamp
- `$inc` to increment the value by the provided amount, for example: `{ "$inc": { "pop": 10 } }`
- `$mul` multiples the value by the provided amount
- `$min` only updates the field if the specified value is less than the existing one
- `$max` only updates the field if the specified value is greater than the existing one
- `$rename` renames a field
- `$set` to set/add a value, for example: `{ "$set": { "pop": 17630 } }`
- `$setOnInsert` sets the value of a field if an update results in an insert of a document. Does not effect on update operations that modify existing documents.
- `$unset` removed the specified field from the document

#### Arrays

- `$` acts as a placeholder to update the first element that matches the query
- `$[]` acts as a placeholder to update all elements in an array for the documents that match the query
- `$[<identifier>]` acts as a placeholder to update all elements that match the `arrayFilters` condition for the documents that match the query
- `$addToSet` adds elements to an array only if they do not already exist
- `$pop` removes the first or last item of an array
- `$pull` removes all array elements that match the query
- `$push` adds an item to an array, for example `{ "$push": { "scores": { "type": "extra credit", "score": 100 } } }`
- `$pullAll` removes all matching values from an array

#### Modifiers

- `$each` modifies the `$push` and `$addToSet` operators to append multiple items for array updates
- `$position` modifies the `$push` operator to specify the position in the array to add elements
- `$slice` modifies the `$push` operator to limit the size of the updated arrays
- `$sort` modifies the `$push` operator to reorder documents stored in an array

#### Bitwise

- `$bit` performs bitwise `AND`, `OR`, and `XOR` updates of integer values

### Deleting

The only time you can be sure you are using `deleteOne` correctly is if you are filtering on `_id` as anything else could return an unexpected document to delete.

To delete a collection you can use `db.collection.drop()`
