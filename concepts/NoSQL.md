## NoSQL

### Terminology

SQl to MongoDB

- database = database
- table = collection
- row = document or BSON document
- column = field
- index = index
- table joins = `$lookup`
- primary key = primary key
- aggregation = aggregation pipeline

### When to Choose MongoDB over RDBS

MongoDB has a lower barrier to entry, developers with programming experience can easily look at collections and see what data there is and add/remove as required. To get the most out of a relational database you must first understand normalisation, referential integrity and relation database design.

This flexibility is useful as long as you do not need all the safety features offered by relational systems.

Relational databases may be a better choice for applications that require very complex but rigid data structures and database schemas across a large number of tables. An example is banking that requires very strong referential integrity and transactional guarantees to maintain exact point-in-time integrity of data. MongoDB does though support ACID properties of transactions.

A key benefit of MongoDB is that it is extremely easy to scale, sharding distributes the data across many servers and replica sets ensure that data is stored in multiple servers for high availability. Scaling typical SQL servers is more limited, usually reduced to vertical scaling and then read replicas.

MongoDB is optimised for write performance (prioritising speed over transaction safety where SQL data needs to be inserted row by row) and because the data is generally located in a single document, any joining logic is generally not required (which is usually quicker in SQL databases as that is required more frequently).

MySQL is faster at selecting a large number of records, whilst MongoDB is significantly faster at inserting or updating a large number of records.

### Four families of NoSQL databases

#### Key Value

The primary key gets you a blob of information. The system can know which server has that key and can send the request to just that server.

#### Graph

Useful for when relationships occur within tables rather than across tables.

#### Column (Wide Column Stores)

Acknowledge that modern data is polymorphic and that similar object can require different columns/fields/attributes.

#### Document

Polymorphic data structures where the relationships are more obvious using embedded arrays and documents. Easy amd natural representation and no complex mapping between application data and how it is stored in the database.
