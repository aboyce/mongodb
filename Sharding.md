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

It uses the metadata for what information is stored in each shard to route queries, this metadata is stored on the config server.

If a query comes in that can spread multiple shards, it has to be sent to all the shards, the returned data will be returned to `mongos` and it will perform a `SHARD_MERGE` of the aggregated data or on a randomly chosen shard in the cluster.

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

### Shard Keys

The shard key is used to distribute the collection's documents across sharded, the key consists of a field or multiple fields in the documents.

_In v4.4 documents can miss the shard key fields._

You select the shard key when sharding a collection.

_In v5 you can reshard a collection by changing the key, ing v4.4 you can refine a shard key by adding a suffix field(s) to the existing key._

A document's shard key determines its distribution across the shards.

#### Shard Key Index

To shard a populated collection, the collection must have an index that starts with the shard key, if sharding an empty collection, the supported index is created.

#### Shard Key Strategy

The choice of key affect performance, efficiency and scalability of a sharded cluster. It can even be a bottleneck on performant hardware and can affect the sharding strategy you cluster can use.

### Chunks

Sharded data is partitioned into chunks, each chunk has an inclusive lower and exclusive upper range based on the shard key.

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

### Connecting to a Sharded

You must connect to the `mongos` router to interact with **any** collection in the sharded cluster. Clients should **never** connect to a single shard to perform operations.

### Sharding Strategies

#### Hashed Sharding

Involves computing an hash of the shard key field's value, each chunk is then assigned a range based on the hashed shard key values.

_Hashing is done automatically, clients do not need to do any hashing._

##### Pros

The hashed values are unlikely to be on the same chunk, data distribution based on hashed values facilitates more even data distribution, especially in data sets where the shard key changes monotonically (always increasing or remaining constant, never decreasing).

##### Cons

Range-based queries on the shard key are less likely to target a single shard, resulting in more cluster wide broadcast operations.

#### Ranged Sharding

Dividing the data into ranges based on the shard key values, each chunk is then assigned a range based on the shard key values.

##### Pros

Values that are close are more likely to reside on the same chunk, this allows for target operations to those shards that contain the required data.

##### Cons

Efficiency depends on shard key, poorly considered keys can result in uneven distribution of data and create performance bottlenecks.
