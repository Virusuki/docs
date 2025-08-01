---
aliases:
- /develop/interact/search-and-query/indexing
- /develop/interact/search-and-query/indexing/
categories:
- docs
- develop
- stack
- oss
- rs
- rc
- oss
- kubernetes
- clients
description: How to index and search JSON documents
linkTitle: Indexing
title: Indexing
weight: 3
---

In addition to indexing Redis hashes, Redis Open Source can also index JSON documents.

## Create index with JSON schema

When you create an index with the [`FT.CREATE`]({{< relref "commands/ft.create/" >}}) command, include the `ON JSON` keyword to index any existing and future JSON documents stored in the database.

To define the `SCHEMA`, you can provide [JSONPath]({{< relref "/develop/data-types/json/path" >}}) expressions.
The result of each JSONPath expression is indexed and associated with a logical name called an `attribute` (previously known as a `field`).
You can use these attributes in queries.

{{% alert title="Note" color="info" %}}
Note: `attribute` is optional for [`FT.CREATE`]({{< relref "commands/ft.create/" >}}).
{{% /alert %}}

Use the following syntax to create a JSON index:

```sql
FT.CREATE {index_name} ON JSON SCHEMA {json_path} AS {attribute} {type}
```

For example, this command creates an index that indexes the name, description, price, and image vector embedding of each JSON document that represents an inventory item:

```sql
127.0.0.1:6379> FT.CREATE itemIdx ON JSON PREFIX 1 item: SCHEMA $.name AS name TEXT $.description as description TEXT $.price AS price NUMERIC $.embedding AS embedding VECTOR FLAT 6 DIM 4 DISTANCE_METRIC L2 TYPE FLOAT32
```

See [Index limitations](#index-limitations) for more details about JSON index `SCHEMA` restrictions.

## Add JSON documents

After you create an index, Redis automatically indexes any existing, modified, or newly created JSON documents stored in the database. For existing documents, indexing runs asynchronously in the background, so it can take some time before the document is available. Modified and newly created documents are indexed synchronously, so the document will be available by the time the add or modify command finishes.

You can use any JSON write command, such as [`JSON.SET`]({{< relref "commands/json.set/" >}}) and [`JSON.ARRAPPEND`]({{< relref "commands/json.arrappend/" >}}), to create or modify JSON documents.

The following examples use these JSON documents to represent individual inventory items.

Item 1 JSON document:

```json
{
  "name": "Noise-cancelling Bluetooth headphones",
  "description": "Wireless Bluetooth headphones with noise-cancelling technology",
  "connection": {
    "wireless": true,
    "type": "Bluetooth"
  },
  "price": 99.98,
  "stock": 25,
  "colors": [
    "black",
    "silver"
  ],
  "embedding": [0.87, -0.15, 0.55, 0.03]
}
```

Item 2 JSON document:

```json
{
  "name": "Wireless earbuds",
  "description": "Wireless Bluetooth in-ear headphones",
  "connection": {
    "wireless": true,
    "type": "Bluetooth"
  },
  "price": 64.99,
  "stock": 17,
  "colors": [
    "black",
    "white"
  ],
  "embedding": [-0.7, -0.51, 0.88, 0.14]
}
```

Use [`JSON.SET`]({{< relref "commands/json.set/" >}}) to store these documents in the database:

```sql
127.0.0.1:6379> JSON.SET item:1 $ '{"name":"Noise-cancelling Bluetooth headphones","description":"Wireless Bluetooth headphones with noise-cancelling technology","connection":{"wireless":true,"type":"Bluetooth"},"price":99.98,"stock":25,"colors":["black","silver"],"embedding":[0.87,-0.15,0.55,0.03]}'
"OK"
127.0.0.1:6379> JSON.SET item:2 $ '{"name":"Wireless earbuds","description":"Wireless Bluetooth in-ear headphones","connection":{"wireless":true,"type":"Bluetooth"},"price":64.99,"stock":17,"colors":["black","white"],"embedding":[-0.7,-0.51,0.88,0.14]}'
"OK"
```

Because indexing is synchronous in this case, the documents will be available on the index as soon as the [`JSON.SET`]({{< relref "commands/json.set/" >}}) command returns.
Any subsequent queries that match the indexed content will return the document.

## Search the index

To search the index for JSON documents, use the [`FT.SEARCH`]({{< relref "commands/ft.search/" >}}) command.
You can search any attribute defined in the `SCHEMA`.

For example, use this query to search for items with the word "earbuds" in the name:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@name:(earbuds)'
1) "1"
2) "item:2"
3) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"connection\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embedding\":[-0.7,-0.51,0.88,0.14]}"
```

This query searches for all items that include "bluetooth" and "headphones" in the description:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(bluetooth headphones)'
1) "2"
2) "item:1"
3) 1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"], \"embedding\":[0.87,-0.15,0.55,0.03]}"
4) "item:2"
5) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"connection\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embedding\":[-0.7,-0.51,0.88,0.14]}"
```

Now search for Bluetooth headphones with a price less than 70:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(bluetooth headphones) @price:[0 70]'
1) "1"
2) "item:2"
3) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"connection\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embedding\":[-0.7,-0.51,0.88,0.14]}"
```

And lastly, search for the Bluetooth headphones that are most similar to an image whose embedding is [1.0, 1.0, 1.0, 1.0]:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(bluetooth headphones)=>[KNN 2 @embedding $blob]' PARAMS 2 blob \x01\x01\x01\x01 DIALECT 2
1) "2"
2) "item:1"
3) 1) "__embedding_score"
   2) "1.08280003071"
   1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"],\"embedding\":[0.87,-0.15,0.55,0.03]}"
2) "item:2"
3) 1) "__embedding_score"
   2) "1.54409992695"
   3) "$"
   4) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"connection\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embedding\":[-0.7,-0.51,0.88,0.14]}"
```

For more information about search queries, see [Search query syntax]({{< relref "/develop/ai/search-and-query/advanced-concepts/query_syntax" >}}).

{{% alert title="Note" color="info" %}}
[`FT.SEARCH`]({{< relref "commands/ft.search/" >}}) queries require `attribute` modifiers. Don't use JSONPath expressions in queries because the query parser doesn't fully support them.
{{% /alert %}}

## Index JSON arrays as TAG

The preferred method for indexing a JSON field with multivalued terms is using JSON arrays. Each value of the array is indexed, and those values must be scalars. If you want to index string or boolean values as TAGs within a JSON array, use the [JSONPath]({{< relref "/develop/data-types/json/path" >}}) wildcard operator.

To index an item's list of available colors, specify the JSONPath `$.colors.*` in the `SCHEMA` definition during index creation:

```sql
127.0.0.1:6379> FT.CREATE itemIdx2 ON JSON PREFIX 1 item: SCHEMA $.colors.* AS colors TAG $.name AS name TEXT $.description as description TEXT
```

Now you can search for silver headphones:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx2 "@colors:{silver} (@name:(headphones)|@description:(headphones))"
1) "1"
2) "item:1"
3) 1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"]}"
```

## Index JSON arrays as TEXT
Starting with RediSearch v2.6.0, full text search can be done on an array of strings or on a JSONPath leading to multiple strings.

If you want to index multiple string values as TEXT, use either a JSONPath leading to a single array of strings, or a JSONPath leading to multiple string values, using JSONPath operators such as wildcard, filter, union, array slice, and/or recursive descent.

To index an item's list of available colors, specify the JSONPath `$.colors` in the `SCHEMA` definition during index creation:

```sql
127.0.0.1:6379> FT.CREATE itemIdx3 ON JSON PREFIX 1 item: SCHEMA $.colors AS colors TEXT $.name AS name TEXT $.description as description TEXT
```

```sql
127.0.0.1:6379> JSON.SET item:3 $ '{"name":"True Wireless earbuds","description":"True Wireless Bluetooth in-ear headphones","connection":{"wireless":true,"type":"Bluetooth"},"price":74.99,"stock":20,"colors":["red","light blue"]}'
"OK"
```

Now you can do full text search for light colored headphones:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx3 '@colors:(white|light) (@name|description:(headphones))' RETURN 1 $.colors
1) (integer) 2
2) "item:2"
3) 1) "$.colors"
   2) "[\"black\",\"white\"]"
4) "item:3"
5) 1) "$.colors"
   2) "[\"red\",\"light blue\"]"
```

### Limitations
- When a JSONPath may lead to multiple values and not only to a single array, e.g., when a JSONPath contains wildcards, etc., specifying `SLOP` or `INORDER` in [`FT.SEARCH`]({{< relref "commands/ft.search/" >}}) will return an error, since the order of the values matching the JSONPath is not well defined, leading to potentially inconsistent results.

   For example, using a JSONPath such as `$..b[*]` on a JSON value such as
   ```json
   {
      "a": [
         {"b": ["first first", "first second"]},
         {"c":
            {"b": ["second first", "second second"]}},
         {"b": ["third first", "third second"]}
      ]
   }
   ```
   may match values in various orderings, depending on the specific implementation of the JSONPath library being used.

   Since `SLOP` and `INORDER` consider relative ordering among the indexed values, and results may change in future releases, an error will be returned.

- When JSONPath leads to multiple values:
  - String values are indexed
  - `null` values are skipped
  - Any other value type will cause an indexing failure

- `SORTBY` only sorts by the first value
- No `HIGHLIGHT` and `SUMMARIZE` support
- `RETURN` of a Schema attribute, whose JSONPath leads to multiple values, returns only the first value (as a JSON String)
- If a JSONPath is specified by the `RETURN`, instead of a Schema attribute, all values are returned (as a JSON String)

### Handling phrases in different array slots:

When indexing, a predefined delta is used to increase positional offsets between array slots for multiple text values. This delta controls the level of separation between phrases in different array slots (related to the `SLOP` parameter of [`FT.SEARCH`]({{< relref "commands/ft.search/" >}})).
This predefined value is set by the configuration parameter `MULTI_TEXT_SLOP` (at module load-time). The default value is 100.

## Index JSON arrays as NUMERIC

Starting with RediSearch v2.6.1, search can be done on an array of numerical values or on a JSONPath leading to multiple numerical values.

If you want to index multiple numerical values as NUMERIC, use either a JSONPath leading to a single array of numbers, or a JSONPath leading to multiple numbers, using JSONPath operators such as wildcard, filter, union, array slice, and/or recursive descent.

For example, add to the item's list the available `max_level` of volume (in decibels):

```sql
127.0.0.1:6379> JSON.SET item:1 $ '{"name":"Noise-cancelling Bluetooth headphones","description":"Wireless Bluetooth headphones with noise-cancelling technology","connection":{"wireless":true,"type":"Bluetooth"},"price":99.98,"stock":25,"colors":["black","silver"], "max_level":[60, 70, 80, 90, 100]}'
OK

127.0.0.1:6379> JSON.SET item:2 $ '{"name":"Wireless earbuds","description":"Wireless Bluetooth in-ear headphones","connection":{"wireless":true,"type":"Bluetooth"},"price":64.99,"stock":17,"colors":["black","white"], "max_level":[80, 100, 120]}'
OK

127.0.0.1:6379> JSON.SET item:3 $ '{"name":"True Wireless earbuds","description":"True Wireless Bluetooth in-ear headphones","connection":{"wireless":true,"type":"Bluetooth"},"price":74.99,"stock":20,"colors":["red","light blue"], "max_level":[90, 100, 110, 120]}'
OK
```

To index the `max_level` array, specify the JSONPath `$.max_level` in the `SCHEMA` definition during index creation:

```sql
127.0.0.1:6379> FT.CREATE itemIdx4 ON JSON PREFIX 1 item: SCHEMA $.max_level AS dB NUMERIC
OK
```

You can now search for headphones with specific max volume levels, for example, between 70 and 80 (inclusive), returning items with at least one value in their `max_level` array, which is in the requested range:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx4 '@dB:[70 80]'
1) (integer) 2
2) "item:1"
3) 1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"],\"max_level\":[60,70,80,90,100]}"
4) "item:2"
5) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"max_level\":[80,100,120]}"
```

You can also search for items with all values in a specific range. For example, all values are in the range [90, 120] (inclusive):

```sql
127.0.0.1:6379> FT.SEARCH itemIdx4 '-@dB:[-inf (90] -@dB:[(120 +inf]'
1) (integer) 1
2) "item:3"
3) 1) "$"
   2) "{\"name\":\"True Wireless earbuds\",\"description\":\"True Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":74.99,\"stock\":20,\"colors\":[\"red\",\"light blue\"],\"max_level\":[90,100,110,120]}"
```

### Limitations

When JSONPath leads to multiple numerical values:
  - Numerical values are indexed
  - `null` values are skipped
  - Any other value type will cause an indexing failure

## Index JSON arrays as GEO and GEOSHAPE

You can use `GEO` and `GEOSHAPE` fields to store geospatial data,
such as geographical locations and geometric shapes. See
[Geospatial indexing]({{< relref "/develop/ai/search-and-query/indexing/geoindex" >}})
to learn how to use these schema types and see the
[Geospatial]({{< relref "/develop/ai/search-and-query/advanced-concepts/geo" >}})
reference page for an introduction to their format and usage.

## Index JSON arrays as VECTOR

Starting with RediSearch 2.6.0, you can index a JSONPath leading to an array of numeric values as a VECTOR type in the index schema.

For example, assume that your JSON items include an array of vector embeddings, where each vector represents an image of a product. To index these vectors, specify the JSONPath `$.embedding` in the schema definition during index creation:

```sql
127.0.0.1:6379> FT.CREATE itemIdx5 ON JSON PREFIX 1 item: SCHEMA $.embedding AS embedding VECTOR FLAT 6 DIM 4 DISTANCE_METRIC L2 TYPE FLOAT32
OK
127.0.0.1:6379> JSON.SET item:1 $ '{"name":"Noise-cancelling Bluetooth headphones","description":"Wireless Bluetooth headphones with noise-cancelling technology","price":99.98,"stock":25,"colors":["black","silver"],"embedding":[0.87,-0.15,0.55,0.03]}'
OK
127.0.0.1:6379> JSON.SET item:2 $ '{"name":"Wireless earbuds","description":"Wireless Bluetooth in-ear headphones","price":64.99,"stock":17,"colors":["black","white"],"embedding":[-0.7,-0.51,0.88,0.14]}'
OK
```

Now you can search for the two headphones that are most similar to the image embedding by using vector search KNN query. (Note that the vector queries are supported as of dialect 2.) For example:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx5 '*=>[KNN 2 @embedding $blob AS dist]' SORTBY dist PARAMS 2 blob \x01\x01\x01\x01 DIALECT 2
1) (integer) 2
2) "item:1"
3) 1) "dist"
   2) "1.08280003071"
   3) "$"
   4) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"],\"embedding\":[0.87,-0.15,0.55,0.03]}"
4) "item:2"
5) 1) "dist"
   2) "1.54409992695"
   3) "$"
   4) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embedding\":[-0.7,-0.51,0.88,0.14]}"
```

If you want to index multiple numeric arrays as VECTOR, use a [JSONPath]({{< relref "/develop/data-types/json/path" >}}) leading to multiple numeric arrays using JSONPath operators such as wildcard, filter, union, array slice, and/or recursive descent.

For example, assume that your JSON items include an array of vector embeddings, where each vector represents a different image of the same product. To index these vectors, specify the JSONPath `$.embeddings[*]` in the schema definition during index creation:

```sql
127.0.0.1:6379> FT.CREATE itemIdx5 ON JSON PREFIX 1 item: SCHEMA $.embeddings[*] AS embeddings VECTOR FLAT 6 DIM 4 DISTANCE_METRIC L2 TYPE FLOAT32
OK
127.0.0.1:6379> JSON.SET item:1 $ '{"name":"Noise-cancelling Bluetooth headphones","description":"Wireless Bluetooth headphones with noise-cancelling technology","price":99.98,"stock":25,"colors":["black","silver"],"embeddings":[[0.87,-0.15,0.55,0.03]]}'
OK
127.0.0.1:6379> JSON.SET item:2 $ '{"name":"Wireless earbuds","description":"Wireless Bluetooth in-ear headphones","price":64.99,"stock":17,"colors":["black","white"],"embeddings":[[-0.7,-0.51,0.88,0.14],[-0.8,-0.15,0.33,-0.01]]}'
OK
```

{{% alert title="Important note" color="info" %}}
Unlike the case with the NUMERIC type, setting a static path such as `$.embedding` in the schema for the VECTOR type does not allow you to index multiple vectors stored under that field. Hence, if you set `$.embedding` as the path to the index schema, specifying an array of vectors in the `embedding` field in your JSON will cause an indexing failure.
{{% /alert %}}

Now you can search for the two headphones that are most similar to an image embedding by using vector search KNN query. (Note that the vector queries are supported as of dialect 2.) The distance between a document to the query vector is defined as the minimum distance between the query vector to a vector that matches the JSONPath specified in the schema. For example:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx5 '*=>[KNN 2 @embeddings $blob AS dist]' SORTBY dist PARAMS 2 blob \x01\x01\x01\x01 DIALECT 2
1) (integer) 2
2) "item:2"
3) 1) "dist"
   2) "0.771500051022"
   3) "$"
   4) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"],\"embeddings\":[[-0.7,-0.51,0.88,0.14],[-0.8,-0.15,0.33,-0.01]]}"
4) "item:1"
5) 1) "dist"
   2) "1.08280003071"
   3) "$"
   4) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"],\"embeddings\":[[0.87,-0.15,0.55,0.03]]}"
```
Note that `0.771500051022` is the L2 distance between the query vector and `[-0.8,-0.15,0.33,-0.01]`, which is the second element in the embedding array, and it is lower than the L2 distance between the query vector and `[-0.7,-0.51,0.88,0.14]`, which is the first element in the embedding array.

For more information on vector similarity syntax, see [Vector fields]({{< relref "/develop/ai/search-and-query/vectors" >}}).

## Index JSON objects

You cannot index JSON objects. FT.CREATE will return an error if the JSONPath expression returns an object.

To index the contents of a JSON object, you need to index the individual elements within the object in separate attributes.

For example, to index the `connection` JSON object, define the `$.connection.wireless` and `$.connection.type` fields as separate attributes when you create the index:

```sql
127.0.0.1:6379> FT.CREATE itemIdx3 ON JSON SCHEMA $.connection.wireless AS wireless TAG $.connection.type AS connectionType TEXT
"OK"
```

After you create the new index, you can search for items with the wireless TAG set to `true`:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx3 '@wireless:{true}'
1) "2"
2) "item:2"
3) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"connection\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"]}"
4) "item:1"
5) 1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"]}"
```

You can also search for items with a Bluetooth connection type:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx3 '@connectionType:(bluetooth)'
1) "2"
2) "item:1"
3) 1) "$"
   2) "{\"name\":\"Noise-cancelling Bluetooth headphones\",\"description\":\"Wireless Bluetooth headphones with noise-cancelling technology\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":99.98,\"stock\":25,\"colors\":[\"black\",\"silver\"]}"
4) "item:2"
5) 1) "$"
   2) "{\"name\":\"Wireless earbuds\",\"description\":\"Wireless Bluetooth in-ear headphones\",\"connection\":{\"wireless\":true,\"type\":\"Bluetooth\"},\"price\":64.99,\"stock\":17,\"colors\":[\"black\",\"white\"]}"
```

## Field projection

[`FT.SEARCH`]({{< relref "commands/ft.search/" >}}) returns the entire JSON document by default. If you want to limit the returned search results to specific attributes, you can use field projection.

### Return specific attributes

When you run a search query, you can use the `RETURN` keyword to specify which attributes you want to include in the search results. You also need to specify the number of fields to return.

For example, this query only returns the `name` and `price` of each set of headphones:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(headphones)' RETURN 2 name price
1) "2"
2) "item:1"
3) 1) "name"
   2) "Noise-cancelling Bluetooth headphones"
   3) "price"
   4) "99.98"
4) "item:2"
5) 1) "name"
   2) "Wireless earbuds"
   3) "price"
   4) "64.99"
```

### Project with JSONPath

You can use [JSONPath]({{< relref "/develop/data-types/json/path" >}}) expressions in a `RETURN` statement to extract any part of the JSON document, even fields that were not defined in the index `SCHEMA`.

For example, the following query uses the JSONPath expression `$.stock` to return each item's stock in addition to the name and price attributes.

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(headphones)' RETURN 3 name price $.stock
1) "2"
2) "item:1"
3) 1) "name"
   2) "Noise-cancelling Bluetooth headphones"
   3) "price"
   4) "99.98"
   5) "$.stock"
   6) "25"
4) "item:2"
5) 1) "name"
   2) "Wireless earbuds"
   3) "price"
   4) "64.99"
   5) "$.stock"
   6) "17"
```

Note that the returned property name is the JSONPath expression itself: `"$.stock"`.

You can use the `AS` option to specify an alias for the returned property:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '@description:(headphones)' RETURN 5 name price $.stock AS stock
1) "2"
2) "item:1"
3) 1) "name"
   2) "Noise-cancelling Bluetooth headphones"
   3) "price"
   4) "99.98"
   5) "stock"
   6) "25"
4) "item:2"
5) 1) "name"
   2) "Wireless earbuds"
   3) "price"
   4) "64.99"
   5) "stock"
   6) "17"
```

This query returns the field as the alias `"stock"` instead of the JSONPath expression `"$.stock"`.

### Highlight search terms

You can [highlight]({{< relref "/develop/ai/search-and-query/advanced-concepts/highlight" >}}) relevant search terms in any indexed `TEXT` attribute.

For [`FT.SEARCH`]({{< relref "commands/ft.search/" >}}), you have to explicitly set which attributes you want highlighted after the `RETURN` and `HIGHLIGHT` parameters.

Use the optional `TAGS` keyword to specify the strings that will surround (or highlight) the matching search terms.

For example, highlight the word "bluetooth" with bold HTML tags in item names and descriptions:

```sql
127.0.0.1:6379> FT.SEARCH itemIdx '(@name:(bluetooth))|(@description:(bluetooth))' RETURN 3 name description price HIGHLIGHT FIELDS 2 name description TAGS '<b>' '</b>'
1) "2"
2) "item:1"
3) 1) "name"
   2) "Noise-cancelling <b>Bluetooth</b> headphones"
   3) "description"
   4) "Wireless <b>Bluetooth</b> headphones with noise-cancelling technology"
   5) "price"
   6) "99.98"
4) "item:2"
5) 1) "name"
   2) "Wireless earbuds"
   3) "description"
   4) "Wireless <b>Bluetooth</b> in-ear headphones"
   5) "price"
   6) "64.99"
```

## Aggregate with JSONPath

You can use [aggregation]({{< relref "/develop/ai/search-and-query/advanced-concepts/aggregations" >}}) to generate statistics or build facet queries.

The `LOAD` option accepts [JSONPath]({{< relref "/develop/data-types/json/path" >}}) expressions. You can use any value in the pipeline, even if the value is not indexed.

This example uses aggregation to calculate a 10% price discount for each item and sorts the items from least expensive to most expensive:

```sql
127.0.0.1:6379> FT.AGGREGATE itemIdx '*' LOAD 4 name $.price AS originalPrice APPLY '@originalPrice - (@originalPrice * 0.10)' AS salePrice SORTBY 2 @salePrice ASC
1) "2"
2) 1) "name"
   2) "Wireless earbuds"
   3) "originalPrice"
   4) "64.99"
   5) "salePrice"
   6) "58.491"
3) 1) "name"
   2) "Noise-cancelling Bluetooth headphones"
   3) "originalPrice"
   4) "99.98"
   5) "salePrice"
   6) "89.982"
```

{{% alert title="Note" color="info" %}}
[`FT.AGGREGATE`]({{< relref "commands/ft.aggregate/" >}}) queries require `attribute` modifiers. Don't use JSONPath expressions in queries, except with the `LOAD` option, because the query parser doesn't fully support them.
{{% /alert %}}

## Index missing or empty values
As of v2.10, you can search for missing properties, that is, properties that do not exist in a given document, using the `INDEXMISSING` option to `FT.CREATE` in conjunction with the `ismissing` query function with `FT.SEARCH`. You can also search for existing properties with no value (i.e., empty) using the `INDEXEMPTY` option with `FT.CREATE`. Both query types require DIALECT 2. Examples below:

```
JSON.SET key:1 $ '{"propA": "foo"}'
JSON.SET key:2 $ '{"propA": "bar", "propB":"abc"}'
FT.CREATE idx ON JSON PREFIX 1 key: SCHEMA $.propA AS propA TAG $.propB AS propB TAG INDEXMISSING

> FT.SEARCH idx 'ismissing(@propB)' DIALECT 2
1) "1"
2) "key:1"
3) 1) "$"
   2) "{\"propA\":\"foo\"}"
```

```
JSON.SET key:1 $ '{"propA": "foo", "propB":""}'
JSON.SET key:2 $ '{"propA": "bar", "propB":"abc"}'
FT.CREATE idx ON JSON PREFIX 1 key: SCHEMA $.propA AS propA TAG $.propB AS propB TAG INDEXEMPTY

> FT.SEARCH idx '@propB:{""}' DIALECT 2
1) "1"
2) "key:1"
3) 1) "$"
   2) "{\"propA\":\"foo\",\"propB\":\"\"}"
```

## Index limitations

### Schema mapping

During index creation, you need to map the JSON elements to `SCHEMA` fields as follows:

- Strings as `TEXT`, `TAG`, or `GEO`.
- Numbers as `NUMERIC`.
- Booleans as `TAG`.
- JSON array
  - Array of strings as `TAG` or `TEXT`.
  - Array of numbers as `NUMERIC` or `VECTOR`.
  - Array of geo coordinates as `GEO`.
  - `null` values in such arrays are ignored.
- You cannot index JSON objects. Index the individual elements as separate attributes instead.
- `null` values are ignored.

### Sortable tags

If you create an index for JSON documents with a JSONPath leading to an array or to multiple values, only the first value is considered by the sort.
