---
title: Listing MongoDB collections by size
date: 2013-09-10 11:37 -04:00
tags: mongo, mongodb, programming
---

In a MongoDB shell, the `db.stats()` command shows you the amount of space occupied by the selected database.

    > db.stats()
    {
      "db" : "my-db",
      "collections" : 242,
      "dataSize" : 7167367172,
      "storageSize" : 7885074432,
       ...
      "fileSize" : 14958985216,
      "nsSizeMB" : 16,
      "ok" : 1
    }


The `db.collection_name.stats()` command returns similar metrics for an individual collection, but there's no convenient way to see how the overall metrics are broken down by collection.

### My Solution

This javascript lists collections in descending order of size and can be executed directly in the mongo shell:

    var collectionNames = db.getCollectionNames(), stats = [];
    collectionNames.forEach(function (n) { stats.push(db[n].stats()); });
    stats = stats.sort(function(a, b) { return b['size'] - a['size']; });
    for (var c in stats) { print(stats[c]['ns'] + ": " + stats[c]['size'] + " (" + stats[c]['storageSize'] + ")"); }

Or, see the gist: [mongodb_collection_sizes.js](https://gist.github.com/joeyAghion/6511184).

Example output (first metric is `size`, second is `storageSize`):

    my-db.foos: 5568196496 (7455608832)
    my-db.bars: 716999376 (929259520)
    ...
    my-db.bazs: 0 (8192)
