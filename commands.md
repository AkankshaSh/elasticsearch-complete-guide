# Basic elastic search commands 

Cluster     
    |   
Nodes   
    |  
Indexes     
    |   
Shards  
    |   
Document

# How to write commands
```
METHOD <space> API_PATH  <space> API_REQUEST_JSON_PARAM
``` 
METHOD: [DELETE, HEAD, GET, PUT]        
API_PATH: [cluster, nodes, indices, shards]


# Check cluster Health
```
GET /_cluster/health
``` 

# Check nodes
```
GET /_cat/nodes?v
```
```
GET /_nodes
```

# List Indices

```
GET /_cat/indices?v
```

# List Shards
```
GET /_cat/shards?v
```


# Create index
```
PUT /pages
```
create index with configuration. Use default ES setting, unless you have good reason.
```
PUT /products 
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```

# Show index
```
GET /products/
```

# Remove a index
```
DELETE /pages
```

#Add documents to index
```
POST /products/_doc
{
  "name": "Coffee maker",
  "price": 64,
  "in_stock": 10
}
```

# Add document with specified id=100
```
PUT /products/_doc/100
{
  "name": "Printer",
  "price": 49,
  "in_stock": 4
}
```

# Retrieve documents
```
GET /products/_doc/100
```

# Updating documents
```
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}
```
Adding new attributes
```
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}
```

# Scripted updates
```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
```

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

```
POST /products/_update/100
{
  "script": {
    "source": """
      if(ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      } else{
        ctx.op = 'noop';
      }
      """
  }
}
```

# Upsert
Either run the script or Insert
```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Guitar",
    "price": 999,
    "in_stock": 5
  }
}
```
# Replace document
Tag is gone

```
PUT /products/_doc/100
{
  "name": "Watch",
  "price": 49,
  "in_stock": 4
}
```

#Delete a document
```
DELETE /products/_doc/101
```
```
GET /products/_doc/101
```



#Concurrency control
```
GET products/_doc/100
```
Retry the below query twice to see magic.
```
POST products/_update/100?if_primary_term=1&if_seq_no=58
{
  "doc": {
    "in_stock": 50
  }
}
```


# Bulk from file
```
curl -H "Content-type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@data/products-bulk.json"
```
Check documents in index.
```
GET /products/_search
```
How 1000 documents are distributed among shards?
```
GET /_cat/shards?v
```



_New___
GET pages/_doc/1
POST pages/_doc/1
{
  "book": "HP"
}



GET /products/_doc/100
GET /products/_doc/101


POST /products/_doc/100
{
  "name" : "Watch",
  "price" : 49,
  "in_stock" : 40
}

POST /products/_doc/100
{
  "name": "Akanksha"
}

{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 51,
  "_seq_no" : 53,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Watch",
    "price" : 49,
    "in_stock" : 4
  }
}

# Number of shards for a index can not be changed. Reason is routing
# where will a document be stored amond multiple shared gets calculated from this formula.

# shard_num = hash(_routing) % num_primary_shards



# Primary term: How many times the primary shard is been reallocated.
# Sequence number: How many total updates have happened in the index
# Concurrency Control


#Concurrency control

GET products/_doc/100

POST products/_update/100?if_primary_term=1&if_seq_no=58
{
  "doc": {
    "in_stock": 50
  }
}



# List all documents in a index
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}


# Update multiple documents in one go
#Make sure all documents have in_stock attribute
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}



# Delete multiple documents in one go

POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}



# Bulk operations
action and meta \n
optional_source \n
action and meta \n
optional_source \n

Difference between index & create action is: Create throws an error if document with given id is present, while index updates the documents if present otherwise adds a new document.

POST _bulk
{"index": {"_index": "products", "_id": 200} }
{"name": "Espresso Machine", "price": 199, "in_stock": 5}
{"create": {"_index": "products", "_id": 201} }
{"name": "Milk Frother", "price": 99, "in_stock": 10}


POST products/_bulk
{"update": {"_index": "products", "_id": 200} }
{"doc": {"in_stock": 4}}
{"delete": {"_index": "products", "_id": 201} }


GET /products/_search
GET /_cat/shards?v



# Text Analysis
# Analyser have 3 stages: charchter filter, tokenizer, token

POST /_analyze
{
  "text": "My name is Akanksha Sharma, while Husband's name is Navdeep Sharma",
  "analyzer": "standard"
}

POST /_analyze
{
  "text": "My name is Akanksha Sharma, while Husband's name is Navdeep Sharma",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}




# Mapping: create index with mapping
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type": "float"},
      "content": { "type": "text"},
      "product_id": { "type": "integer"},
      "author": {
        "properties": {
          "first_name": { "type": "text"},
          "last_name": { "type": "text"},
          "email": { "type": "keyword"}
        }
      }
    }
  }
}


PUT /reviews/_doc/1
{
  "rating": 3.5,
  "content": "Nice work, Akanksha",
  "product_id": 123,
  "author": {
    "first_name": "Akanksha",
    "last_name": "Sharma",
    "email": "a@a.com"
  }
}


# Retrive mapping

GET /reviews/_mapping/
GET /reviews/_mapping/field/rating
GET /reviews/_mapping/field/author.email


# One more way to create index with mapping: Same mapping as above
PUT /reviews_dot_notation
{
  "mappings": {
    "properties": {
      "rating": { "type": "float"},
      "content": { "type": "text"},
      "product_id": { "type": "integer"},
      "author.first_name": { "type": "text"},
      "author.last_name": { "type": "text"},
      "author.email": { "type": "keyword"}
    }
  }
}
GET /reviews/_mapping/



# Update mapping of an existing index
PUT /reviews/_mapping
{
  "properties": {
    "created_at": { "type": "date"}
  }
}
GET /reviews/_mapping/


# Date datatype in ES
PUT /reviews/_doc/1
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all",
  "product_id": 123,
  "created_at": "2020-01-01",
  "author": {
    "first_name": "Chandler",
    "last_name": "Bing",
    "email": "chandler@chandler.com"
  }
}
PUT /reviews/_doc/2
{
  "rating": 4.5,
  "content": "Could be better",
  "product_id": 123,
  "created_at": "2020-01-02T13:07:41Z",
  "author": {
    "first_name": "Joe",
    "last_name": "Tribbiani",
    "email": "joe@joe.com"
  }
}
PUT /reviews/_doc/3
{
  "rating": 3.5,
  "content": "Very useful",
  "product_id": 123,
  "created_at": "2020-01-03T09:21:51+01:00",
  "author": {
    "first_name": "Phoebe ",
    "last_name": "Buffay",
    "email": "phoebe@phoebe.com"
  }
}
PUT /reviews/_doc/4
{
  "rating": 4.5,
  "content": "Interesting",
  "product_id": 123,
  "created_at": "1436011284000",
  "author": {
    "first_name": "Monica",
    "last_name": "Geller",
    "email": "monica@monica.com"
  }
}
PUT /reviews/_doc/5
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all",
  "product_id": 123,
  "created_at": "2020-03-27",
  "author": {
    "first_name": "Rachel",
    "last_name": "Green",
    "email": "rachel@rachel.com"
  }
}
PUT /reviews/_doc/6
{
  "rating": 3.5,
  "content": "Nice work",
  "product_id": 123,
  "author": {
    "first_name": "Ross",
    "last_name": "Geller",
    "email": "ross@ross.com"
  }
}
GET /reviews/_search










