## Aggregation Framework Performance

There are two categories of aggregation queries.

### 1. Realtime Processing

These are used to provide data to **applications**, query performance for these is **more** important.

### 2. Batched Processing

These are used to provide data for **analytics**, query performance for these is **less** important.

### Using indexes and Pipeline Position

If a pipeline stage cannot use an index, any stages after that cannot then use indexes. The query optimiser will do what it can to ensure that a stage can be moved forward to utilise indexes but only if it can be sure it would not change the output.

To see how the pipeline will run and if we are using indexes, you can pass the `{ explain: true }` document to the aggregation.

Ideally stages such as `$match` and `$sort` could be closest to the front of the pipeline to get the most out of existing indexes. If you are doing a `$limit` and `$sort`, try to keep these early in the pipeline, that way the server can do a `top-k` sort. This is where the server is able to only allocate memory for the final number of documents (the limit) which can happen even without indexes, this can be very performant.

### Memory Constraints

Results are subject to a 16MB document limit, aggregations are generally output as a single document which is susceptible to this limit. The limit does not apply as they flow through the pipeline, so you can be over and then project the output down under the limit. Using `$limit` and `$project` are the best ways to avoid this problem.

For each stage in the pipeline, there is a limit of 100MB of RAM. The best way to avoid this is to ensure that you reduce your memory requirements by hitting indexes as they are much smaller than the documents they reference. The `{ allowDiskUse: true }` allows you to get around this by using the disk but it comes with the cost of serious performance degradation. This makes more sense with batch processing.

### Sharded Cluster Performance

Similar to standard query performance, the shard key plays a big part in the performance, if the aggregation can be sent to a single shard then it would be performant. If it has to go across shards then all of those will have to perform the aggregations and then has to be merged/combined. This merge is often done on random shards, but for some operations it has to take place on the primary shard. This should be considered if you do a lot of these operations then the primary shard will be working a lot harder than the others and limits the benefits of the horizontal scaling. A larger primary shard may be required in this case.

The following will require the primary shard to do the merge workload:

- `$out`
- `$facet`
- `$lookup`
- `$graphLookup`

### Avoid Needless `$project` Stages

By adding extra project stages the aggregation framework assumed we knew what we were doing. If the aggregation framework can determine the final document based only on the initial input, internally it will project away unnecessary fields. By adding extra projections, it may have to resort back the document to fetch these values.

For example, if we have an index on a `name` and we only use that value in a `$match` and `$group` then the framework doesn't have to fetch any extra data for us.

#### Avoid `$unwind` if possible

Unwinding an array creates a lot more documents that are then often narrowed back down, this can be expensive with large datasets. If possible, try to use accumulator expressions `$map`, `$reduce`, and `$filter` before you unwind.
