# Elasticsearch 5.x Cheatsheet

This is a collection of the most common commands I run while administering Elasticsearch databases. The variables shown between the open and closed tags, "<" and ">", should be replaced with a name you choose.

https://www.elastic.co/guide/en/elasticsearch/client/curator/5.x/command-line.html

I would suggest using my ELK vagrant project to use these commands:

https://github.com/apolloclark/elk
<br/><br/><br/>





## Notes

- default cluster name is "elasticsearch"
- default, each index has 5 primary shards, 1 replica
- better to over-provision shards initially, ~4 shards-per-node is good
- scale out nodes horizontally over time, until it's one-to-one with nodes
- Lucene can address up to 2 billion documents (2^31 - 128)
- mappings are essentially flattened into a single, global schema for the entire index
- doc values often reserve a fixed amount of disk space for every document
- shard data size should be balanced between nodes
- a single slow node will slow down the entire cluster
- version 5.x added "text" and "keyword" data types, replacing "string"
- lenient boolean is deprecated
- analyzer = filter, tokenizer, token filter
- max content legnth default = 100 MB
- max URL length default = 4 KB
- max header size default = 8 KB
<br/><br/><br/>





## Setup

##### clear the screen
```shell
(CTRL + L)
```

##### configure
```shell
/etc/elasticsearch/elasticsearch.yml
```

##### debug logs
```shell
/var/log/elasticsearch/elasticsearch.log
/var/log/elasticsearch/elasticsearch.log.*
/var/log/elasticsearch/elasticsearch_deprecation.log
/var/log/elasticsearch/elasticsearch_index_search_slowlog.log
/var/log/elasticsearch/elasticsearch_index_indexing_slowlog.log.log
```

##### formatting

https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html

```shell
# print output with indentation
?pretty=true

# print output in YAML format
?format=yaml

# print units with human readable units
?human=<true | false>

# print ouput flattened
?flat_settings=<true | false>

# explain more details about query execution
?explain

# include detailed debug output on failure
&error_trace=true

# filter to only receive specific fields
&filter_path=<field>
&filter_path=<field>,<field>
&filter_path=<field>.<field>
&filter_path=<field>.*
&filter_path=<field>.**
&filter_path=-<field>
```
<br/><br/><br/>





## Configuration

```shell
# disable selecting indexes within the request body
rest.action.multi.allow_explicit_index: false

# disable automatic index creation
action.auto_create_index: false

# set automatic index creation
action.auto_create_index: <+aaa*,-bbb*,+ccc*,-*>

# disable automatic mapping
index.mapper.dynamic: false

# configure number of shards to search
action.search.shard_count.limit

# set search timeout limit
search.default_search_timeout

# enable fast search cancellation
search.low_level_cancellation: true
```
<br/><br/><br/>





## Cluster

##### show version
```shell
curl -s -XGET 'http://127.0.0.1:9200?filter_path=version.number&pretty=false' | \
  awk -F'"' {'print $6'}
```

##### list cluster name
``` shell
curl -s -XGET 'http://127.0.0.1:9200?filter_path=cluster_name&pretty=false' | \
  awk -F'"' {'print $4'}
```

##### show cluster health
```shell
curl -XGET 'http://127.0.0.1:9200/_cluster/health?pretty'
```

##### show cluster state
```shell
curl -XGET 'http://127.0.0.1:9200/_cluster/state?pretty'
```

##### show cluster metric stats
```shell
curl -XGET 'http://127.0.0.1:9200/_cluster/stats?pretty'
```

##### show cluster settings
```shell
curl -XGET 'http://127.0.0.1:9200/_cluster/settings?pretty'
```

##### show all cluster settings
```shell
curl -XGET 'http://127.0.0.1:9200/_settings?pretty'
```
<br/><br/><br/>





## Indices

https://www.elastic.co/blog/index-vs-type

##### all indices
```shell
curl -XGET 'http://127.0.0.1:9200/_cat/indices?v'
```

##### all indices monitoring
```shell
curl -XGET 'http://127.0.0.1:9200/_all/_stats?pretty'
curl -XGET 'http://127.0.0.1:9200/_all/_segments?pretty'
curl -XGET 'http://127.0.0.1:9200/_all/_recovery?pretty&human'
```

##### single index monitoring
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_stats?pretty'
curl -XGET 'http://127.0.0.1:9200/<index>/_segments?pretty'
curl -XGET 'http://127.0.0.1:9200/<index>/_recovery?pretty&human'
```

##### indices management

```shell
POST /<index>/_cache/clear
POST /<index>/_refresh
POST /<index>/_flush
POST /<index>/_forcemerge
POST /<index>/_upgrade
GET /<index>/_upgrade?pretty&human
```

##### create index
```shell
curl -XPUT 'http://127.0.0.1:9200/<index>?pretty'

curl -XPUT 'http://127.0.0.1:9200/twitter?pretty'
```

##### one index, all details
```shell
curl -XGET 'http://127.0.0.1:9200/<index>?pretty'

curl -XGET 'http://127.0.0.1:9200/twitter?pretty'

curl -XGET 'http://127.0.0.1:9200/.kibana?pretty'
```

##### one index, alias
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_alias?pretty'

curl -XGET 'http://127.0.0.1:9200/twitter/_alias?pretty'

curl -XGET 'http://127.0.0.1:9200/.kibana/_alias?pretty'
```

##### one index, settings
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_settings?pretty'

curl -XGET 'http://127.0.0.1:9200/twitter/_settings?pretty'

curl -XGET 'http://127.0.0.1:9200/.kibana/_settings?pretty'
```

##### delete one index
```
curl -XDELETE 'http://127.0.0.1:9200/<index>?pretty'

curl -XDELETE 'http://127.0.0.1:9200/twitter?pretty'
```
<br/><br/><br/>





## Mappings

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html

##### field datatypes

https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html

```shell
logic = boolean
number = long, integer, short, byye, double, float, half_float, scaled_float
ranges = interger_range, float_range, long_range, double_range, date_range
string = text, keyword
dates = date
data = binary, array, object, nested
geo = geo_point, geo_shape
special = ip, completion, token_count, murmur3, attachment
```

##### list all indices, all mappings
```shell
curl -XGET 'http://127.0.0.1:9200/_mapping?pretty'
```

##### list one index, all mappings
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_mapping?pretty'

curl -XGET 'http://127.0.0.1:9200/twitter/_mapping?pretty'

curl -XGET 'http://127.0.0.1:9200/.kibana/_mapping?pretty'
```
<br/><br/><br/>





## Types

##### list all indices, all types
```shell
curl -s -XGET 'http://127.0.0.1:9200/_mapping' | \
  jq 'to_entries | .[] | {(.key): .value.mappings | keys}'
```

##### list one index, all types
```shell
curl -s -XGET 'http://127.0.0.1:9200/<index>/_mapping' | \
  jq '.[].mappings[] | keys'

curl -s -XGET 'http://127.0.0.1:9200/twitter/_mapping' | \
  jq '.[].mappings | keys'

curl -s -XGET 'http://127.0.0.1:9200/.kibana/_mapping' | \
  jq '.[].mappings | keys'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping' | \
  jq '.[].mappings | keys'

curl -s -XGET 'http://127.0.0.1:9200/metricbeat-*/_mapping' | \
  jq '.[].mappings | keys'

curl -s -XGET 'http://127.0.0.1:9200/heartbeat-*/_mapping' | \
  jq '.[].mappings | keys'
```

##### list one index, one type, all mappings
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_mapping/<type>'

curl -XGET 'http://127.0.0.1:9200/twitter/_mapping/tweet?pretty'

curl -XGET 'http://127.0.0.1:9200/.kibana/_mapping/index-pattern?pretty'

curl -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping/authlog?pretty'
```

##### list one index, all types, all mappings, collapsed to type
```shell
curl -s -XGET 'http://127.0.0.1:9200/<index>/_mapping' | \
  jq '. |= .[].mappings' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/.kibana/_mapping' | \
  jq '. |= .[].mappings' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping' | \
  jq '. |= .[].mappings' | \
  jq 'del(.[].properties.type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/metricbeat-*/_mapping' | \
  jq '. |= .[].mappings' | \
  jq 'del(.[].properties.type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/heartbeat-*/_mapping' | \
  jq '. |= .[].mappings' | \
  jq 'del(.[].properties.type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'
```

##### list one index, one type, all mappings, collapsed to type
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_mapping/<type>?pretty'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping/authlog' | \
  jq '.[].mappings[]' | \
  jq 'del(.[].type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping/syslog' | \
  jq '.[].mappings[]' | \
  jq 'del(.[].type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_mapping/misclog' | \
  jq '.[].mappings[]' | \
  jq 'del(.[].type)' | \
  jq 'walk( if type=="object" and has("properties") then . |= .properties else . end )' | \
  jq 'walk( if type=="object" and has("type") then . |= .type else . end )'
```
<br/><br/><br/>





## Nodes

##### list nodes
```shell
curl -XGET 'http://127.0.0.1:9200/_cat/nodes?v'
```

##### show node status
```shell
curl -XGET 'http://127.0.0.1:9200/_nodes/stats?pretty'
```
<br/><br/><br/>





## Shards

##### list shards
```shell
curl -XGET 'http://127.0.0.1:9200/_cat/shards?v'
```
<br/><br/><br/>





## Documents

##### insert data
```shell
curl -XPOST 'http://127.0.0.1:9200/twitter/tweet/?pretty' \
  -H 'Content-Type: application/json' \
-d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html

##### search all documents, all indices
```shell
curl -XGET 'http://127.0.0.1:9200/_all/_search?pretty=true&q=*:*'

curl -s -XGET 'http://127.0.0.1:9200/_all/_search?pretty=true&q=*:*' |\
  jq '.hits.hits'
```

##### search all documents, one index
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/_search?pretty=true&q=*:*'

curl -s -XGET 'http://127.0.0.1:9200/twitter/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'
  
curl -s -XGET 'http://127.0.0.1:9200/.kibana/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'

curl -s -XGET 'http://127.0.0.1:9200/metricbeat-*/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'

curl -s -XGET 'http://127.0.0.1:9200/heartbeat-*/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'
```

##### search all documents, one index, one type
```shell
curl -XGET 'http://127.0.0.1:9200/<index>/<type>/_search?pretty=true&q=*:*'

curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/authlog/_search?pretty=true&q=*:*' |\
  jq '.hits.hits[]._source'
```

##### search filebeat documents, count results

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html

```shell
curl -s -XGET -G 'http://127.0.0.1:9200/filebeat-*/_search' \
    -d 'q=message:install' \
    -d 'size=0' \
    -d 'terminate_after=1' \
    -d 'pretty' | \
  jq '.hits.total'
```

##### search filebeat documents, get only the message field

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-collapse.html

```shell
curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_search?pretty=true&q=message:install' | \
  jq '[.hits.hits[]._source.message]'

curl -s -XGET -G 'http://127.0.0.1:9200/filebeat-*/misclog/_search' \
    -d '_source=message' \
    -d 'filter_path=hits.hits._source.message' \
    -d 'pretty' | \
  jq '[.hits.hits[]._source.message]'
```

##### search filebeat documents, for installation, get only the message field

https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-collapse.html

```shell
curl -s -XGET 'http://127.0.0.1:9200/filebeat-*/_search?pretty=true&q=message:install' | \
  jq '[.hits.hits[]._source.message]'

curl -s -XGET -G 'http://127.0.0.1:9200/filebeat-*/misclog/_search' \
    -d 'q=message:install' \
    -d '_source=message' \
    -d 'filter_path=hits.hits._source.message' \
    -d 'pretty' | \
  jq '[.hits.hits[]._source.message]'
```


# Elasticsearch Cheatsheet - an overview of commonly used Elasticsearch API commands

# cat paths
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}

# Important Things
bin/elasticsearch                                                       # Start Elastic instance
curl -X GET  'http://localhost:9200/?pretty=true'                       # View instance metadata
curl -X POST 'http://localhost:9200/_shutdown'                          # Shutdown Elastic instance
curl -X GET 'http://localhost:9200/_cat?pretty=true'                    # List all admin methods
curl -X GET 'http://localhost:9200/_cat/indices?pretty=true'            # List all indices
curl -X GET 'http://localhost:9200/_cluster/health?pretty=true'         # View Cluster Health

# Index, Type Basics
curl -X GET  'http://localhost:9200/<index name>'                       # View specific index
curl -X POST 'http://localhost:9200/<index name>'                       # Create an index
curl -X DELETE 'http://localhost:9200/<index name>'                     # Delete an index

curl -X GET  'http://locahost:9200/<index name>/<type>/<id>'            # Retrieve a specific document
curl -X POST 'http://locahost:9200/<index name>/<type>/'                # Create a document
curl -X PUT  'http://locahost:9200/<index name>/<type>/<id>'            # Create/Update a specific document
curl -X DELETE 'http://localhost:9200/<index name>/<type>/<id>'         # Delete a specific document

curl -X GET  'http://localhost:9200/<index name>/_mappings'             # View mappings for index
curl -X GET  'http://localhost:9200/<index name>/_settings'             # View setting information for an index

curl -X GET  'http://localhost:9200/<index name>/<type>/_mappings'      # View mappings for an index type
curl -X GET  'http://localhost:9200/<index name>/<type>/_settings'      # View setting information for an index type

curl -X GET  'http://localhost:9200/<index name>/_search'               # Search an index
curl -X GET  'http://localhost:9200/<index name>/<type>/_search'        # Search an index type

# Bulk API
curl -X GET 'http://localhost:9200/_bulk'                               
curl -X GET 'http://localhost:9200/<index name>/_bulk' 
curl -X GET 'http://localhost:9200/<index name>/<type>/_bulk' 

# Elastic River Basics
curl -X GET 'http://localhost:9200/_river/_meta'                      # View River settings
curl -X GET 'http://localhost:9200/_river/<index name>/_meta'         # View River meta data for index
curl -X GET 'http://localhost:9200/_river/<index name>/_meta/_source' # View River source for index
curl -X GET 'http://localhost:9200/_river/<index name>/_status'       # View River status
curl -X GET 'http://localhost:9200/_river/<index name>/_search'       # Seach the River Index

@StephenPuiszis
https://gist.github.com/stephen-puiszis
