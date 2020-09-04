## Basic elastic search commands 


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


# Create index with configuration
# Use default ES setting, unless you have good reason.
PUT /products 
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}

GET /products/

#Add documents to index

POST /products/_doc
{
  "name": "Coffee maker",
  "price": 64,
  "in_stock": 10
}

# Add document with specified id=100
PUT /products/_doc/100
{
  "name": "Printer",
  "price": 49,
  "in_stock": 4
}


# Retrive documents
GET /products/_doc/100


# Updating documents
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}

POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}

# Scripted updates
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}

POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}

POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}

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


# Upsert: Either run the script or Insert
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

# Replace document
# Tag is gone
PUT /products/_doc/100
{
  "name": "Watch",
  "price": 49,
  "in_stock": 4
}


#Delete a document
DELETE /products/_doc/101
GET /products/_doc/101

# Remove a index
# [DELETE, HEAD, GET, PUT]
PUT /pages
DELETE /pages


______
GET /_cat/nodes?v

GET /_nodes

GET /_cat/indices?v


GET /_cat/shards?v


