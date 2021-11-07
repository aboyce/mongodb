## Data Modelling

When designing data models, always consider the application usage of the data; queries, updates, processing, as well as th inherent structure of the data itself.

### Flexible Schema

By default, MongoDB collections do not require their documents to have the same schema. The documents in a collection do not need to have the same set of fields and the data type for a field can differ across documents in a collection. To change the structure of documents (add new fields, remove existing fields, change the field values to a new type) you just update the documents to the new structure.

In practise, the documents in a collection share a similar structure and you can enforce document validation rules for a collection during update and insert operations.

### Document Structure

MongoDB allows related data to be embedded within a single document.

#### Embedded Data

Embedded documents capture relationships between data by storing related data in a single document. These denormalised data models allow applications to retrieve and manipulate related data in a single database operation. For most using MongoDB this denormalised data model is optimal.

In general, use embedded data models when:

- you have 'contains' relationships between entities
- you have one-to-many relationships between entities, with the many being contained in the one parent document

Embedding generally provides better performance for read operations, as well as the ability to request and retrieve related data in a single operation. It also makes it possible to update related data in a single atomic write operation.

Use dot notation to get to the embedded documents.

#### References

References store the relationships between data by including links or references from one document to another. Applications can resolve these references to access the related data, there are broadly normalised data models.

In general, use normalised data models when:

- embedding would result in duplication of data but would not provide sufficient read performance advantages to outweigh the implications of the duplication
- you need to represent more complex many-to-many relationships
- you need to model large hierarchical data sets

To join collections, MongoDB provides the aggregation stages `$lookup` and `$graphLookup`. It also provides reference to join data across collections.

#### Atomicity of Write Operations

A write operation is atomic on the level of a single document, even if the operation modifies multiple embedded documents within a single document.

If you require atomic operations you should ensure you use a denormalised data model with embedded data in a single document.

When a single write operation modifies multiple documents (for example using `db.collection.updateMany()`) the modification of each document is atomic, but the operation as a whole is not atomic.

When performing multi-document write operations, whether through a single write or multiple write operations, other operation may interleave.

For situations that require atomicity of reads and writes to multiple documents (in a single or multiple collections) you can use multi-document transactions. In most cases, multi-document transactions incurs a greater performance cost over single document writes. These should also not compensate for ineffective schema design.
