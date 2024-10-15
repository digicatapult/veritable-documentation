# Scope 3 Emissions Query
This document describes the scope 3 emissions Veritable query.

## Query purpose
The Scope 3 Emissions query allows a node to compute the scope 3 emissions
produced when manufacturing a particular product.

## Query request
### Schema
The schema for the `data` field of a scope 3 emissions query request is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/request/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query request schema for scope 3 emissions",
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    }
  },
  "required": [ "subjectId" ]
}
```

## Query response
### Schema
The schema for the `data` field of a Scope 3 Emissions query response is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query response schema for Scope 3 Emissions",
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "mass": {
      "type": "number"
    },
    "unit": {
      "type": "string",
      "oneOf": [ "ug", "mg", "g", "kg", "tonne"],
    },
    "partialResponses": {
      "type": [ "array", "null"],
      "items": {
        "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1"
      }
    }
  },
  "required": [ "subjectId", "mass", "unit" ]
}
```