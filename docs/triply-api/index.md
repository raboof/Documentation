---
title: "Triply API"
path: "/docs/triply-api"
---

Each Triply instance has a fully RESTfull API. All functionality, from managing
the Triply instance to working with your data, is done through the API. This
document describes the general setup of the API, contact
[support@triply.cc](mailto:support@triply.cc) for more information.

## Authentication

When a dataset is published publicly, most of the read operation on
that dataset can be performed without authentication.

Write operations and read operations on datasets that are published
internally or privately require authentication.

### Creating an API token

Authentication is implemented through API tokens. An API token can be
created within the TriplyDB UI in the following way:

1. Log into your TriplyDB instance.
2. Click on the user menu in the top-right corner and click on “User settings”.
3. Go to the “API tokens” tab.
4. Click the “Create token” button, enter a description for the
   token (e.g., “test-token”) and select the appropriate access
   rights.
5. Click on “Create” and copy the created API token (a lengthy
   string of characters). This string is only shown once, upon
   creation, and must not be shared with others. (Other users
   can create their own token in the here described way.)

### Using the API token

API tokens are used by specifying them in an HTTP request header as
follows:

```none
Authorization: Bearer TOKEN
```

In the above, `TOKEN` should be replaced by your personal API token (a
lengthy sequence of characters). See [Creating an API token](#Creating-an-API-token) for
information on how to create an API token.

## Exporting linked data

Every TriplyDB API path that returns Linked Data provides a number of serializations to choose from. We support the following serializations:

| Serialization                                       | Media type              | File extension |
| --------------------------------------------------- | ----------------------- | -------------- |
| [TriG](https://www.w3.org/TR/trig/)                 | `application/trig`      | `.trig`        |
| [N-Triples](https://www.w3.org/TR/n-triples/)       | `application/n-triples` | `.nt`          |
| [N-Quads](https://www.w3.org/TR/n-quads/)           | `application/n-quads`   | `.nq`          |
| [Turtle](https://www.w3.org/TR/turtle/)             | `text/turtle`           | `.ttl`         |
| [JSON-LD](https://json-ld.org/spec/latest/json-ld/) | `application/ld+json`   | `.jsonld`      |

To request a serialization, use one of the following mechanisms:

1. Add an `Accept` header to the request. E.g. `Accept: application/n-triples`
2. Add the extension to the URL path. E.g. https://api.triplydb.com/datasets/Triply/iris/download.nt

## Datasets

Triply API requests are always directed towards a specific URI path.
URI paths will often have the following form:

```none
https://api.INSTANCE/datasets/ACCOUNT/DATASET/
```

Upper-case letter words must be replaced by the following values:

- `INSTANCE` :: The host name of the TriplyDB instance that you want
  to use.
- `ACCOUNT` :: The name of a specific user or a specific
  organization.
- `DATASET` :: The name of a specific dataset.

Here is an example of a URI path that points to the Triply API for
the Pokémon dataset:

```none
https://api.triplydb.com/datasets/academy/pokemon/
```

### Triple Pattern Fragments (TPF)

Triple Pattern Fragments (TPF) is a community standard that allows
individual Linked Datasets to be queried for Triply Patterns (TP), a
subset of the more complex SPARQL query language. The Triply API
implements [Triple Pattern
Fragments](http://www.hydra-cg.com/spec/latest/triple-pattern-fragments/)
version 2019-01-18 and [Linked Data
Fragments](http://www.hydra-cg.com/spec/latest/linked-data-fragments/)
version 2016-06-05.

The Triple Pattern Fragments (TPF) API is available for all datasets
in Triply and does not require running a dedicated service.


#### URI path

TPF requests are sent to the following URI path:

```none
https://api.INSTANCE/datasets/ACCOUNT/DATATSET/fragments
```

#### Reply format

Since TPF replies distinguish between data and metadata that are
stored in different graphs, it is recommended to request the TriG
content type with the following HTTP request header:

```none
Accept: application/trig
```

#### Query parameters

Triple Pattern Fragments (TPF) uses the following query parameters in
order to retrieve only those triples that adhere to a specified Triple
Pattern:

| _Key_       | _Value_                       | _Purpose_                                                                          |
| ----------- | ----------------------------- | ---------------------------------------------------------------------------------- |
| `subject`   | A URL-encoded IRI.            | Only return triples where the given IRI appears in the subject position.           |
| `predicate` | A URL-encoded IRI.            | Only return triples where the given IRI appears in the predicate position.         |
| `object`    | A URL-encoded IRI or literal. | Only return triples where the given IRI or literal appears in the object position. |

#### Example request

```bash
curl -G \
       'https://api.triplydb.com/datasets/academy/pokemon/fragments' \
       --data-urlencode 'predicate=http://www.w3.org/2000/01/rdf-schema#label' \
       -H 'Accept: application/trig'
```

### Exporting data


To export the linked data of a dataset, use the following path:

```none
https://api.INSTANCE/datasets/ACCOUNT/DATATSET/download
```


#### Query parameters

By default, an export includes all linked data graphs. Use a query argument to specify a particular graph.

| _Key_       | _Value_                       | _Purpose_                                                                          |
| ----------- | ----------------------------- | ---------------------------------------------------------------------------------- |
| `graph`   | A URL-encoded IRI.            | Only download the export of the given graph IRI.           |

#### Example request

```bash
curl 'https://api.triplydb.com/datasets/academy/pokemon/download' \
       -H 'Accept: application/trig' > export.trig.gz
```

## Services

Some API requests require the availability of a specific service over
the dataset. These requests are directed towards a URI path of the
following form:

```none
https://api.INSTANCE/datasets/ACCOUNT/DATASET/services/SERVICE/
```

Upper-case letter words must be replaced by the following values:

- `SERVICE` :: The name of a specific service that has been started
  for the corresponding dataset.

- See the previous section for [Datasets](#Datasets) to learn the meaning of
  `INSTANCE`, `ACCOUNT`, and `DATASET`.

Here is an example of a URI path that points to a SPARQL endpoint
over the Pokémon dataset:

```none
https://api.triplydb.com/datasets/academy/pokemon/services/pokemon/sparql
```

### Sparql

### Jena

### Elastic

The text search API returns a list of Linked Data entities based on a
supplied text string. The text string is matched against the text in
literals and IRIs that appear in the Linked Data description of the
returned entities.

The text search API is only available for a dataset after an
ElasticSearch service has been created for that dataset.

Two types of searches can be performed: a simple search, and a custom 
search. Simple searches require one search term for a fuzzy match. Custom
searches accept a JSON object conforming to [the elasticsearch query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

#### URI path

Text search requests are sent to the following URI path:

```none
https://api.INSTANCE/datasets/ACCOUNT/DATASET/services/SERVICE/search
```

#### Reply format

The reply format is a JSON object. Search results are returned in the
JSON array that is stored under key sequence `"hits"/"hits"`. The
order in which search results appear in the array is meaningful:
better matches appear earlier.

Every search result is represented by a JSON object. The name of the
Linked Data entity is specified under key sequence `"_id"`.
Properties of the Linked Data entity are stored as IRI keys. The
values of these properties appear in a JSON array in order to allow
more than one object term per predicate term (as is often the case in
Linked Data).

The following code snippet shows part of the reply for the below
example request. The reply includes two results for search string
“mew”, returning the Pokémon Mew (higher ranked result) and Mewtwo
(lower ranked result).

```json
{
  "hits": {
    "hits": [
      {
        "_id": "https://triply.cc/academy/pokemon/id/pokemon/mew",
        "http://open vocab org/terms/canonicalUri": [ "http://pokedex.dataincubator.org/pokemon/151" ],
        "https://triply cc/academy/pokemon/def/baseAttack": [ 100 ],
        "https://triply cc/academy/pokemon/def/name": [ "MEW", "MEW", "MEW", "MEW", "MEW", "ミュウ" ],
        …
      },
      {
        "_id": "https://triply.cc/academy/pokemon/id/pokemon/mewtwo",
        "http://open vocab org/terms/canonicalUri": [ "http://pokedex.dataincubator.org/pokemon/150" ],
        "https://triply cc/academy/pokemon/def/baseAttack": [ 110 ],
        "https://triply cc/academy/pokemon/def/name": [ "MEWTU", "MEWTWO", "MEWTWO", "MEWTWO", "MEWTWO", "ミュウツー" ],
        …
      }
    ]
  },
  …
}
```

#### Examples

##### Simple search
Perform a search for the string *mew*:
```bash
curl 'https://api.triplydb.com/datasets/academy/pokemon/services/search/search?query=mew'
```

##### Custom search
Perform a search using the custom query:
```json
{
  "query": {
    "simple_query_string": {
      "query": "pikachu"
    }
  }
}
```
```bash
curl -X POST 'https://api.triplydb.com/datasets/academy/pokemon/services/search/search' \
     -d '{"query":{"simple_query_string":{"query":"pikachu"}}}' \
     -H 'content-type: application/json'
```
