Launching Solr in Solr Cloud mode

bin\solr.cmd start -e cloud

Indexing data

```
java -jar -Dc=test -Durl=http://localhost:8888/solr/update example\exampledocs\post.jar example\exampledocs\*
```

Searching

phrase "a+b"/"a%2Bb"

schema

managed schema and field guessing

creating field

```
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-field": {"name":"name", "type":"text_general", "multiValued":false, "stored":true}}' http://localhost:8983/solr/films/schema
```

or use admin UI

Creating copy field for searching without specifying field

```
curl -X POST -H 'Content-type:application/json' --data-binary '{"add-copy-field" : {"source":"*","dest":"_text_"}}' http://localhost:8983/solr/films/schema
```

or use admin UI

faceting

Faceting allows the search results to be arranged into subsets (or buckets, or categories), providing a count for each subset. There are several types of faceting: field values, numeric and date ranges, pivots (decision tree), and arbitrary query faceting

range faceting

date and price

```
curl 'http://localhost:8983/solr/films/select?q=*:*&rows=0'\
    '&facet=true'\
    '&facet.range=initial_release_date'\
    '&facet.range.start=NOW-20YEAR'\
    '&facet.range.end=NOW'\
    '&facet.range.gap=%2B1YEAR'
```

currently not supported in the admin page 

Another faceting type is pivot facets, also known as "decision trees", allowing two or more fields to be nested for all the various possible combinations. 

```
bin/solr delete -c films
```

indexing your own data

creating your own collection

```
./bin/solr create -c localDocs -s 2 -rf 2
./bin/post -c localDocs ~/Documents
```

 [Data Import Handler (DIH)](https://solr.apache.org/guide/8_8/uploading-structured-data-store-data-with-the-data-import-handler.html#uploading-structured-data-store-data-with-the-data-import-handler) to connect to databases, mail servers, ...

even if you index content in this tutorial more than once, it does not duplicate the results found. This is because the example Solr schema (a file named either `managed-schema` or `schema.xml`) specifies a `uniqueKey` field called `id`.

Whenever you POST commands to Solr to add a document with the same value for the `uniqueKey` as an existing document, it automatically replaces it for you.

`numDocs` and `maxDoc`

`numDocs` represents the number of searchable documents in the index

The `maxDoc` value may be larger as the `maxDoc` count includes logically deleted documents that have not yet been physically removed from the index. 

deleting data

```
bin/post -c localDocs -d "<delete><id>SP2514N</id></delete>"
bin/post -c localDocs -d "<delete><query>*:*</query></delete>"
```

spatial query

Solr has sophisticated geospatial support, including searching within a specified distance range of a given location (or within a bounding box), sorting by distance, or even boosting results by the distance.

#### Solr control script reference

Solr includes a script known as “bin/solr” that allows you to perform many common operations on your Solr installation or cluster.

You can start and stop Solr, create and delete collections or cores, perform operations on ZooKeeper and check the status of Solr and configured shards.

##### Starting and restart

The `start` command starts Solr. The `restart` command allows you to restart Solr while it is already running or if it has been stopped already.

```
bin/solr start [options]
bin/solr start -help
bin/solr restart [options]
bin/solr restart -help
```

When using the `restart` command, you must pass all of the parameters you initially passed when you started Solr.

-a/-D "" additional parameters for JVM

-c Start Solr in SolrCloud mode, which will also launch the embedded ZooKeeper instance included with Solr.

-d <dir>  Define a server directory, defaults to `server` (as in, `$SOLR_HOME/server`). (more common to use the same server directory for each instance and use a unique Solr home directory using the -s option)

-e <name> example configuration

-h <hostname> Start Solr with the defined hostname. If this is not specified, 'localhost' will be assumed.

```
bin/solr start -h search.mysolr.com
```

-m <memory>

-p <port> default:8983

-s <dir> Sets the `solr.solr.home` system property If set, the specified directory should contain a `solr.xml` file, unless `solr.xml` exists in ZooKeeper. The default value is `server/solr`.

-v bin/solr start -f -v INFO->Debug

-q Info->Warning

-z <zkhost> bin/solr start -c -z server1:2181,server2:2181



