## Shell

Create/switch to a new database

```
use DATABASE_NAME
```

Add new document

```
db.COLLECTION_NAME.save({ a: 1, b: 2 })
```

### Server Tools

`mongostat` is a utility that provides statistics on a running `mongod` or `mongos` process.

### Backup and Restoring

Depending on the use case, i.e. creating DB backups or exporting in a human readable format for another application you can export/import in JSON or BSON format.

#### JSON

`mongoexport` will export the data in JSON (or CSV) format and can be reimported with `mongoimport`. By default it will just output to the `stout`, if you want to store the output as JSON you can provide a `o` flag with the json filename.

#### BSON

`mongodump` will export the data in BSON format and can be reimported with `mongorestore`. It will also provides a JSON metadata file about the exported collection. `mongorestore` has a `drop` flag which will drop the collection if it exists on the restore.

### Database Commands

Database commands provide the foundation for interacting with MongoDB, use `db.runCommand()` to run a database command.

### Shell Helpers

There are shell helpers to interact with specific parts of MongoDB.

- `db.<method>()` - interact with the database
- `db.<collection>.method()` - collection level operations
- `rs.<method>()` - replica set deployment and management
- `sh.<method>()` - shared cluster deployment and management

#### Database Helpers

```
db.createUser()
db.dropUser()
```

#### Collection Management

```
db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()
```

#### Database Management

```
db.dropDatabase()
db.createCollection()
```

#### Database Status

```
db.serverStatus()
```
