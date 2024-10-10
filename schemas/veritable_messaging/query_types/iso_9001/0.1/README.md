# ISO 9001 Query
This document describes the ISO 9001 Veritable query.

## Query purpose
The ISO 9001 query allows a node to check if a 'subject' has an associated ISO
9001 certificate.

## Query request
### Schema
The schema for the `body` field of an ISO 9001 query request is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/request/0.1",
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

### Example
An ISO 9001 query request for a subject with ID `39C6F7EF` is as follows:
```json
{
  "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/request/0.1",
  "from": "did:example:alice",
  "to": [ "did:example:bob" ],
  "created_time": 1704067200,
  "expires_time": 1704067260,
  "body": {
    "productId": "39C6F7EF"
  }
}
```

## Query response
### Schema
The schema for the `body` field of an ISO 9001 query response is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query response schema for ISO 9001 compliance",
  "type": "object",
  "properties": {
    "id": {
      "type": "string"
    },
    "body": {
      "subjectId": {
        "type": "string"
      },
      "compliant": {
        "type": "boolean"
      },
      "certificationDate": {
        "type": [ "string", "null"],
        "format": "date"
      },
      "certificateReference": {
        "type": [ "string", "null"],
      },
      "partialResponses": {
        "type": [ "array", "null"],
        "items": {
          "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1"
        }
      },
      "required": [ "subjectId", "compliant" ],
      "dependentRequired": {
        "certificationDate": ["certificateReference"],
        "certificateReference": ["certificationDate"],
      }
    },
    "required": [ "body" ]
  }
}
```

The `subjectId` field can be populated however is useful for
identifying the subject in a way that nodes will understand how to
find information on the product in their internal databases.  It is
possible that this is only meaningful between the immediate querier
and responder: e.g. an invoice reference number.

### Example
Suppose there is a subject with ID `39C6F7EF` which depends on a number of other
subjects:
- `39C6F7EF`
  - `19237CB2`
    - `20DF8EE3`
    - `7CA09D81`

An ISO 9001 query response would look like the following:
```json
{
  "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "thid": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1",
  "from": "did:example:bob",
  "to": [ "did:example:alice" ],
  "created_time": 1704067230,
  "body": {
    "subjectId": "39C6F7EF",
    "compliant": true,
    "certificationDate": "2020-01-01T00:00:00+01:00:00",
    "certificateReference": "9CDCFDB0",
    "partialResponses": [
      {
        "id": "81660576-653f-433a-9eae-c42a5164575e",
        "body": {
          "subjectId": "19237CB2",
          "compliant": true,
          "certificationDate": "2020-01-01T00:00:00+01:00:00",
          "certificateReference": "9CDCFDB0",
          "partialResponses": [
            {
              "id": "04d911ea-ee46-43fd-af0d-48cf33f260b1",
              "body": {
                "subjectId": "20DF8EE3",
                "compliant": true,
                "certificationDate": "2022-01-01T00:00:00+01:00:00",
                "certificateReference": "1E0844B6",
                "partialResponses": []
              },
            },
            {
              "id": "8825b9f9-f136-4ca6-aab0-62a50d128d33",
              "body": {
                "subjectId": "7CA09D81",
                "compliant": true,
                "certificationDate": "2021-01-01T00:00:00+00:00:00",
                "certificateReference": "6BD14B92",
                "partialResponses": []
              }
            }
          ]
        }
      } 
    ]
  }
}
```