# Elasticsearch Kibana Dev Tools Command Cheat Sheet

## 1. Cluster Health & Status

### Check cluster health
```
GET /_cluster/health
```

### Check detailed cluster health
```
GET /_cluster/health?level=indices
```

### Check cluster stats
```
GET /_cluster/stats
```

### Check cluster state
```
GET /_cluster/state
```

### Check node information
```
GET /_nodes
GET /_nodes/stats
GET /_cat/nodes?v
```

---

## 2. Index Management

### List all indices
```
GET /_cat/indices?v
```

### List indices with specific pattern
```
GET /_cat/indices/log*?v
```

### Check specific index
```
GET /index_name
```

### Check index settings
```
GET /index_name/_settings
```

### Check index mappings
```
GET /index_name/_mapping
```

### Create an index
```
PUT /index_name
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```

### Delete an index
```
DELETE /index_name
```

### Close an index
```
POST /index_name/_close
```

### Open an index
```
POST /index_name/_open
```

### Check index size and document count
```
GET /_cat/indices/index_name?v&h=index,docs.count,store.size
```

---

## 3. Document Operations

### Index a document (with auto-generated ID)
```
POST /index_name/_doc
{
  "field1": "value1",
  "field2": "value2"
}
```

### Index a document (with specific ID)
```
PUT /index_name/_doc/1
{
  "field1": "value1",
  "field2": "value2"
}
```

### Get a document by ID
```
GET /index_name/_doc/1
```

### Update a document
```
POST /index_name/_update/1
{
  "doc": {
    "field1": "new_value"
  }
}
```

### Delete a document
```
DELETE /index_name/_doc/1
```

### Bulk operations
```
POST /_bulk
{ "index": { "_index": "index_name", "_id": "1" }}
{ "field1": "value1" }
{ "delete": { "_index": "index_name", "_id": "2" }}
{ "update": { "_index": "index_name", "_id": "3" }}
{ "doc": { "field1": "updated_value" }}
```

---

## 4. Search Queries

### Match all documents
```
GET /index_name/_search
{
  "query": {
    "match_all": {}
  }
}
```

### Match query
```
GET /index_name/_search
{
  "query": {
    "match": {
      "field_name": "search_term"
    }
  }
}
```

### Term query (exact match)
```
GET /index_name/_search
{
  "query": {
    "term": {
      "field_name.keyword": "exact_value"
    }
  }
}
```

### Range query
```
GET /index_name/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

### Bool query (combining conditions)
```
GET /index_name/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "field1": "value1" }}
      ],
      "filter": [
        { "term": { "field2": "value2" }}
      ],
      "should": [
        { "match": { "field3": "value3" }}
      ],
      "must_not": [
        { "match": { "field4": "value4" }}
      ]
    }
  }
}
```

### Wildcard query
```
GET /index_name/_search
{
  "query": {
    "wildcard": {
      "field_name": "val*"
    }
  }
}
```

### Multi-match query
```
GET /index_name/_search
{
  "query": {
    "multi_match": {
      "query": "search term",
      "fields": ["field1", "field2"]
    }
  }
}
```

---

## 5. Aggregations

### Terms aggregation (group by)
```
GET /index_name/_search
{
  "size": 0,
  "aggs": {
    "group_by_field": {
      "terms": {
        "field": "field_name.keyword"
      }
    }
  }
}
```

### Date histogram
```
GET /index_name/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day"
      }
    }
  }
}
```

### Stats aggregation
```
GET /index_name/_search
{
  "size": 0,
  "aggs": {
    "stats_field": {
      "stats": {
        "field": "numeric_field"
      }
    }
  }
}
```

### Avg, Sum, Min, Max
```
GET /index_name/_search
{
  "size": 0,
  "aggs": {
    "avg_value": { "avg": { "field": "numeric_field" }},
    "sum_value": { "sum": { "field": "numeric_field" }},
    "min_value": { "min": { "field": "numeric_field" }},
    "max_value": { "max": { "field": "numeric_field" }}
  }
}
```

---

## 6. Mapping Management

### Create index with mapping
```
PUT /index_name
{
  "mappings": {
    "properties": {
      "field1": { "type": "text" },
      "field2": { "type": "keyword" },
      "field3": { "type": "integer" },
      "field4": { "type": "date" }
    }
  }
}
```

### Add new field to existing mapping
```
PUT /index_name/_mapping
{
  "properties": {
    "new_field": {
      "type": "text"
    }
  }
}
```

### View mapping
```
GET /index_name/_mapping
```

---

## 7. Index Aliases

### Create an alias
```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "index_name",
        "alias": "alias_name"
      }
    }
  ]
}
```

### Remove an alias
```
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "index_name",
        "alias": "alias_name"
      }
    }
  ]
}
```

### List all aliases
```
GET /_cat/aliases?v
```

### Switch alias atomically
```
POST /_aliases
{
  "actions": [
    { "remove": { "index": "old_index", "alias": "my_alias" }},
    { "add": { "index": "new_index", "alias": "my_alias" }}
  ]
}
```

---

## 8. Reindex Operations

### Reindex from one index to another
```
POST /_reindex
{
  "source": {
    "index": "source_index"
  },
  "dest": {
    "index": "destination_index"
  }
}
```

### Reindex with query
```
POST /_reindex
{
  "source": {
    "index": "source_index",
    "query": {
      "match": {
        "field": "value"
      }
    }
  },
  "dest": {
    "index": "destination_index"
  }
}
```

---

## 9. Index Templates

### Create index template
```
PUT /_index_template/template_name
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" }
      }
    }
  }
}
```

### View index templates
```
GET /_index_template
GET /_index_template/template_name
```

### Delete index template
```
DELETE /_index_template/template_name
```

---

## 10. Shard Management

### Check shard allocation
```
GET /_cat/shards?v
```

### Check shard allocation for specific index
```
GET /_cat/shards/index_name?v
```

### Explain shard allocation
```
GET /_cluster/allocation/explain
```

### Reroute shards manually
```
POST /_cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "index_name",
        "shard": 0,
        "from_node": "node1",
        "to_node": "node2"
      }
    }
  ]
}
```

---

## 11. Performance & Monitoring

### Check pending tasks
```
GET /_cat/pending_tasks?v
```

### Check thread pool
```
GET /_cat/thread_pool?v
```

### Check recovery status
```
GET /_cat/recovery?v
```

### Check segments
```
GET /_cat/segments?v
GET /index_name/_segments
```

### Force merge (optimize)
```
POST /index_name/_forcemerge?max_num_segments=1
```

### Refresh index
```
POST /index_name/_refresh
```

### Flush index
```
POST /index_name/_flush
```

### Clear cache
```
POST /index_name/_cache/clear
```

---

## 12. Snapshot & Restore

### Create snapshot repository
```
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/my_backup"
  }
}
```

### Create snapshot
```
PUT /_snapshot/my_backup/snapshot_1
{
  "indices": "index_name",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

### List snapshots
```
GET /_snapshot/my_backup/_all
```

### Restore snapshot
```
POST /_snapshot/my_backup/snapshot_1/_restore
```

---

## 13. Analysis & Tokenization

### Analyze text
```
POST /_analyze
{
  "analyzer": "standard",
  "text": "The quick brown fox"
}
```

### Analyze with specific index analyzer
```
POST /index_name/_analyze
{
  "field": "field_name",
  "text": "text to analyze"
}
```

---

## 14. Count & Stats

### Count documents
```
GET /index_name/_count
```

### Count with query
```
GET /index_name/_count
{
  "query": {
    "match": {
      "field": "value"
    }
  }
}
```

### Index stats
```
GET /index_name/_stats
```

---

## 15. Delete Operations

### Delete by query
```
POST /index_name/_delete_by_query
{
  "query": {
    "match": {
      "field": "value"
    }
  }
}
```

### Delete all documents in index
```
POST /index_name/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

---

## 16. Useful Cat APIs

### All cat commands
```
GET /_cat
```

### Master node
```
GET /_cat/master?v
```

### Count per index
```
GET /_cat/count?v
```

### Allocation
```
GET /_cat/allocation?v
```

### Fielddata
```
GET /_cat/fielddata?v
```

### Plugins
```
GET /_cat/plugins?v
```

---

## Tips & Best Practices

1. **Add `?v` to cat APIs** for column headers
2. **Add `?pretty` to any query** for formatted JSON output
3. **Use `size: 0` in aggregations** when you only need aggregation results
4. **Use `_source: false`** when you only need document IDs
5. **Use scroll API** for large result sets
6. **Use bulk API** for indexing multiple documents efficiently
7. **Monitor cluster health regularly** to catch issues early

---

## Common Query Parameters

- `?v` - Verbose (shows column headers in cat APIs)
- `?pretty` - Pretty-print JSON output
- `?format=json` - Return output as JSON (cat APIs)
- `?h=column1,column2` - Show only specific columns (cat APIs)
- `?s=column:asc` - Sort by column (cat APIs)
- `?help` - Show available columns (cat APIs)
