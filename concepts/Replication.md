## Replication

MongoDB uses asynchronous statement based replication because it is platform independent and allows more flexibility within a replica set.

In MongoDB a group of nodes that all hold the same data is called a Replica Set. All interaction takes place with the primary node and the secondaries job is to keep in sync with it through replication.

The nodes decide which secondary will become the primary in an election, this is done by the secondaries voting on which node will become the primary.

At the heart of the replication mechanism is the operations log (Op Log), this is a statement based log that keeps track of all write operations acknowledged by the replica sets.

You can set a priority on which node should be the primary if the circumstances are appropriate. This integer value between 0 and 1000, members with a higher priority tend to be elected primaries more often. A change in a priority will trigger an election as that is considered a topology change. Setting the priority to 0 effectively excludes the member from ever becoming a primary.

Replica sets can go upto **50** members, this can be useful to spread our data out across geographical locations. A maximum of **7** of those members can be voting members (more than 7 can delay the election process without adding any value to availability).

### Replication Configuration

A JSON object that defines the configuration options for the replica set, can be configured manually from the shell.

There are a set of mongo shell replication helper methods to make it easier to manage:

- `rs.add()`
- `rs.initiate()`
- `rs.remove()`
- `rs.reconfig()`
- `rs.config()`

There are also some helper commands for more information:

- `rs.status()` reports health on the replica set nodes, uses the heartbeat data
- `rs.isMaster()` describes the role of the node we ran the command on
- `rs.serverStatus()['repl']` similar to `isMaster`, a subset of `serverStatus`
- `rs.printReplicationInfo()` only returns oplog data relative to the current node

### RAFT

The protocol under the hood of the replication, the TLDR is:

- if there is no leader the nodes will volunteer to be elected and request a vote, they all have a clock at a slightly different interval to prevent everyone requesting at the same time
- the candidate with the majority of votes is elected to be new leader
- if a tie happens, they try again at the next interval with a new election, this is not ideal though from a clients perspective
- the leader then sends out heartbeats which reset the others clocks to prevent a new election
- the writes happen on the leader
- the leader receives a request to change something, stores that in its log, then broadcasts the change to the rest of the nodes
- the nodes receive this change and temporarily add it to their logs
- when the leader had received an acknowledgement from the majority of the nodes it will acknowledge that the request has been successful to the requester
- the leader will then broadcast that the change was successful and the other nodes will then commit this change to their log

- if a split happens a new election term starts
- a split without a majority will not be able to confirm requests
- when they rejoin the majority (with an assumed new leader) the previous leader will recognise that it's term is no longer the latest and start following again

#### Elections

Elections take place whenever there is a change in topology, reconfiguring will always trigger an election that may or may not elect the same primary.

A new primary will always be elected when:

- the current primary becomes unavailable
- the current primary steps down with `rs.stepDown()`

The priority is the likelihood that a node will become the primary during an election, the default primary for a node is 1. A node with a priority of 0 cannot be a primary, these are then class as passive nodes in the topology.

### Arbiter

Holds no data but can vote in elections, it cannot become a primary. This avoids and split votes due to even numbers. They are not advised though as they can cause significant consistency issues. These are configured with the config `arbiterOnly`, hidden nodes must have a `priority` of 0 as they cannot become primaries.

### Hidden Nodes

These can provide specific readonly workloads or have copies of the data which are hidden from the application. These are configured with the config `hidden`, hidden nodes must have a `priority` of 0 as they cannot become primaries.

### Delayed Nodes

These are hidden nodes but with a delay in their replication, this adds resilience to application level corruption without relying on cold backup files. For example, you could delay the replication by an hour and then if anything irreversible happens there is an hour gap to restore any data. These are configured with the config `slaveDelay` which is the time to delay in seconds. If you have a delayed node it is also assumed that the `hidden` config is set to `true` and the `priority` config is set to 0.

### Reading from a Secondary

By default MongoDB does not let you read from a secondary node, this is because there is a chance it is out of date from the primary. To permit this behaviour you can use `rs.slaveOk()`.

Even though you can then read from the secondary, you will never be able to write to a secondary node.

### Stuck in Secondaries

If the primary loses access to the other nodes, i.e. the other secondaries become unavailable, the primary no longer holds a majority so would become itself a secondary node. This would mean that all nodes in the replica set are then secondaries and we cannot write anything to the replica set.

In a three node replica set that lost two of the secondaries, the full replica set would then be stuck.

### Write Concerns

The number of acknowledgements of a write will increase durability but will also increase the time it takes for a write to be acknowledged to the client.

Write concern levels:

- `0` - don't wait for an acknowledgement
- `1` (default) - wait for an acknowledgement from the primary only
- `>=2` - wait for acknowledgement from the primary and one or more secondaries
- `majority` - wait for acknowledgement from the majority of the replica set members (a simple majority, divide the number of replicas by 2 and round up)

Write concern options:

- `wtimeout` - the time to wait for the requested write concern before marking the operation as failed (this does not mean that the operation itself failed, just that the application did not get the level of durability that it requested)
- `j` (journal) - requires the node to commit the write operation to the journal before returning an acknowledgement (this indicates that the operation has been written to disk rather than just into memory)

If you have a write concern of more nodes than are currently available (e.g. you request 4 but only 3 are available) this write operation would be blocked from happening. This is the purpose of the `wtimeout` option, to avoid an operation from running/pending for too long, the error would be about the guarantee not being reached rather than the write itself failed.

### Read Concerns

Similar to write concern but for writes, where the developer can request a specific durability for reads. If a read occurred on a primary that had not had chance to replicate the data and then went offline, when it returned online the write that was read would be rolled back, this would result in the application having data that no longer existed in the database.

You can choose whether you receive the most recent data in the cluster or the data received by the majority of members in the cluster.

A document that does not meet the specified read concern is not a document that is guaranteed to be lost, it just means at the time of the read it had not propagated to enough nodes to meet the requested durability guarantees.

Read concern levels:

- `local` - return the most recent data in the cluster, anything written to the primary can be classed as local
- `available` - the same as `local` read concerns in replica sets but special for sharded deployments (this is the default for read operations against secondaries)
- `majority` - only returns data that has been acknowledged by a majority of nodes in the replica set (the only way data could be lost here would be if a majority of replica set members went down and the data had not been replicated to the remaining replica set members, very unlikely)
- `linearizable` - this is an extension to `majority` in that it provides read your own write functionality (this is best if you want the combination of fast but safe data), it has the strongest durability guarantees

### Read Preferences

Read preferences allow you to route your read operations to specific members of a replica set, it is principally a driver side setting and may vary slightly for each version.

Read preference modes:

- `primary` (default) - routes all read operations to the primary only
- `primaryPreferred` - routes read operations to the primary, if the primary is unavailable via a failover or election event, the secondaries can be used
- `secondary` - routes read operations to only secondaries in the replica set
- `secondaryPreferred` - routes read operations to the secondaries, if all of the secondaries are unavailable then they are routed to the primary
- `nearest` - routes the read operation to the node with the least network latency to the host regardless of the members type

Due to the replication delay, reading from secondaries can result in stale data. It is possible to read stale data in all preferences apart from `primary` but the trade off is that the secondaries are only there for availability and are not sharing the load. How stale the data is depends on the delay replicating data to the secondaries (could be significant if the secondaries are geographically separate).
