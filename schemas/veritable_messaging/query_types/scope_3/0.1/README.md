# Scope 3 Emissions Query
This document describes the Scope 3 Emissions Veritable query.

## Query purpose
The Scope 3 Emissions query allows a node to compute the Scope 3 emissions
produced when manufacturing a particular product.

## Query request
### Schema
The schema for the `body` field of a Scope 3 Emissions query request is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/request/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query request schema for ISO 9001 compliance",
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    },
    "required": [ "subjectId" ]
  }
}
```

## Query response
### Schema
The schema for the `body` field of a Scope 3 Emissions query response is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "A Veritable question query schema for Scope 3 emissions",
  "type": "object",
  "properties": {
    "mass": {
      "type": "number"
    },
    "unit": {
      "type": "string",
      "oneOf": [ "ug", "mg", "g", "kg", "tonne"],
    },
    "required": [ "mass", "unit" ]
  }
}
```