[![Build Status](https://travis-ci.org/elisajs/elisa.svg?branch=master)](https://travis-ci.org/elisajs/elisa)

API for accessing to databases.

*Proudly made with ♥ in Valencia, Spain, EU.*

# Elisa.js

**Elisa** is an API for connecting to databases.

Official site: [elisajs.org](http://elisajs.org).

Her features are:

- This is independent of the DBMS. A query using an *Elisa* driver must return
  the same result in any database. So, the developers can switch from a database
  to another without problems.
- This has the same API for the browser and *Node.js*. So, the developers can
  access to databases using the same code in the browser app and the Node.js app.
- This can work with key-value stores (*Redis*) and document collections (*CouchDB*, *PouchDB* and *MongoDB*).
  Proximately, *Elisa* is going to support SQL databases (*PostgreSQL*, *SQL Server*, *SQLite*, etc.).
- This has a synchronous API and other asynchronous.
- This is easy for learning.
- This is simple for using.

# Preview examples

## Asynchronous example

```
//import driver to use
const Driver = require("elisa-pouchdb").Driver;

//get driver to use
const driver = Driver.getDriver("PouchDB");

//document collection operations
driver.openConnection({host: "localhost", port: 5984, db: "testing"}, function(error, cx) {
  const bands = cx.db.getCollection("core.bands");

  //insert
  bands.insert([{name: "The National", year: 1999}, {name: "Echo and the Bunnymen", year: 1978}]);

  //update
  bands.update({name: "The National", {tags: {$add: "indie"}});

  //remove
  bands.remove({year: {$between: [1990, 2000]}});

  //find
  bands.find({tags: {$contain: "indie"}}, function(error, res) {
    for (let band of res.docs) console.log(band.id, band.name, band.year);
  });

  bands.findOne({name: "The National"}}, function(error, band) {
    if (band) console.log(band.id, band.name, band.year);
  });
});

//key-value store operations
driver.openConnection({host: "localhost", port: 5984, db: "testing"}, function(error, cx) {
  const bands = cx.db.getStore("core.bands");

  //insert
  bands.insert([{id: "The National", year: 1999}, {id: "Echo and the Bunnymen", year: 1978}]);

  //update
  bands.update({id: "The National"}, {tags: {$add: "indie"}});

  //remove
  bands.remove({id: "The National"});

  //find
  bands.find({id: "The National"}, function(error, res) {
    for (let band of res.docs) console.log(band.id, band.name, band.year);
  });
});
```

## Synchronous example

```
//import driver to use
const Driver = require("elisa-pouchdb").Driver;

//get driver to use
const driver = Driver.getDriver("PouchDB");

//open connection
const cx = driver.openConnection({type: "sync"}, {host: "localhost", port: 5984, db: "testing"});

//document collection operations
var bands = cx.db.getCollection("core.bands");

//insert
bands.insert([{name: "The National", year: 1999}, {name: "Echo and the Bunnymen", year: 1978}]);

//update
bands.update({name: "The National", {tags: {$add: "indie"}});

//remove
bands.remove({year: {$between: [1990, 2000]}});

//find
var res = bands.find({tags: {$contain: "indie"}});
for (let band of res.docs) console.log(band.id, band.name, band.year);

var band = bands.findOne({name: "The National"}});
if (band) console.log(band.id, band.name, band.year);

//key-value store operations
var bands = cx.db.getStore("core.bands");

//insert
bands.insert([{id: "The National", year: 1999}, {id: "Echo and the Bunnymen", year: 1978}]);

//update
bands.update({id: "The National"}, {tags: {$add: "indie"}});

//remove
bands.remove({id: "The National"});

//find
var res = bands.find({id: "The National"});
for (let band of res.docs) console.log(band.id, band.name, band.year);
```

# Drivers

A **driver** is a component enabling a browser/Node.js app to interact with databases.
The *Elisa* driver is an analogous concept to the JDBC/ODBC/ADO.NET drivers.

To connect a database, firstly we have to load its driver. We must use the
`Driver` class, concretely its `getDriver()` method:

```
getDriver(name : string) : Driver
```

Next, we show an example to get the *PouchDB* driver:

```
//load PouchDB driver
const Driver = require("elisa-pouchdb").Driver;

//get the PouchDB driver
var driver = Driver.getDriver("PouchDB");
```

First, we have to get the `Driver` class of the driver package. Next, we have
to get the driver to use by its name.

# Connections

A **connection** is an object to access to a concrete database. We can use the
`Driver.createConnection()` method:

```
//async connection
createConnection(opts : object) : Connection
createConnection({type: "async"}, opts : object) : Connection

//sync connection
createConnection({type: "sync"}, opts : object) : Connection
```

A **synchronous connection** performs all operations, `find()`, `insert()`, `update()`, etc.,
synchronously. An **asynchronous connection**, asynchronously. By default, the connections are
asynchronous.

Example:

```
//async
var cx = driver.createConnection({host: "localhost", port: 5984, db: "testing"});

//sync
var cx = driver.createConnection({type: "sync"}, {host: "localhost", port: 5984, db: "testing"});
```

The connection options are driver dependent.

## Opening a connection

Once we have the connection object, we have to open it, using its `open()` method:

```
//sync connection
open()

//async connection
open(callback : function(error))
```

An example:

```
//sync connection
cx.open();

//async connection
cx.open(function(error) {
  ...
});
```

We can create the connection and open it in one step with the `Driver.openConnection()` method:

```
//sync connection
openConnection({type: "sync"}, opts : object) : Connection

//async connection
openConnection(opts : object, callback : function(error, cx))
```

Example:

```
//sync connection
var cx = driver.openConnection({type: "sync"}, {host: "localhost", port: 5984, db: "testing"});

//async connection
driver.openConnection({host: "localhost", port: 5984, db: "testing"}, function(error, cx) {
  ...
});
```

## Closing connections

To close a connection, we can use its `close()` method:

```
//sync connection
close()

//async connection
close(callback ?: function(error))
```

## Connection status

To know if a connection is opened or closed, we can use the following properties:

```
opened : boolean
closed : boolean
```

To know if the connection is opened and currently is connected, the `connected()` method:

```
//sync connection
connected() : boolean

//async connection
connected(callback : function(error, connected))
```

The connection can be lost if the server, for example, is shut down. In that case,
the `opened` property will be `true`, but `connected()` will return `false`.

# Server

The **server** is an object to represent the database server. Its object is got
from the `Connection.server` property:

```
server : Server
```

This object contains the following properties:

- `host` (string). The server host.
- `port` (number). The server port.
- `version` (string). The server version.

# Databases

The **database** is an object to represent a database in the database server.
The current database is accessed using the `Connection.db` property:

```
db : Database
```

Example:

```
console.log(cx.db.name);
```

# Schemas

A **schema** is a logical container of database objects. If the DBMS doesn't support
this concept, the driver must implement it. So for example, *PouchDB* doesn't support
the schema concept, but its *Elisa* driver does it.

To get a schema object, we can use the following methods of the database object:

```
//sync connection
getSchema(name : string) : Schema
findSchema(name : string) : Schema

//async connection
getSchema(name : string) : Schema
findSchema(name : string, callback : function(error, sch))
```

The first method, `getSchema()`, doesn't check whether the schema exists.
The second one, `findSchema()`, does it.

Example:

```
//sync connection
var hr = db.getSchema("hr");
var hr = db.findSchema("hr");

//async connection
var hr = db.getSchema("hr");
db.findSchema("hr", function(error, sch) {
  ...  
});
```

If we only need to know if a schema exists, we can use the `Database.hasSchema()` method:

```
//sync connection
hasSchema(name : string) : boolean

//async connection
hasSchema(name : string, callback : function(error, exists))
```

# Stores

A **store** is a data container where every data has a key and a value. If the DBMS doesn't support
this concept natively, the driver should do it. So for example, *PouchDB* and *MongoDB* don't support
the stores, but their drivers do it.

To get a store object, we must use the following methods:

```
//sync connection
db.getStore(schema : string, store : string) : Store
db.getStore(qn : string) : Store
db.findStore(schema : string, store : string) : Store
db.findStore(qn : string) : Store

schema.getStore(name : string) : Store
schema.findStore(name : string) : Store

//async connection
db.getStore(schema : string, store : string) : Store
db.getStore(qn : string) : Store
db.findStore(schema : string, store : string, callback : function(error, store))
db.findStore(qn : string, callback : function(error, store))

schema.getStore(name : string) : Store
schema.findStore(name : string, callback : function(error, store))
```

The `getStore()` method doesn't check whether the store exists. `findStore()` does it.

Examples:

```
//sync connection
var emp = db.getStore("hr.Employee");
var emp = db.findStore("hr.Employee");

//async connection
var emp = db.getStore("hr.Employee");
db.findStore("hr.Employee", function(error, emp) {
  ...
});
```

To know if a store exists, we can use the `hasStore()` method:

```
//sync connection
Database.hasStore(schema : string, store : string) : boolean
Database.hasStore(qn : string) : boolean
Schema.hasStore(name : string) : boolean

//async connection
Database.hasStore(schema : string, store : string, callback : function(error, exists))
Database.hasStore(qn : string, callback : function(error, exists))
Schema.hasStore(name : string, callback : function(error, exists))
```

## FQN and QN

The **full qualified name** (FQN) is the full name to a store object: `db.schema.store`. The
**qualified name** (QN) is the partial name: `schema.store`.

## Inserting items

To insert a value, we use the `insert()` method:

```
//sync connection
insert(value : object)
insert(values : object[])

//async connection
insert(value : object, callback ?: function(error))
insert(values : object[], callback ?: function(error))
```

Every value must be an object where its `id` property is the key. Example:

```
//sync connection
bands.insert({id: "The National", year: 1999});
bands.insert([
  {id: "The National", year: 1999},
  {id: "Echo and the Bunnymen", year: 1978}
]);

//async connection
bands.insert({id: "The National", year: 1999}, function(error) {
  ...
});

bands.insert([
  {id: "The National", year: 1999},
  {id: "Echo and the Bunnymen", year: 1978}
], function(error) {
  ...
});
```

## Finding items

To find an item, we can use the `find()` method or the `findOne()` method:

```
//sync connection
findOne({id: "key"}) : object
find({id: "key"}) : object

//async connection
findOne({id: "key"}, callback : function(error, item))
find({id: "key"}, callback : function(error, item))
```

Example:

```
//sync connection
var band = bands.findOne({id : "The National"});

//async connection
bands.findOne({id: "The National"}, function(error, item) {
  ...
});
```

If we want to get all the items from a store, we must use its `findAll()` method:

```
//sync connection
findAll() : Result

//async connection
findAll(callback : function(error, result))
```

The result is an object that contains:

- `length` (number). The number of items.
- `docs` (object[]). The documents.
- `rows` (object[]). Alias of `docs`.

Example:

```
//sync connection
var res = bands.findAll();
for (let band of res.docs) console.log(band.id, band.year);

bands.findAll(function(error, res) {
  if (!error) {
    for (let band of res.docs) console.log(band.id, band.year);
  }
});
```

If we need to know if a key exists, be able to use the `Store.hasId()` method:

```
//sync connection
hasId("id") : boolean

//async connection
hasId("id", callback : function(error, exists))
```

To get the number of items, we will use the `Store.count()` method:

```
//sync connection
count() : number

//async connection
count(callback : function(error, count))
```

## Updating values

To update a value property, we can use the `update()` method:

```
//sync connection
update({id: "key"}, update : object)

//async connection
update({id: "key"}, update : object, callback ?: function(error))
```

The first parameter must be an object with the `id` property, that is, the key to update.
The second parameter must contain the change operators. Example:

```
//sync connection
bands.update({id: "The National"}, {year: 1999, origin: "Cincinnati, Ohio"});

//async connection
bands.update({id: "The National"}, {year: 1999, origin: "Cincinnati, Ohio"}, function(error) {
  ...
});
```

### Update operators

The update object is similar to MongoDB and to Mango:

- `{field: value}`. Set the new value.
- `{field: {$set: value}}`. Set the new value.
- `{field: {$inc: value}}`. Increment the value.
- `{field: {$dec: value}}`. Decrement the value.
- `{field: {$mul: value}}`. Multiply the value.
- `{field: {$div: value}}`. Divide the value.
- `{field: {$add: value}}`. Add an item to an array if the value doesn't exist.
- `{field: {$push: value}}`. Add an item at the end of an array.
- `{field: {$concat: value}}`. Concatenate a string at the end of another.
- `{field: {$pop: value}}`. Remove the given number of items.

The format is very simple:

```
field: {$operator: value}
```

Examples of operations:

```
//year += 2
{year: {$inc: 2}}

//add to set
{tags: {$add: "indie"}}
```

## Removing items

To remove an item, we must use the `remove()` method:

```
//sync connection
remove({id: "key"})

//async connection
remove({id: "key"}, callback ?: function(error))
```

Example:

```
//sync connection
bands.remove({id: "The National"});

//async connection
bands.remove({id: "The National"}, function(error) {
  ...
});
```

# Collections

A **collection** is a document container. If the DBMS doesn't support this
concept natively, the driver should do it. For example, *PouchDB* and *CouchDB*
are document databases, but right now these don't support the collection concept.
But its *Elisa* drivers do it.

To get a collection object, we must use the following methods:

```
//sync connection
db.getCollection(schema : string, coll : string) : Collection
db.getCollection(qn : string) : Collection
db.findCollection(schema : string, coll : string) : Collection
db.findCollection(qn : string) : Collection

schema.getCollection(name : string) : Collection
schema.findCollection(name : string) : Collection

//async connection
db.getCollection(schema : string, coll : string) : Collection
db.getCollection(qn : string) : Collection
db.findCollection(schema : string, coll : string, callback : function(error, coll))
db.findCollection(qn : string, callback : function(error, coll))

schema.getCollection(name : string) : Collection
schema.findCollection(name : string, callback : function(error, coll))
```

The `getCollection()` method doesn't check whether the collection exists. `findCollection()` does it.

Examples:

```
//sync connection
var emp = db.getCollection("hr.Employee");
var emp = db.findCollection("hr.Employee");

//async connection
var emp = db.getCollection("hr.Employee");
db.findCollection("hr.Employee", function(error, emp) {
  ...
});
```

To know if a collection exists, we can use the `hasCollection()` method:

```
//sync connection
Database.hasCollection(schema : string, coll : string) : boolean
Database.hasCollection(qn : string) : boolean
Schema.hasCollection(name : string) : boolean

//async connection
Database.hasCollection(schema : string, coll : string, callback : function(error, exists))
Database.hasCollection(qn : string, callback : function(error, exists))
Schema.hasCollection(name : string, callback : function(error, exists))
```

## Inserting documents

To insert a document, we use the `insert()` method:

```
//sync connection
insert(doc : object)
insert(docs : object[])

//async connection
insert(doc : object, callback ?: function(error))
insert(docs : object[], callback ?: function(error))
```

The `doc` must be an object where its `id` property is the key. If the `id` isn't
set, the driver must do it.

Example:

```
//sync connection
bands.insert({name: "The National", year: 1999});

//async connection
bands.insert({name: "The National", year: 1999}, function(error) {
  ...
});
```

## Finding documents

To find documents, we can use:

```
//sync connection
findOne(filter : object) : object
find(filter : object) : Result
findAll() : Result

//async connection
findOne(filter : object, callback : function(error, doc))
find(filter : object, callback : function(error, result))
findAll(callback : function(error, result))
```

Example:

```
//sync connection
var band = bands.findOne({name: "The National"});
console.log(band.name, band.year);

var res = bands.find({tags: {$contain: "indie"}});
for (let band of res.docs) console.log(band.name, band.year);

//async connection
bands.findOne({name: "The National"}, function(error, doc) {
  console.log(doc.name, doc.year);
});

bands.find({tags: {$contain: "indie"}}, function(error, result) {
  if (!error) {
    for (let band of result.docs) console.log(band.name, band.year);
  }
});
```

If we need to know if a key exists, be able to use the `Collection.hasId()` method:

```
//sync connection
hasId("id") : boolean

//async connection
hasId("id", callback : function(error, exists))
```

To get the number of items, we will use the `Collection.count()` method:

```
//sync connection
count() : number

//async connection
count(callback : function(error, count))
```

### Filter object

The filter object sets a condition to restrict the documents to return. The operators are:

- `{field: value}`. The field is equal to another.
- `{field: {$eq: value}}`. The field is equal to another.
- `{field: {$ne: value}}`. The field isn't equal to another. Alias: `$neq`.
- `{field: {$lt: value}}`. The field is less than another.
- `{field: {$lte: value}}`. The field is less than or equal to another. Alias: `$le`.
- `{field: {$gt: value}}`. The field is greater than another.
- `{field: {$gte: value}}`. The field is greater than or equal to another. Alias: `$ge`.
- `{field: {$between: [start, end]}}`. The field value is between two.
- `{field: {$nbetween: [start, end]}}`. The field value is not between two. Alias: `$notBetween`.
- `{field: {$like: value}}`. The field matches a pattern.
- `{field: {$nlike: value}}`. The field doesn't match a pattern. Alias: `$notLike`.
- `{field: {$contain: value}}`. The field contains a value.
- `{field: {$ncontain: value}}`. The field doesn't contain a value. Alias: `$notContain`.

The format is very simple:

```
field: {$operator: value}
```

The same format is used in the filter operators and in the update operators.
MongoDB doesn't so.

Example:

```
res = bands.find({year: {$between: [1990, 2000]}, tags: {$contain: "indie"}});
```

## Updating documents

To update documents, we can use the `update()` method:

```
//sync connection
update(filter : object, update : object)

//async connection
update(filter : object, update : object, callback ?: function(error))
```

Example:

```
//sync connection
bands.update({name: "The National"}, {year: 1999, origin: "Cincinnati, Ohio"});

//async connection
bands.update({name: "The National"}, {year: 1999, origin: "Cincinnati, Ohio"}, function(error) {
  ...
});
```

## Removing documents

To remove an item, we must use the `remove()` method:

```
//sync connection
remove(filter : object)

//async connection
remove(filter : object, callback ?: function(error))
```

Example:

```
//sync connection
bands.remove({name: "The National"});

//async connection
bands.remove({name: "The National"}, function(error) {
  ...
});
```

## Collection query

The `Collection.q()` method returns an object to allow to perform queries with projections,
filters, sorts, etc.:

```
//sync/async connection
q() : CollectionQuery
query() : CollectionQuery
```

Example:

```
q = bands.q();
```

### Projections

We can restrict the fields to return with the `project()` method:

```
//sync/async connection
project(field : string) : CollectionQuery
project(...fields : string[]) : CollectionQuery
project(fields : object) : CollectionQuery
```

Examples:

```
q.project("name", "year");
q.project({name: "nm", year: "yr"});  //name as nm and year as yr
```

### Filter

To set the filter, we can use the `filter()` method:

```
//sync/async connection
filter(filter : object) : CollectionQuery
```

Example:

```
q.project("name").filter({year: 1990});
```

Note. All the query methods return the same query. So we can chain several
operations in the same expression.

### Limit

To limit the number of documents returned by the query:

```
//sync/async connection
limit(count : number) : CollectionQuery
```

Example:

```
q.project("name").filter({year: 1990}).limit(10);
```

### Skip the first documents

To skip the first documents:

```
//sync/async connection
offset(count : number) : Collection
```

Example:

```
q.project("name").filter({year: 1990}).offset(20).limit(10)
```

### Sort

To sort the result, we can use the `sort()` method:

```
//sync/async connection
sort(field : string) : QueryCollection
sort(...fields : string[]) : QueryCollection
sort(fields : object) : QueryCollection
```

Example:

```
q.filter({year: 1990}).sort(year);
q.filter({year: 1990}).sort({year: "ASC"});
```

If an object is specified, the values can be `ASC` or `DESC`.

### Run

To run the query, we can use the `find()` method or the `run()` method:

```
//sync connection
find(filter ?: object) : Result
run() : Result

//async connection
find(filter ?: object, callback : function(error, result))
run(function(error, result))
```

The `find()` method allows to set the filter, while the `run()` doesn't.

Example:

```
//sync connection
var res = q.project("name").find({tags: {$contain: "indie"}});
for (let band of res.docs) console.log(band.id, band.name, band.year);

//async connection
q.project("name").find({tags: {$contain: "indie"}}, function(error, res) {
  if (!error) {
    for (let band of res.docs) console.log(band.id, band.name, band.year);
  }
});
```