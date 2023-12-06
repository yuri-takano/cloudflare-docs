---
title: Metadata filtering
pcx_content_type: concept
weight: 6
---

# Metadata Filtering

In addition to providing an input vector to your query, you can also filter by [vector metadata](/vectorize/learning/insert-vectors/#metadata) associated with every vector. Query results only include vectors that match `filter` criteria. 

## Supported operations

Optional `filter` property on `query()` method specifies metadata filter.
- Operators: `$eq` (equals), `$ne` (not equals)
- `filter` must be non-empty object whose compact JSON representation must be less than 2048 bytes
- `filter` object keys cannot be empty, contain `"`, start with `$`, or be longer than 512 characters

Valid `filter` examples:

Implicit `$eq` operator
```json
{ streaming_platform: "netflix" }
```

Explicit operator
```json
{ "someKey": { "$ne": true } }
```

Implicit logical `AND` with multiple keys
```json
{ "pandas.nice": 42, "someKey": { "$ne": true } }
```

Keys define nesting with `.` (dot)
```json
{ "pandas.nice": 42 } // looks for { "pandas": { "nice": 42 } }
```

## Examples

### Add metadata

With the following index defintion:

```sh
$ npx wrangler vectorize create tutorial-index --dimensions=3 --metric=cosine
```

Metadata can be added when [inserting or upserting vectors](/vectorize/learning/insert-vectors/#examples).

```ts
const newMetadataVectors: Array<VectorizeVector> = [
	{ 
    id: "1", 
    values: [32.4, 74.1, 3.2], 
    metadata: { url: "/products/sku/13913913", streaming_platform: "netflix" } 
  },
	{ 
    id: "2", 
    values: [15.1, 19.2, 15.8], 
    metadata: { url: "/products/sku/10148191", streaming_platform: "hbo" } 
  },
	{ 
    id: "3", 
    values: [0.16, 1.2, 3.8], 
    metadata: { url: "/products/sku/97913813", streaming_platform: "amazon" } 
  },
	{ id: "4",
    values: [75.1, 67.1, 29.9], 
    metadata: { url: "/products/sku/418313", streaming_platform: "netflix" } 
  },
	{ 
    id: "5", 
    values: [58.8, 6.7, 3.4], 
    metadata: { url: "/products/sku/55519183", streaming_platform: "hbo" } 
  },
];

// Upsert vectors with added metadata, returning a count of the vectors upserted and their vector IDs
let upserted = await env.YOUR_INDEX.upsert(newMetadataVectors);
```

### Query examples

Use the `query()` method:

```ts
let queryVector: Array<number> = [54.8, 5.5, 3.1];
// Best match is vector id = 5 (score closet to 1)
let originalMatches = await env.YOUR_INDEX.query(queryVector, { topK: 3, returnValues: true, returnMetadata: true });
```

Results without metadata filtering:

```json
{"matches":[{"vectorId":"5","score":0.999909486},{"vectorId":"4","score":0.789848214},{"vectorId":"2","score":0.611976262}]}
```

The same `query()` method with a `filter` property supports metadata filtering.

```ts
let queryVector: Array<number> = [54.8, 5.5, 3.1];
// Best match is vector id = 4 with metadata filter
let metadataMatches = await env.YOUR_INDEX.query(queryVector, { topK: 3, filter: { streaming_platform: "netflix" }, returnValues: true, returnMetadata: true } )
```

Results with metadata filtering:

```json
{"matches":[{"vectorId":"4","score":0.789848214},{"vectorId":"1","score":0.491185264}]}
```

## Limitations
- Only newly created indexes on or after 2023-12-06 support metadata filtering.