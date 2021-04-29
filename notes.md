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