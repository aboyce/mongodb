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
