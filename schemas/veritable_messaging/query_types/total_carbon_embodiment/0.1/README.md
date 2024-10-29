# Total Carbon Embodiment Query
This document describes the Total Carbon Embodiment Veritable query.

## Query purpose
The Total Carbon Embodiment query allows a node to compute the carbon
produced when manufacturing a particular product.

## Query request
### Schema
The schema for the `data` field of a Total Carbon Embodiment query request is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/request/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query request schema for Total Carbon Embodiment",
  "type": "object",
  "properties": {
    "subjectId": {
      "type": "string"
    }
  },
  "required": [ "subjectId" ]
}
```

The `subjectId` field can be populated however is useful for
identifying the subject in a way that nodes will understand how to
find information on the product in their internal databases.  It is
possible that this is only meaningful between the immediate querier
and responder: e.g. an invoice reference number.

## Query response
### Schema
The schema for the `data` field of a Total Carbon Embodiment query response is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query response schema for total_carbon_embodiment",
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
        "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/totaL_carbon_embodiment/response/0.1"
      }
    }
  },
  "required": [ "subjectId", "mass", "unit" ]
}
```