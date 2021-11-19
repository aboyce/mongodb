## Shell

Create/switch to a new database

```
use DATABASE_NAME
```

Add new document

```
db.COLLECTION_NAME.save({ a: 1, b: 2 })
```

### Backup and Restoring

Depending on the use case, i.e. creating DB backups or exporting in a human readable format for another application you can export/import in JSON or BSON format.

#### JSON

`mongoexport` will export the data in JSON format and can be reimported with `mongoimport`.

#### BSON

`mongodump` will export the data in BSON format and can be reimported with `mongorestore`.

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
