## Querying

### The `$`

Denotes the use of an operator, or addresses the field value (like a pointer).

### Comparison Operators

- `$eq` - equal to
- `$ne` - no equal to
- `$gt` - greater than
- `$lt` - less than
- `$gte` - greater than or equal to
- `$lte` - less than or equal to

They are used in the follow syntax:

```
{ field: { operator: value } }
```

If an operator is not specified, `$eq` is used as a default.

### Logic Operators

- `$and` - match all of the specified query clauses
- `$or` - at least one of the query clauses is matched
- `$nor` - fail to match both given clauses
- `$not` - negates the query requirement

`$and`, `$or` and `$nor` are used in the follow syntax:

```
{ operator: [ { statement1 }, { statement2 }, ... ] }
```

`$not` is used in the follow syntax:

```
{ $not: { statement } }
```

If an operator is not specified, `$and` is used as a default.

### Expressive Query Operator

- `$expr` - allows the use of aggregation expressions within the query language, it also allows us to use variables and conditional statements

It is used in the follow syntax:

```
{ $expr: { expression } }
```

An example of finding an object that has `field_a` set the same as `field_b` would be:

```
db.collection.find({
  $expr: { $eq: [ "$field_a", "$field_b" ] }
})
```

### Array Operators

- `$size` will return a cursor with all documents where the specified array field is exactly a given length
- `$all` will return a cursor with all documents in which the specified array field contains all the given elements regardless of their order in the array

If you `find` on an array, it will have to be an exact match in the same order as the search, to ignore the ordering, you can use `$all` to match on all values but ignore the order in the query.

For example:

```
{ "field_a": { "$size": 2, "$all": [ "value_1", "value_2" ] } }
```
