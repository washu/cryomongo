# cryomongo

### A pure Crystal MongoDB driver.

A dependencies-free MongoDB driver written in pure Crystal.

**Compatible MongoDB versions: 3.6+**

### Work in progress!

![Working…](https://media.giphy.com/media/o0vwzuFwCGAFO/giphy.gif)

**Implemented**

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

**Next**

- https://github.com/mongodb/specifications/tree/master/source/causal-consistency
- https://github.com/mongodb/specifications/blob/master/source/sessions/driver-sessions.rst
- https://github.com/mongodb/specifications/blob/master/source/retryable-reads/retryable-reads.rst
- https://github.com/mongodb/specifications/blob/master/source/retryable-writes/retryable-writes.rst
- https://github.com/mongodb/specifications/tree/master/source/transactions
- https://github.com/mongodb/specifications/tree/master/source/command-monitoring
- https://github.com/mongodb/specifications/tree/master/source/compression

## Installation

1. Add the dependency to your `shard.yml`:

```yaml
dependencies:
  mongo:
    github: elbywan/cryomongo
```

2. Run `shards install`

## Usage

```crystal
require "cryomongo"

client = Mongo::Client.new("mongodb://localhost:27017")
database = client["database_name"]
collection = database["collection_name"]

collection.insert_one({ one: 1 })
collection.replace_one({ one: 1 }, { two: 2 })
bson = collection.find_one({ two: 2 })
puts bson.not_nil!.["two"] # => 2
collection.delete_one({ two: 2 })
puts collection.count_documents # => 0
```

TODO: Write usage instructions here

## Development

TODO: Write development instructions here

## Contributing

1. Fork it (<https://github.com/elbywan/cryomongo/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors

- [elbywan](https://github.com/elbywan) - creator and maintainer
