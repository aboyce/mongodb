## Logging

There are two logging facilities for tracking activities on the database.

### Process Log

Displays activity on the MongoDB instance, it collections activity into one of the following:

- `ACCESS`: messages related to access control, such as authentication
- `COMMAND`: messages related to database commands
- `CONTROL`: messages related to control activities, such as initialisation
- `FTDC`: messages related to the diagnostic data collection mechanism
- `GEO`: messages related to the parsing of geospatial shapes
- `INDEX`: messages related to indexing operations
- `NETWORK`: messages related to network activities, such as accepting connections
- `QUERY`: messages related to queries, include query planner activities
- `REPL`: messages related to replica sets, such as initial sync or heartbeats
- `REPL_HB`: messages related to replica set heartbeats
- `ROLLBACK`: messages related to rollback operations
- `SHARDING`: messages related to sharding operations

### Log Verbosity Levels

- `-1` means to inherit from the parent
- `0` the default verbosity, to include informational messages
- `1` to `5` increases the verbosity level to include debug messages

### Log Severity Levels

- `F` for fatal
- `E` for error
- `W` for warning
- `I` for informational (verbosity level 0)
- `D` for debug (verbosity level 1-5)
