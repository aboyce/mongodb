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
- if a tie happens, they try again at the next interval
- the leader then sends out heartbeats which reset the others clocks to prevent a new election
- the writes happen on the leader
- the leader receives a request to change something, stores that in its log, then broadcasts the change to the rest of the nodes
- the nodes receive this change and temporarily add it to their logs
- when the leader had received an acknowledgement from the majority of the nodes it will acknowledge that the request has been successful to the requester
- the leader will then broadcast that the change was successful and the other nodes will then commit this change to their log

- if a split happens a new election term starts
- a split without a majority will not be able to confirm requests
- when they rejoin the majority (with an assumed new leader) the previous leader will recognise that its term is no longer the latest and start following again

### Arbiter

Holds no data but can vote in elections, it cannot become a primary. This avoids and split votes due to even numbers. They are not advised though as they can cause significant consistency issues. These are configured with the config `arbiterOnly`, hidden nodes must have a `priority` of 0 as they cannot become primaries.

### Hidden Nodes

These can provide specific readonly workloads or have copies of the data which are hidden from the application. These are configured with the config `hidden`, hidden nodes must have a `priority` of 0 as they cannot become primaries.

### Delayed Nodes

These are hidden nodes but with a delay in their replication, this adds resilience to application level corruption without relying on cold backup files. For example, you could delay the replication by an hour and then if anything irreversible happens there is an hour gap to restore any data. These are configured with the config `slaveDelay` which is the time to delay in seconds. If you have a delayed node it is also assumed that the `hidden` config is set to `true` and the `priority` config is set to 0.
