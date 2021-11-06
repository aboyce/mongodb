## Sharding

A way of distributing data across multiple machines, used to support deployments with very large data sets and hight throughput operations. This is required when the capacity of a single sever can be overwhelmed.

MongoDB supports horizontal scaling through sharding, it shards at the collection lever, distributing data across the sharded in the cluster.

A sharded cluster consists of:

- `shard`; a subset of the sharded data, each shard can be deployed as a replica set
- `mongos`; acts as a query router, an interface between client applications and the sharded cluster
- `config servers`; store metadata and configuration for the cluster

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
