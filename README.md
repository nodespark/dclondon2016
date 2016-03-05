# Elasticsearch sense commands dclondon2016:

# Check if the Elasticsearch is up and running.
GET /

# Create an index into Elasticsearch.
PUT /london
{
  "number_of_shards": 1, 
  "number_of_replicas": 0
}

# Get index settings.
GET /london/_settings

# Delete an index.
DELETE /london

GET /_all
# Get cluster healt and state
GET /_cluster/health
GET /_cluster/state

# Using Cat API.
GET /_cat/indices


#########################################################
#                 CRUD DOCUMENT

# Create a document.
# Specify the /[INDEX]/[DOCUMENT TYPE]/[ID]
PUT /london/news/1
{
  "title": "A news title",
  "author": {
    "firstname": "Nikolay",
    "lastname":  "Ignatov"
  },
  "commentcount": 100,
  "publish_date": "2016-02-29T20:00:00-0200"
}

# Check the mapping (schema).
GET /london/news/_mapping
GET /london/_mapping

# Retrive the document.
GET /london/news/1

# Retrive a non existing document.
GET /london/news/nonexists

# Put other document
PUT /london/news/2
{
  "title": "News about London Camp 2016",
  "author": {
    "firstname": "Shay",
    "lastname":  "Banon"
  },
  "publish_date": "2016-03-01T20:00:00-0200"
}

# Retrive the second document.
GET /london/news/2

# 1. Override it - PUT it again. COMPLETELY OVERRIDE!
# 2. Partial update via UPDATE API - POST
POST /london/news/2/_update
{
  "doc": {
    "commentcount": 20,
    "facebook_comments": 20
  }
}

# Retrive the update document.
GET /london/news/2

GET /london/news/_mapping

# Using the version of the document.
GET /london/news/2
PUT /london/news/2?version=6
{
  "title": "News about London Camp 2016",
  "author": {
    "firstname": "Shay",
    "lastname":  "Banon"
  },
  "publish_date": "2016-03-01T20:00:00-0200"
}

#  Delete document
DELETE /london/news/2

########################################################
#               BULK API

POST /london/news/_bulk
{"index":{"_id": 3}}
{"title": "London Camp 2016 BULK API","author":{"firstname": "Shay","lastname":  "Banon"},"publish_date": "2016-03-02T20:00:00-0200"}
{"index":{"_id": 4}}
{"title": "Title 4 testing","author":{"firstname": "Nikolay","lastname":  "Ignatov"},"publish_date": "2016-03-01T18:00:00-0200","commentcount": 99}
{"index":{"_id": 5}}
{"title": "Title 5 testing","author":{"firstname": "Nikolay","lastname":  "Banon"},"publish_date": "2016-03-01T13:00:00-0200"}
{"delete":{"_id": 2}}

#########################################################

# SEARCH API
GET /london/news/_search

# Search using the DSL
POST /london/news/_search
{
  "query": {
    "match": {
      "author.firstname": "nikolay"
    }
  }
}

# Aggregation search.
POST /london/news/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "testagg": {
      "terms": {
        "field": "author.firstname",
        "size": 10
      },
      "aggs": {
        "comment_sum": {
          "sum": {
            "field": "commentcount"
          }
        }
      }
    }
  }
}

# The power of DSL
GET /london/news/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "author.firstname": "nikolay"
        }}
      ],
      "should": [
        {"match": {
          "title": {
            "query": "news"
          }
        }},
        {"match": {
          "author.lastname": "ignatov"
        }}
      ]
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "author.lastname": {}
    }
  }
}



# Text analysis phase = Tokenization + Normalization.
# Tokenization is done by one tokenized and normalization
# is made by 0 or more token filters like lowercase and etc.
GET /_analyze?tokenizer=standard
{
  "analyzer" : "standard",
  "explain" : true,
  "text" : "Drupal Camp London has been great!"
}

# Cat all the installed plugins.
GET /_cat/plugins

