## Architecture

- MongoDB Query Language
- MongoDB Document Data Model
  - Namespaces
  - Indexes
  - Data structures
  - Replication (Write and read concerns)
- Storage Layer
  - System calls
  - Disk flush
  - File Structures
  - Compression
- Security
- Administration

### High Availability and Failover

- Uses Raft protocol
- Replica Sets are groups of different MongoDs that share the same information
- Primary and x number of secondaries
- Primary handles interaction and the changes are sent out to the secondaries

#### Scalable

- Sharded clusters provide scalability
- Composed of shards which are replica sets

#### Config Servers

- Special type of replica set
- Manages meta information of our system

### Document

Way to organise and store data as a set of field-value pairs.

### Collection

An organised store of documents in MongoDB, usually with common fields between documents.

### BSON

- BSON is a binary JSON data format
- It extends the JSON value types to include integers, doubles, dates and binary data (to support images etc)
- Lightweight; the space required to represent data is kept to a minimum, good for storage efficiency and network transfer times
- Traversable; to support the variety of operations necessary to support reading/writing/indexing
- Efficient; encoding and decoding data into/from BSON in the drivers can be done quickly (due to the use of C data types)
- MongoDB drivers send and receive data as BSON, when data is written to MongoDB it is stored as BSON
- The drivers then map BSON to an appropriate data type for the language

### Local Database

This is local to the node and is not replicated, you can edit it with the correct permissions but you should avoid touching it as it contains important information for MongoDB to operate.

`oplog.rs` is central to our replication mechanism.

The size of our oplog will impact the replication window, if a node goes offline it can check with the other nodes which has the last update that it made in its oplog, if it exists, it will just replay the following actions and be back up to date. If the node has been offline for too long, or too much has changed resulting in the other oplogs to have moved on, the node will be in recovery mode.

You can change the oplog size, but will default at 5% of the available disk.
