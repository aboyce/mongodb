## Sharding

A way of distributing data across multiple machines, used to support deployments with very large data sets and hight throughput operations. This is required when the capacity of a single sever can be overwhelmed.

MongoDB supports horizontal scaling through sharding, it shards at the collection lever, distributing data across the sharded in the cluster.

A sharded cluster consists of:

- Shard
- Config servers
- `mongos`

#### Shard

A subset of the sharded data, each shard can/should be deployed as a replica set.

There is also a primary shard, each database will be assigned a primary shard and all none sharded collections on that database will remain on that shard as not all collections on a sharded cluster need to be distributed. You can change the primary shard in a database if required.

#### `mongos`

Acts as a query router, an interface between client applications and the sharded cluster. Clients now connect to `mongos` rather than individually and we can have many instances to service applications across the sharded cluster.

It uses the metadata for what information is stored in each shard to route queries, this metadata is stored on the config server but a cache is stored within `mongos`.

If a query comes in that can spread multiple shards, it has to be sent to all the shards, the returned data will be returned to `mongos` and it will perform a `SHARD_MERGE` of the aggregated data or on a randomly chosen shard in the cluster.

When it is tasked with a query that spreads multiple shards, it will open a cursor with each shard and ask it for the results, these are then merged and returned. For specific manipulation the following occurs:

- `sort()` the `mongos` pushes the sort to each shard and merge-sorts the result
- `limit()` the `mongos` pushes the limit to each targeted shard, then reapplies the limit to the merged set of results
- `skip()` the `mongos` performs the skip on the merged set of results

`mongos` inherits users from the config servers.

#### Config Servers

Store metadata and configuration for the cluster, this data is required to be highly available and required frequently by `monogos`. To ensure availability, we deploy a Config Server Replica Set.

These keep track of mappings from which data is available from with which area of the shard, this can simply be letters A-M are on shard 0 for example. They are also responsible for moving the data around, if the section of names A-M is too large compared to the other shards, it will change the mappings/locations to spread the data out more evenly. THey would update the metadata locally and then send this information out to the correct shards.

##### Config Database

You should not write any data directly to it, it is used and maintained internally by MongoDB. You can use `sh.status()` to get useful data about the sharded cluster, a lot of this is also available in the config database.

There is some useful information in:

- `db.databases.find().pretty()`
- `db.collections.find().pretty()`
- `db.shards.find().pretty()`
- `db.chunks.find().pretty()`
- `db.mongos.find().pretty()`

### When to Shard

Things to check before sharding would be:

- Is it still economically viable to vertically scale up (would it solve bottlenecks, are there larger nodes available at a sensible price?)
- Going bigger is not always easier, larger backups, larger indexes etc. all add complexity to just making things bigger. A general rule is that individual servers should contain 2TB - 5TB of data, more than that makes it too time consuming to operate.
- Specific workloads benefit from horizontal scaling, aggregations and geographical concerns may have more impact than just a bigger node. Some aggregations can be parallelised and would be better running on a sharded cluster

### Chunks

Sharded data is partitioned into chunks, each chunk has an inclusive lower and exclusive upper range based on the shard key.

You can find the chunks and their information with:

```
use config
db.chunks.findOne()
```

A chunks lower bound is inclusive where the upper bound is exclusive.

When a collection is sharded with documents it is immediately defined with one chunk, this initial chunk goes from min key to max key. The different values the shard key may hold is called the key space of the sharded collection. As time progresses, the cluster will separate that initial chunk in to several others to allow data to be evenly distributed between shards, all documents of the same chunk live in the same shard.

By default, MongoDB takes 64MB as the default chunk size, so if a chunk approaches that size it will be split. You can define the chunk size between 1MB and 1024MB, it can be changed at runtime.

Shards could contain multiple chunks.

#### Jumbo Chunks

These are larger than the defined chunk size and cannot be moved and there will be no attempt to balance it. In some circumstances you will not be able to split down a jumbo chunk, this is where the shard key frequency comes in to help avoid this scenario.

#### Balancing

As you insert data into a collection, the number of chunks on any given shard will grow. The MongoDB balancer identifies which shards have too many chunks and will automatically move chunks across the shards in the cluster to balance them out.

The balancer runs on the primary member of the config server replica set, it is completely automated and requires minimal user configuration.

If it detects there is an imbalance it starts a balancer round, a given shard cannot participate in more than one migration at a time. For a cluster of 3 you can only move one chunk, the balancer will keep triggering rounds until it detects that the cluster is balanced. It will also detect if a chunk needs to be split up.

There is a performance impact to this maintenance, because of this you can control the balancer. If you stop the balancer mid round that last round will compete. Some of the ways to interact are:

- `sh.startBalancer(timeout, interval)`
- `sh.stopBalancer(timeout, interval)`
- `sh.setBalancerState(boolean)`

### How to Shard

You can enable sharding with `sh.enableSharding(database)` for a specific database, this does not do any sharding just that they are now eligible for sharding. You then want to `db.collection.createIndex()` to create the indexes for the shard keys. You can then `sh.shardCollection("database.collection", { shard key })` to actually shard the collection.

### Shard Keys

The shard key is used to distribute the collection's documents across sharded, it is the indexed field or fields to partition data in a sharded collection and to distribute it across the shards in the cluster.

MongoDB uses the shard key to separate the documents into logical groupings that MongoDB can then distribute. These groups are called **Chunks**. Because the shard key is used to chunk the documents, it must be present in every document.

If you provide the shard key are part of your read query, MongoDB can direct that read request to a specific server in the cluster. Ideally you shard keys should cover the majority of the queries you run on that collection, if not every single node in the cluster will have to be queried.

_In v4.4 documents can miss the shard key fields._

You select the shard key when sharding a collection.

_In v5 you can reshard a collection by changing the key, ing v4.4 you can refine a shard key by adding a suffix field(s) to the existing key._

A document's shard key determines its distribution across the shards.

If you have a compound shard key, you can perform targeted queries using a prefix of the shard key in order, for example, if you have a shard key made up of `{ "a": 1, "b": 1, "c": 1 }` the following queries can be targeted to an individual shard:

- `db.collection.find({ "a": ... })`
- `db.collection.find({ "a": ..., "b": ... })`
- `db.collection.find({ "a": ..., "b": ..., "c": ... })`

If though you use random parts of the compound key, you will not be able to target specific shards but instead require a scatter/gather query, for example:

- `db.collection.find({ "b": ... })`
- `db.collection.find({ "c": ... })`

#### Shard Key Index

To shard a populated collection, the collection must have an index that starts with the shard key, if sharding an empty collection, the supported index is created. The indexes must exist first before you can select the indexed fields for your shard key.

#### Shard Key Strategy

The choice of key affect performance, efficiency and scalability of a sharded cluster. It can even be a bottleneck on performant hardware and can affect the sharding strategy you cluster can use.

The goal is a shard key whose values provide good write distribution:

- Cardinality; the number of unique values, it should have high cardinality, i.e. there are many possible unique shard key values
- Frequency; how often a unique value occurs in the data, it should have low frequency, i.e. low repetition of a unique shard key value, so you want to avoid a scenario where too many have the same value effectively overloading certain shards leaving other inactive
- Monotonic change; the shard key value changes at a steady and predictable rate (i.e. timestamps and dates), this is bad because all of the most recent documents will end up the higher value boundary chunk whilst the others become less and less active. The `_id` field is monotonically increasing, so sharding on the `_id` field is not a good idea

The ideal shard key is:

- High cardinality (lots of unique values)
- Low frequency (very little repetition of those unique values)
- Non monotonically changing (non-linear change in values)

Read isolation should also be considered, if your query can include the shard key it will be very quick as it only has to ask the matching shard. If you don't use the shard key for most queries you are adding more work across the cluster.

### Advantages

#### Read/writes

Requests are distributed across the shards, so each shard processes a subset of cluster operations. You can horizontally scale across the cluster by adding more shards. Queries that include the shard key or prefix of a compound shard key, `mongos` can target the query at a specific shard or set of shards in a targeted operation.

#### Storage Capacity

Each shard contains a subset of the total cluster data, so less to handle. As the data grows more shards can be added to increase the storage capacity.

#### High Availability

Config servers and shards as replica sets provide increased availability. If a sharded replica set becomes unavailable, the sharded cluster can continue to perform partial read and writes. Data that existed on those unavailable shards would not be accessed, reads or writes dedicated at available shards can still continue.

### Considerations

Sharded clusters and more complex and require more maintenance, planning and execution.

Once sharded, there is no method to unshard a collection.

Shard key choice should be carefully considered to avoid scalability and performance issues.

### Sharded and Non-sharded Collections

A database can have a mixture of sharded and unsharded collections. Sharded collections are partitioned and distributed across the shards in the cluster, unsharded collections are stored on a primary shard.

### Connecting to a Sharded Cluster

You must connect to the `mongos` router to interact with **any** collection in the sharded cluster. Clients should **never** connect to a single shard to perform operations.

### Sharding Strategies

#### Hashed Sharding

Involves computing an hash of the shard key field's value, each chunk is then assigned a range based on the hashed shard key values.

_Hashing is done automatically, clients do not need to do any hashing._

##### Pros

The hashed values are unlikely to be on the same chunk, data distribution based on hashed values facilitates more even data distribution, especially in data sets where the shard key changes monotonically (always increasing or remaining constant, never decreasing).

##### Cons

Range-based queries on the shard key are less likely to target a single shard, resulting in more cluster wide broadcast operations. They cannot support geographically isolated read operations using zoned sharding. You cannot create hashed compound indexes, can only be created on a single non-array field (this is why they are best for the monotonically changing fields like timestamps).

#### Ranged Sharding

Dividing the data into ranges based on the shard key values, each chunk is then assigned a range based on the shard key values.

##### Pros

Values that are close are more likely to reside on the same chunk, this allows for target operations to those shards that contain the required data.

##### Cons

Efficiency depends on shard key, poorly considered keys can result in uneven distribution of data and create performance bottlenecks.
