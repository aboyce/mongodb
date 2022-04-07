## Monitoring and Debugging

#### Groups on Namespace (hottest collections)

```
db.aggregate([ { $currentOp: {} }, { $group: { _id: "$ns", count: { $sum: 1 } } }, { $sort: { _id:1 } } ])
```

#### Long running operations (number of seconds)

```
db.aggregate([ { $currentOp: {} }, { $match: {"secs_running": { $gte: 1 } } } ])
```
