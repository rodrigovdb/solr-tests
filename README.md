# Using a container

Creating a volume
```
$ docker volume create solr
```

Creating a container
```
$ docker container create --hostname=solr-local --name=solr-local -v solr:/opt/solr/server/solr/mycores \
  -p 8983:8983 \
  -p 8984:8984 \
  -p 7574:7574 \
  solr
```

Starting the container
```
$ docker container start solr-local
```

Accessing the web interface: `http://localhost:8983`

Accessing via bash
```
$ docker exec -i -t solr-local bash
```

## Exercise #1

[Exercise 1](http://lucene.apache.org/solr/guide/7_1/solr-tutorial.html#exercise-1)
```
$ bin/solr start -e cloud
```
and chose:
* 2 (number of nodes)
* 8984 (port of node1)
* 7574 (port of node2)
* techproducts (name of new collection)
* 2 (number of chards)
* 2 (number of replicas)
* `sample_techproducts_configs` (configuration for the techproducts collection)

or just run with a single line
```
$ bin/solr create -c techproducts -s 2 -rf 2
```

Indexing techproducts
```
$ bin/post -c techproducts -p 8984 example/exampledocs/*
```

### Querying

Searching term "foundation" in all fields
```
$ curl "http://localhost:8983/solr/techproducts/select?q=foundation"
```

Searching for term "electronics" on "cat" fields
```
$ curl "http://localhost:8984/solr/techproducts/select?q=cat:electronics
```

Searching for a composite term "CAS latency"
```
$ curl "http://localhost:8984/solr/techproducts/select?q=CAS+latency"
```

Searching for documents that contain terms "electronics" and "music" (+elelectronics +music)
```
$ curl "http://localhost:8984/solr/techproducts/select?q=%2Belectronics%20%2Bmusic"
```

Searching for documents that contain the erm "electronicas" and  **DON'T** contain the term "music"(+electronics -music)
```
$ curl "http://localhost:8984/solr/techproducts/select?q=%2Belectronics%20-music"
```

More about search: http://lucene.apache.org/solr/guide/7_1/searching.html#searching

## Exercise #2
[Exercise 2](http://lucene.apache.org/solr/guide/7_1/solr-tutorial.html#exercise-2)

### Preparing and seeding

Create a new collection
```
$ bin/solr create -c films -s 2 -rf 2
```

Create the "names" Field
```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/films/schema
```

Create a "catchall" Copy Field
```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/films/schema
```

Indexin Sample Film Data using `json` (choose anyone)
```
$ bin/post -c films example/films/films.json
```
### Facets

One of Solr’s most popular features is faceting. Faceting allows the search results to be arranged into subsets (or buckets, or categories), providing a count for each subset.
There are several types of faceting: field values, numeric and date ranges, pivots (decision tree), and arbitrary query faceting.

1. Is required use `facet=true` in order to faceting
2. Use `rows=0` means that resultset doesn't includes docs, just facet counts

Facet counts from all documents
```
$ curl "http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=true&facet.field=genre_str"
```

Facet counts from all documents, but just with count > 200
```
$ curl "http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=true&facet.field=genre_str&facet.mincount=200"
```

Facet count with range of `initial_relase_date` between two values
```
$ curl 'http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=true&facet.range=initial_release_date&facet.range.start=NOW-20YEAR&facet.range.end=NOW&facet.range.gap=%2B1YEAR'
```

#### Pivot Facets

Another faceting type is pivot facets, also known as "decision trees", allowing two or more fields to be nested for all the various possible combinations.  Using the films data, pivot facets can be used to see how many of the films in the "Drama" category (the `genre_str` field) are directed by a director. Here’s how to get at the raw data for this scenario:

```
$ curl 'http://localhost:8983/solr/films/select?q=*:*&rows=0&facet=on&facet.pivot=genre_str,directed_by_str'
```

## Exercise #3

Accessing via bash
```
$ docker exec -i -t solr-local bash
```

### Building

Creating my own Core
```
$ bin/solr create -c songs -s 2 -rf 2
```

Creating fields
```
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/songs/schema
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"artist", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/songs/schema
$ curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/songs/schema
```

Indexin Songs Data
```
$ bin/post -c songs data/sample/songs.csv
```

### Searcing

Using Facet to count songs by key, just from Rock a Lenha
```
$ curl "http://localhost:8983/solr/songs/select?q=band:rock%20a%20lenha&rows=0&facet=true&facet.field=key"
```
