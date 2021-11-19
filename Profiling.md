## Profiling

Profilers are enabled at the database level, so the operations on each database are profiled separately. When enabled, the profiler will store operations for all operation on an database in a new collection called `system.profile` within the database.

This collection holds profiling data for:

- CRUD
- Administrative operations
- Configuration operations

It has three settings:

- `0` - the profiler is off and does not collect and data, this is the default level
- `1` - the profiler collects data for the operations that take longer than the value of `slowms` (100ms)
- `2` - the profiler collects data for all operations (this can add quite a log of load on the system)

`db.getProfilingLevel()` will show the current level, ensure you `use database` to select the database to check. To change the profiler level use `db.setProfilingLevel(1)`. You can also update the `slowms` value with `db.setProfilingLevel( 1, { slowms: 500 } )`.

To query the data you can:

```javascript
db.system.profile.find().pretty()
```
