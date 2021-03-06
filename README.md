<div align="center">
	<img src="icon.svg" width="128" height="128" />
	<h1>cryomongo</h1>
  <h3>A MongoDB driver written in pure Crystal.</h3>
  <a href="https://travis-ci.org/elbywan/cryomongo"><img alt="travis-badge" src="https://travis-ci.org/elbywan/cryomongo.svg?branch=master"></a>
  <a href="https://github.com/elbywan/cryomongo/tags"><img alt="GitHub tag (latest SemVer)" src="https://img.shields.io/github/v/tag/elbywan/cryomongo"></a>
  <a href="https://github.com/elbywan/cryomongo/blob/master/LICENSE"><img alt="GitHub" src="https://img.shields.io/github/license/elbywan/cryomongo"></a>
</div>

<hr/>

#### Cryomongo is a high-performance MongoDB driver written in pure Crystal. (i.e. no C dependencies needed.)

*Compatible with MongoDB 3.6+. Tested against: 4.0 and 4.2.*

**⚠️ BETA state.**

> If you are looking for a higher-level object-document mapper library, you might want to check out the [`moongoon`](https://github.com/elbywan/moongoon) shard.

## Installation

1. Add the dependency to your `shard.yml`:

```yaml
dependencies:
  cryomongo:
    github: elbywan/cryomongo
```

2. Run `shards install`

## Usage

### Minimal working example

```crystal
require "cryomongo"

# Create a Mongo client, using a standard mongodb connection string.
client = Mongo::Client.new # defaults to: "mongodb://localhost:27017"

# Get database and collection.
database = client["database_name"]
collection = database["collection_name"]

# Perform crud operations.
collection.insert_one({ one: 1 })
collection.replace_one({ one: 1 }, { two: 2 })
bson = collection.find_one({ two: 2 })
puts bson.not_nil!.["two"] # => 2
collection.delete_one({ two: 2 })
puts collection.count_documents # => 0
```

## Features

- **[CRUD operations](https://docs.mongodb.com/manual/crud/index.html)**
- **[Aggregation](https://docs.mongodb.com/manual/aggregation/) (except: Map-Reduce)**
- **[Bulk](https://docs.mongodb.com/manual/reference/method/Bulk/index.html)**
- **[Read](https://docs.mongodb.com/manual/reference/read-concern/index.html) and [Write](https://docs.mongodb.com/manual/reference/write-concern/) Concerns**
- **[Read Preference](https://docs.mongodb.com/manual/core/read-preference/index.html)**
- **[Authentication](https://docs.mongodb.com/manual/core/authentication/index.html) (only: SCRAM mechanisms)**
- **[TLS encryption](https://docs.mongodb.com/manual/core/security-transport-encryption/)**
- **[Indexes](https://docs.mongodb.com/manual/indexes/index.html)**
- **[GridFS](https://docs.mongodb.com/manual/core/gridfs/index.html)**
- **[Change Streams](https://docs.mongodb.com/manual/changeStreams/index.html)**
- **[Admin/Diagnostic commands](https://elbywan.github.io/cryomongo/Mongo/Commands.html)**
- **[Tailable and Awaitable cursors](https://docs.mongodb.com/manual/core/tailable-cursors/index.html)**
- **[Collation](https://docs.mongodb.com/manual/reference/collation/index.html)**
- **Standalone, [Sharded](https://docs.mongodb.com/manual/sharding/) or [ReplicaSet](https://docs.mongodb.com/manual/replication/) topologies**

## Conventions

- Methods and arguments names are in **snake case**.
- Object arguments can usually be passed as a **[NamedTuple](https://crystal-lang.org/api/NamedTuple.html)**, **[Hash](https://crystal-lang.org/api/Hash.html)**, **[BSON::Serializable](https://github.com/elbywan/bson.cr#serialization)** or a **[BSON](https://elbywan.github.io/bson.cr/BSON.html)** instance.

## Documentation

**The generated API documentation is available [here](https://elbywan.github.io/cryomongo/Mongo.html).**

### Connection

```crystal
require "cryomongo"

# Mongo::Client is the root object for interacting with a MongoDB deployment.
# It is responsible for monitoring the cluster, routing the requests and managing the socket pools.

# A client can be instantiated using a standard mongodb connection string.

# Client options can be passed as query parameters…
client = Mongo::Client.new("mongodb://address:port/database?appname=MyApp")
# …or with a Mongo::Options instance…
options = Mongo::Options.new(appname: "MyApp")
client = Mongo::Client.new("mongodb://address:port/database", options)
# …or both.

# Instantiate objects to interact with a specific database or a collection.
database   = client["database_name"]
collection = database["collection_name"]

# The overwhelming majority of programs should use a single client and should not bother with closing clients.
# Otherwise, to free the underlying resources a client must be manually closed.
client.close
```

```crystal
# To enable SSL/TLS, use the `tls` option, alongside the `tlsCAFile` and `tlsCertificateKeyFile` options.
uri = "mongodb://localhost:27017/?tls=true&tlsCAFile=./ca.crt&tlsCertificateKeyFile=./client.pem"
ssl_client = Mongo::Client.new uri
```

**Links**

- [Mongo::Client](https://elbywan.github.io/cryomongo/Mongo/Client.html)
- [Mongo::Options](https://elbywan.github.io/cryomongo/Mongo/Options.html)

### Authentication

*Cryomongo only supports the SCRAM-SHA1 and SCRAM-SHA256 authentication methods without SASLprep.*

```crystal
require "cryomongo"

# To use authentication, specify a username and password when passing an URI to the client constructor.
# Authentication methods depend on the server configuration and on the value of the `authMechanism` query parameter.
client = Mongo::Client.new("mongodb://username:password@localhost:27017")
```

### Basic operations

```crystal
require "cryomongo"

client = Mongo::Client.new

# Most CRUD operations are performed at collection-level.
collection = client["database_name"]["collection_name"]

# The examples below are very basic, but the methods can accept all the options documented in the MongoDB manual.

## Create

# Insert a single document
collection.insert_one({key: "value"})
# Insert multiple documents
collection.insert_many((1..100).map{|i| { count: i }}.to_a)

## Read

# Find a single document
document = collection.find_one({ _id: BSON::ObjectId.new("5eed35600000000000000000") })
document.try { |d| puts d.to_json }
# Find multiple documents.
cursor = collection.find({ qty: { "$gt": 4 }})
elements = cursor.to_a # cursor is an Iterable(BSON)

## Update

# Replace a single document.
collection.replace_one({ name: "John" }, { name: "Jane" })
# Update a single document.
collection.update_one({ name: "John" }, { "$set": { name: "Jane" }})
# Update multiple documents
collection.update_many({ name: { "$in": ["John", "Jane"] }}, { "$set": { name: "Jules" }})
# Find one document and replace it
document = collection.find_one_and_replace({ name: "John" }, { name: "Jane" })
puts document.try &.["name"]
# Find one document and update it
document = collection.find_one_and_update({ name: "John" }, { "$set": { name: "Jane" }})
puts document.try &.["name"]

## Delete

# Delete one document
collection.delete_one({ age: 20 })
# Delete multiple documents
collection.delete_many({ age: { "$lt": 18 }})
# find_one_and_delete
document = collection.find_one_and_delete({ age: { "$lt": 18 }})
puts document.try &.["age"]

# Aggregate

# Perform an aggregation pipeline query
cursor = collection.aggregate([
  {"$match": { status: "available" }}
  {"$limit": 5},
])
cursor.try &.each { |bson| puts bson.to_json }

# Distinct collection values
values = collection.distinct(
  key: "field",
  filter: { age: { "$gt": 18 }}
)

# Documents count
counter = collection.count_documents({ age: { "$lt": 18 }})

# Estimated count
counter = collection.estimated_document_count
```

**Links**

- [Mongo::Collection](https://elbywan.github.io/cryomongo/Mongo/Collection.html)
- [Mongo::Database](https://elbywan.github.io/cryomongo/Mongo/Database.html)

### Bulk operations

```crystal
require "cryomongo"

client = Mongo::Client.new

# A Bulk object can be initialized by calling `.bulk` on a collection.
collection = client["database_name"]["collection_name"]
bulk = collection.bulk
# A bulk is ordered by default.
bulk.ordered? # => true

500.times { |idx|
  # Build the queries by calling bulk methods multiple times.
  bulk.insert_one({number: idx})
  bulk.delete_many({number: {"$lt": 450}})
  bulk.replace_one({ number: idx }, { number: idx + 1})
}

# Execute all the queries and return an aggregated result.
pp bulk.execute(write_concern: Mongo::WriteConcern.new(w: 1))
```

**Links**

- [Mongo::Bulk](https://elbywan.github.io/cryomongo/Mongo/Bulk.html)

### Indexes

```crystal
require "cryomongo"

client = Mongo::Client.new
collection = client["database_name"]["collection_name"]

# Create one index without options…
collection.create_index(
  keys: {
    "a":  1,
    "b":  -1,
  }
)
# or with options (snake_cased)…
collection.create_index(
  keys: {
    "a":  1,
    "b":  -1,
  },
  options: {
    unique: true
  }
)
# and optionally specify the name.
collection.create_index(
  keys: {
    "a":  1,
    "b":  -1,
  },
  options: {
    name: "index_name",
  }
)

# Follow the same rules to create multiple indexes with a single method call.
collection.create_indexes([
  {
    keys: { a: 1 }
  },
  {
    keys: { b: 2 }, options: { expire_after_seconds: 3600 }
  }
])
```

**Links**

- [Mongo::Collection](https://elbywan.github.io/cryomongo/Mongo/Collection.html)

### GridFS

```crystal
require "cryomongo"

client = Mongo::Client.new
database = client["database_name"]

# A GridFS bucket belong to a database.
gridfs = database.grid_fs

# Upload
file = File.new("file.txt")
id = gridfs.upload_from_stream("file.txt", file)
file.close

# Download
stream = IO::Memory.new
gridfs.download_to_stream(id, stream)
puts stream.rewind.gets_to_end

# Find
files = gridfs.find({
  length: {"$gte": 5000},
})
files.each { |file|
  puts file.filename
}

# Delete
gridfs.delete(id)

# And many more methods… (check the link below.)
```

**Links**

- [Mongo::GridFS::Bucket](https://elbywan.github.io/cryomongo/Mongo/GridFS/Bucket.html)

### Change streams

```crystal
require "cryomongo"

# Change streams can watch a client, database or collection for change.
# This code snippet will focus on watching a single collection.

client = Mongo::Client.new
collection = client["database_name"]["collection_name"]

spawn {
  cursor = collection.watch(
    [
      {"$match": {"operationType": "insert"}},
    ],
    max_await_time_ms: 10000
  )
  # cursor.of(BSON) converts fetched elements to the Mongo::ChangeStream::Document(BSON) type.
  cursor.of(BSON).each { |doc|
    puts doc.document_key
    puts doc.full_document.to_json
  }
}

100.times do |i|
  collection.insert_one({count: i})
end

sleep
```

**Links**

- [Mongo::ChangeStream::Cursor](https://elbywan.github.io/cryomongo/Mongo/ChangeStream/Cursor.html)
- [Mongo::ChangeStream::Document](https://elbywan.github.io/cryomongo/Mongo/ChangeStream/Document.html)

## Raw commands

```crystal
require "cryomongo"

# Commands can be run on a client, database or collection depending on the command target.

client = Mongo::Client.new

# Call the `.command` method to run a command against the server.
# The first argument is a `Mongo::Commands` sub-class, followed by the mandatory arguments
# and finally an *options* named tuple containing the optional parameters in snake_case.
result = client.command(Mongo::Commands::ServerStatus, options: {
  repl: 0
})
puts result.to_bson
```
**Links**

- [Mongo::Commands](https://elbywan.github.io/cryomongo/Mongo/Commands.html)
- [Mongo::Client#command](https://elbywan.github.io/cryomongo/Mongo/Client.html#command(commandcmd,write_concern:WriteConcern?=nil,read_concern:ReadConcern?=nil,read_preference:ReadPreference?=nil,server_description:SDAM::ServerDescription?=nil,ignore_errors=false,**args)-instance-method)
- [Mongo::Database#command](https://elbywan.github.io/cryomongo/Mongo/Database.html#command(operation,write_concern:WriteConcern?=nil,read_concern:ReadConcern?=nil,read_preference:ReadPreference?=nil,**args)-instance-method)
- [Mongo::Collection#command](https://elbywan.github.io/cryomongo/Mongo/Collection.html#command(operation,write_concern:WriteConcern?=nil,read_concern:ReadConcern?=nil,read_preference:ReadPreference?=nil,**args)-instance-method)

## Concerns and Preference

```crystal
require "cryomongo"

# Instantiate Read/Write Concerns and Preference
read_concern = Mongo::ReadConcern.new(level: "majority")
write_concern = Mongo::WriteConcern.new(w: 1, j: true)
read_preference = Mongo::ReadPreference.new(mode: "primary")

# They can be set at the client, database or client level…
client = Mongo::Client.new
database = client["database_name"]
collection = database["collection_name"]

client.read_concern = read_concern
database.write_concern = write_concern
collection.read_preference = read_preference

# …or by passing an extra argument when calling a method.
collection.find(
  filter: { key: "value" },
  read_concern:  Mongo::ReadConcern.new(level: "local"),
  read_preference: Mongo::ReadPreference.new(mode: "secondary")
)
```

**Links**

- [Mongo::ReadConcern](https://elbywan.github.io/cryomongo/Mongo/ReadConcern.html)
- [Mongo::WriteConcern](https://elbywan.github.io/cryomongo/Mongo/WriteConcern.html)
- [Mongo::ReadPreference](https://elbywan.github.io/cryomongo/Mongo/ReadPreference.html)

## Specifications

The goal is to to be compliant with most of the [official MongoDB set of specifications](https://github.com/mongodb/specifications).

**✅ Implemented**

 The following specifications are implemented:

- https://github.com/mongodb/specifications/tree/master/source/message
- https://github.com/mongodb/specifications/tree/master/source/crud
- https://github.com/mongodb/specifications/blob/master/source/find_getmore_killcursors_commands.rst
- https://github.com/mongodb/specifications/blob/master/source/driver-bulk-update.rst
- https://github.com/mongodb/specifications/blob/master/source/read-write-concern/read-write-concern.rst
- https://github.com/mongodb/specifications/blob/master/source/enumerate-collections.rst
- https://github.com/mongodb/specifications/blob/master/source/enumerate-databases.rst
- https://github.com/mongodb/specifications/blob/master/source/enumerate-indexes.rst
- https://github.com/mongodb/specifications/tree/master/source/connection-string
- https://github.com/mongodb/specifications/tree/master/source/uri-options (except validation)
- https://github.com/mongodb/specifications/tree/master/source/server-discovery-and-monitoring
- https://github.com/mongodb/specifications/blob/master/source/server-selection/server-selection.rst
- https://github.com/mongodb/specifications/blob/master/source/max-staleness/max-staleness.rst
- https://github.com/mongodb/specifications/tree/master/source/connection-monitoring-and-pooling (loosely - using the crystal-db pool)
- https://github.com/mongodb/specifications/blob/master/source/auth/auth.rst (SHA1 / SHA256 only - without SASLprep)
- https://github.com/mongodb/specifications/blob/master/source/index-management.rst (no IndexView fluid syntax)
- https://github.com/mongodb/specifications/tree/master/source/gridfs
- https://github.com/mongodb/specifications/tree/master/source/change-streams

**⏳Next**

The following specifications are to be implemented next:

- https://github.com/mongodb/specifications/tree/master/source/causal-consistency
- https://github.com/mongodb/specifications/blob/master/source/sessions/driver-sessions.rst
- https://github.com/mongodb/specifications/blob/master/source/retryable-reads/retryable-reads.rst
- https://github.com/mongodb/specifications/blob/master/source/retryable-writes/retryable-writes.rst
- https://github.com/mongodb/specifications/tree/master/source/transactions
- https://github.com/mongodb/specifications/tree/master/source/command-monitoring
- https://github.com/mongodb/specifications/tree/master/source/compression

## Contributing

1. Fork it (<https://github.com/elbywan/cryomongo/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [elbywan](https://github.com/elbywan) - creator and maintainer

## Credit

- Icon made by [Smashicons](https://www.flaticon.com/authors/smashicons) from [www.flaticon.com](www.flaticon.com).
