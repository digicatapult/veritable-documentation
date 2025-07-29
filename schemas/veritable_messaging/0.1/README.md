# Veritable messaging
Veritable allows a user to ask a supply chain question and obtain a response.
This document describes the Veritable message structure and flow.  Separate
documents describe query and response formats, which are the Veritable message
payloads.  Different query types use the same standard Veritable message
structure and flow, but the message payload is different for different query
types (compare a compliance query with a numerical query).

## Transport
Veritable uses
[DRPC](https://github.com/hyperledger/aries-rfcs/blob/ea87d2e37640ef944568e3fa01df1f36fe7f0ff3/features/0804-didcomm-rpc/README.md)
for communication between nodes -- a form of JSON-RPC over
[DIDComm](https://identity.foundation/didcomm-messaging/spec/).  The reader is
assumed to be familiar with the [DIDComm
specifications](https://identity.foundation/didcomm-messaging/spec).

The DIDComm Protocol Identifier URI (PIURI) for Veritable is
`https://github.com/digicatapult/veritable-documentation/tree/main/docs/veritable_messaging/0.1`.
It is standard practice to define the PIURI for DIDComm protocols, but note that
this is not used in the JSON RPC format.

## Query flow
A Veritable query consists of two DRPC methods: the `submit_query_request` method
(asking the question) and the `submit_query_response` method (giving the answer).  Each
of these is a single DRPC exchange consisting of two messages as shown below.

`submit_query_request`:
```
  A                                         B
       query request (DRPC request)
      ------------------------------------>
       acknowledgement (DRPC response)
      <------------------------------------
```

`submit_query_response`:
```
  A                                         B
       query response (DRPC request)
      <------------------------------------
       acknowledgement (DRPC response)
      ------------------------------------>
```

Defining the query request and query response as two separate DRPC calls allows
for an asynchronous message flow.

## Veritable query message structure
Veritable currently defines three types of query: ISO 9001, Total Carbon Embodiment,
and Audit.  The DIDComm Message Type URIs (MTURIs), which are included inside DRPC messages, are as follows:
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/response/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1`

An additional 'acknowledgement' message is also defined, with the following
MTURI:
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1`

This subsection provides templates for different kinds of query request and
response messages.  The templates show how the JSON RPC data structure is
populated.

### Veritable message schema
Veritable queries and responses are contained within Veritable messages, which
provide various metadata.  The schema for the `params` field of a Veritable DRPC
message is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable message schema",
  "type": "object",
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string"
    },
    "createdTime": {
      "type": "number"
    },
    "expiresTime": {
      "type": "number"
    },
    "data": {
      "type": "object",
      "oneOf": [
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/request/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/request/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/response/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/request/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1" },
        { "$ref": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1" }
      ]
    },
    "required": [ "id", "type", "createdTime", "data" ]
  }
}
```

### `submit_query_request` method
#### DRPC Request
The DRPC request message follows the following template:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/<query type>/request/0.1",
    "createdTime": 1704067200,
    "expiresTime": 1704067260,
    "data": {
    // Veritable query request as described in the <query type> document
    }
  }
}
```
The `id` field is a UUID used to link the query to the response.  The DRPC
response and the DRPC request and response messages for the corresponding
`submit_query_response` method call all use the same `id` in this field so that
the response can be linked to the query.  In future Veritable iterations, the
query request and response will be linked via DIDComm thread IDs (see [Threaded
responses](#threaded-responses)).

The `data` field is populated according to the query type (identified by the
MTURI value in the `type` field).  These data structures are defined in the
MTURIs listed above.

Note that the DIDComm connection used to send this JSON RPC call (i.e. the DRPC
message) may include its own `createdTime` and `expiresTime` fields.  These
'outer' times refer to expiry of the DRPC response, whereas the 'inner' times
shown above refer to the expiry of the query request.

The query request `expiresTime` field should be set to a 'reasonable' timeframe in which the
responder is expected to be able to determine and send a response to the query request.  By
default, the `expiresTime` is set to 60 seconds after the `createdTime`.

A querier should not call the `submit_query_request` method again before
`expiresTime` has passed. If it does pass without a corresponding
`submit_query_response` call being received, the querier may call
`submit_query_request` again.  In this situation, the new `submit_query_request`
DRPC request message must use the same `id` as the as in the first
`submit_query_request` DRPC request message so that the responder knows it only
needs to respond once to possibly multiple query requests.

#### DRPC Response
The DRPC response message is the Veritable `acknowledgement` message:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1",
    "createdTime": 1704067200,
    "expiresTime": 1704067260,
    "data": {
      "returnCode": 0
    }
  }
}
```

The optional `expiresTime` field used in the DRPC response message in the
`submit_query_request` method indicates approximately when the querier should
expect a `submit_query_response` DRPC request message.  This allows the
responder to adjust querier expectations to cope with complex queries or queries
that must be decomposed into a large number of partial queries.

The `expiresTime` field in the DRPC response should generally not be later than
the `expiresTime` field in the corresponding DRPC request, since this implies
the query might only be answerable after it expires.  If this is the case, the
responder must provide an `returnCode` of `1` in the acknowledgement, and `expiresTime` should be used
by the querier to decide on whether to call `submit_query_request` again with a
new `expiresTime` value or not to repeat the query.  Return codes are defined in
the [Query acknowledgement](../query_ack/0.1/README.md) documentation.

If the querier receives a `0` return code (no error) then it should not call the
`submit_query_request` method again for the same query until the latest value of
the `expiresTime` values from the DRPC request and DRPC response messages has
passed.

### `submit_query_response` method

#### DRPC Request
A query response message follows the following query response template:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_response",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/<query type>/response/0.1",
    "createdTime": 1704067233,
    "expiresTime": 1704067833,
    "data": {
    // Veritable query response as described in the <query type> document
      "partialResponses": []
    }
  }
}
```
The `data` field is populated according to the query type (identified by the
MTURI value in the `type` field).  These data structures are defined in the
MTURIs listed above.

The `expiresTime` field is optional and may be used to limit the validity of
the response.  Future Veritable iterations may support query response caching
(see [Response caching](#response-caching)).

The `partialResponses` field is described in the [Query processing and partial
queries](#query-processing-and-partial-queries) section, below.  The
`partialResponses` field should be treated the same if it is absent, present and
equal to `null`, or present and equal to an empty array, `[]`; the preference is
for it to be absent when not used.

#### DRPC Response
The DRPC response in the `submit_query_response` method is defined as a Veritable
`acknowledgement` message:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_response",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1",
    "createdTime": 1704067200,
    "expiresTime": 1704067260,
    "data": {
      "returnCode": 0
    }
  }
}
```

The `expiresTime` field is not currently used in the `submit_query_response`
DRPC response message.  It may be used in future Veritable iterations to
indicate for how long the answer should be considered valid (for time-sensitive
responses).  The notion of response validity may also be used in future
Veritable iterations for caching query responses (see [Response
caching](#response-caching)).

## Query processing and partial queries
A core assumption in Veritable is that some suppliers and customers may only be
content to respond to queries if they originate from an entity with which they
have a pre-existing business relationship.

As such, answering complex supply chain queries is expected to involve some
nodes breaking queries down into _partial queries_ and collating the responses
to pass information upwards.

In Veritable's first iteration, each responding node includes _partial query
responses_ in its own query response message, possibly adding its own data,
before sending it. Embedding query responses means response aggregation can be
done by the 'top-level' querier however they find useful, and reduces the
amount of state each node has to store for audits.  Crucially, the embedding
process is designed not to leak information about the identity of the node that 
provided a partial query response.

The `partialResponses` array is an array of objects of the same type as
`params`.  To construct the array, the `params` field is extracted from partial
query responses and placed as an object inside the `partialResponses` array.
See the [ISO 9001 query](../query_types/iso_9001/0.1/README.md) documentation
for examples.  The order of responses has no meaning, and may be randomised by a
node to conceal the order in which partial query responses were received.

Note that the `id` on a partial query is _not_ the same as the `id` of the query
from which the partial queries were derived.  These identifiers are used during
audit (see [Audit query](../query_types/audit/0.1/README.md)) and must be unique
for each (partial) query.

## State
For auditing purposes (in the first version of Veritable), each node should
store a database holding pairs (Veritable node ID, message ID).  The Veritable
node ID is a 'permanent' identifier (such as a company name and number) for the
message sender, and the message ID is the value of the `id` field in the DRPC
message.  The reasoning is explained in the [Audit
query](../query_types/audit/0.1/README.md) documentation.

## Future work
Some current Veritable limitations are inherited from the upstream project
[credo-ts](https://github.com/openwallet-foundation/credo-ts), the software on
which Veritable is built.  Future iterations of Veritable may make use of
additional DIDComm features when supported by `credo-ts`.  These features are
explained below.

### Threaded responses
In the first iteration of Veritable, the `submit_query_request` and
`submit_query_response` messages are tied together 'manually' using the `id`.  Future
iterations of Veritable may use [DIDComm
threads](https://identity.foundation/didcomm-messaging/spec/#threads).

If no query response is sent before expiry of the query request message, the
querier can call `submit_query_request` again in the same thread.

### Confidence scores
Future Veritable iterations may include a notion of query response 'confidence', which may vary with
- the percentage of partial queries issued for which a response was obtained;
- the perceived reliability of the data (e.g. if it has been cached -- see
  [below](#response-caching)).

### Adding signatures
DIDComm allows messages to be signed to provide non-repudiation.
This would be helpful when auditing messages since there is cryptographic proof
that a node provided a given response.  However, it possibly introduces privacy
concerns, so doing so requires careful thought and is not planned for Veritable
at this time.

### Claiming responses
It may be useful for nodes to be able to lay claim to responses.  They may wish
to do this to prove (at a later date) that they provided a query response at a
given point in time.  The challenge is to do this without compromising the
node's privacy. 

We now describe one possible mechanism using `did:key`, which may be implemented
in future Veritable iterations.  Below, we assume ECDSA with curve P256 and
SHA256 is used.

#### Methodology
For each query response, the responding node generates a single-use `did:key`
DID and signs the query response body.  When a node constructs its response, it
creates its response object in the usual way, modifies it as described below,
and then signs the resulting data object.

It is suggested that the [JOSE framework](https://jose.readthedocs.io) should be
used to add signatures as it is a well-established and robust framework and
simplifies the route to using JSON Web Encryption in the future, which may be
desirable.  Encryption could be used to conceal intermediate query responses
from all but the original querier.

#### Signing process
JSON Web Signatures comprise a header, the payload, and the signature, base64url
encoded and concatenated with '`.`'.  The payload is represented using base64url
because this avoids the problem that fields in JSON objects may be written and
parsed in any order, which could cause signature verification to fail.

The process for generating signatures for Veritable messages is as follows:

1. Construct the `params` field as for a 'normal' response.
```json
{
  "id": "03f3e4e7-47b6-4bf6-9076-ad4cc361b52c",
  "createdTime": 1730994619,
  "expiresTime": 1731599419,
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/request/0.1",
  "data": {
    "subjectId": {
      "idType": "product_and_quantity",
      "content": {
        "productId": "Test1",
        "quantity": 1
      }
    }
  }
}
```
2. Generate a new `did:key`.
```
did:key:z2dmzD81cgPx8Vki7JbuuMmFYrWPgYoytykUZ3eyqht1j9KbsnU4EL71rner8MnzLDPXnFuUoEBS7YERXEwX5enQ1WyEoNnARK6EmFVPbZhpTYj1BJX93cc1vTyUdqJYU948cdtJM4u2CMJXYiWM4c7M1UMKZ7k31ZiUxNyDmzDdQFuVmv
```
3. Add the field `did` to the `params` structure and populate it with
   `did:key` identifier as a string
```json
{
  "id": "03f3e4e7-47b6-4bf6-9076-ad4cc361b52c",
  "createdTime": 1730994619,
  "expiresTime": 1731599419,
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/total_carbon_embodiment/request/0.1",
  "data": {
    "subjectId": {
      "idType": "product_and_quantity",
      "content": {
        "productId": "Test1",
        "quantity": 1
      }
    }
  },
  "did": "did:key:z2dmzD81cgPx8Vki7JbuuMmFYrWPgYoytykUZ3eyqht1j9KbsnU4EL71rner8MnzLDPXnFuUoEBS7YERXEwX5enQ1WyEoNnARK6EmFVPbZhpTYj1BJX93cc1vTyUdqJYU948cdtJM4u2CMJXYiWM4c7M1UMKZ7k31ZiUxNyDmzDdQFuVmv"
}
```
and then encode this object using UTF-8 and base64url:
```base64
ewogICJpZCI6ICIwM2YzZTRlNy00N2I2LTRiZjYtOTA3Ni1hZDRjYzM2MWI1MmMiLAogICJjcmVhdGVkVGltZSI6IDE3MzA5OTQ2MTksCiAgImV4cGlyZXNUaW1lIjogMTczMTU5OTQxOSwKICAidHlwZSI6ICJodHRwczovL2dpdGh1Yi5jb20vZGlnaWNhdGFwdWx0L3Zlcml0YWJsZS1kb2N1bWVudGF0aW9uL3RyZWUvbWFpbi9zY2hlbWFzL3Zlcml0YWJsZV9tZXNzYWdpbmcvcXVlcnlfdHlwZXMvdG90YWxfY2FyYm9uX2VtYm9kaW1lbnQvcmVxdWVzdC8wLjEiLAogICJkYXRhIjogewogICAgInN1YmplY3RJZCI6IHsKICAgICAgImlkVHlwZSI6ICJwcm9kdWN0X2FuZF9xdWFudGl0eSIsCiAgICAgICJjb250ZW50IjogewogICAgICAgICJwcm9kdWN0SWQiOiAiVGVzdDEiLAogICAgICAgICJxdWFudGl0eSI6IDEKICAgICAgfQogICAgfQogIH0sCiAgImRpZCI6ICJkaWQ6a2V5OnoyZG16RDgxY2dQeDhWa2k3SmJ1dU1tRllyV1BnWW95dHlrVVozZXlxaHQxajlLYnNuVTRFTDcxcm5lcjhNbnpMRFBYbkZ1VW9FQlM3WUVSWEV3WDVlblExV3lFb05uQVJLNkVtRlZQYlpocFRZajFCSlg5M2NjMXZUeVVkcUpZVTk0OGNkdEpNNHUyQ01KWFlpV000YzdNMVVNS1o3azMxWmlVeE55RG16RGRRRnVWbXYiCn0
```
4. Create the JWS Protected Header object
```json
{"typ":"JWT", "alg":"ES256"}
```
and encode it using UTF-8 and base64url:
```
eyJ0eXAiOiJKV1QiLCAiYWxnIjoiRVMyNTYifQo
```
5. Sign the Protected Header concatenated with the
   payload by '`.`' (i.e. `0x2E`) as a string of ASCII characters, i.e. the following:
```
eyJ0eXAiOiJKV1QiLCAiYWxnIjoiRVMyNTYifQo.ewogICJpZCI6ICIwM2YzZTRlNy00N2I2LTRiZjYtOTA3Ni1hZDRjYzM2MWI1MmMiLAogICJjcmVhdGVkVGltZSI6IDE3MzA5OTQ2MTksCiAgImV4cGlyZXNUaW1lIjogMTczMTU5OTQxOSwKICAidHlwZSI6ICJodHRwczovL2dpdGh1Yi5jb20vZGlnaWNhdGFwdWx0L3Zlcml0YWJsZS1kb2N1bWVudGF0aW9uL3RyZWUvbWFpbi9zY2hlbWFzL3Zlcml0YWJsZV9tZXNzYWdpbmcvcXVlcnlfdHlwZXMvdG90YWxfY2FyYm9uX2VtYm9kaW1lbnQvcmVxdWVzdC8wLjEiLAogICJkYXRhIjogewogICAgInN1YmplY3RJZCI6IHsKICAgICAgImlkVHlwZSI6ICJwcm9kdWN0X2FuZF9xdWFudGl0eSIsCiAgICAgICJjb250ZW50IjogewogICAgICAgICJwcm9kdWN0SWQiOiAiVGVzdDEiLAogICAgICAgICJxdWFudGl0eSI6IDEKICAgICAgfQogICAgfQogIH0sCiAgImRpZCI6ICJkaWQ6a2V5OnoyZG16RDgxY2dQeDhWa2k3SmJ1dU1tRllyV1BnWW95dHlrVVozZXlxaHQxajlLYnNuVTRFTDcxcm5lcjhNbnpMRFBYbkZ1VW9FQlM3WUVSWEV3WDVlblExV3lFb05uQVJLNkVtRlZQYlpocFRZajFCSlg5M2NjMXZUeVVkcUpZVTk0OGNkdEpNNHUyQ01KWFlpV000YzdNMVVNS1o3azMxWmlVeE55RG16RGRRRnVWbXYiCn0
```
6. The resulting signature is then encoded using base64url:
```
AehxPcfvrnmF_FdJj05_Yum11toS_O1KolXAGc2l08T-KpTBL-A-LuuvMzNw-bGfyr2uvaQr7NbFxvTWlef5xQ
```
7. The header/payload string is concatenated with the signature, separated using '`.`', to produce the JWS, and this is placed in the message as follows:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_response",
  "params": {
    "signedResponse": "eyJ0eXAiOiJKV1QiLCAiYWxnIjoiRVMyNTYifQo.ewogICJpZCI6ICIwM2YzZTRlNy00N2I2LTRiZjYtOTA3Ni1hZDRjYzM2MWI1MmMiLAogICJjcmVhdGVkVGltZSI6IDE3MzA5OTQ2MTksCiAgImV4cGlyZXNUaW1lIjogMTczMTU5OTQxOSwKICAidHlwZSI6ICJodHRwczovL2dpdGh1Yi5jb20vZGlnaWNhdGFwdWx0L3Zlcml0YWJsZS1kb2N1bWVudGF0aW9uL3RyZWUvbWFpbi9zY2hlbWFzL3Zlcml0YWJsZV9tZXNzYWdpbmcvcXVlcnlfdHlwZXMvdG90YWxfY2FyYm9uX2VtYm9kaW1lbnQvcmVxdWVzdC8wLjEiLAogICJkYXRhIjogewogICAgInN1YmplY3RJZCI6IHsKICAgICAgImlkVHlwZSI6ICJwcm9kdWN0X2FuZF9xdWFudGl0eSIsCiAgICAgICJjb250ZW50IjogewogICAgICAgICJwcm9kdWN0SWQiOiAiVGVzdDEiLAogICAgICAgICJxdWFudGl0eSI6IDEKICAgICAgfQogICAgfQogIH0sCiAgImRpZCI6ICJkaWQ6a2V5OnoyZG16RDgxY2dQeDhWa2k3SmJ1dU1tRllyV1BnWW95dHlrVVozZXlxaHQxajlLYnNuVTRFTDcxcm5lcjhNbnpMRFBYbkZ1VW9FQlM3WUVSWEV3WDVlblExV3lFb05uQVJLNkVtRlZQYlpocFRZajFCSlg5M2NjMXZUeVVkcUpZVTk0OGNkdEpNNHUyQ01KWFlpV000YzdNMVVNS1o3azMxWmlVeE55RG16RGRRRnVWbXYiCn0.AehxPcfvrnmF_FdJj05_Yum11toS_O1KolXAGc2l08T-KpTBL-A-LuuvMzNw-bGfyr2uvaQr7NbFxvTWlef5xQ"
  },
  "id": "0d6e79c4-48e2-4953-945f-c5bfdcbe82aa"
}
```

#### Discussion
At a later point in time, the node can prove they signed the message by signing
a random challenge value.  Since new `did:key`s are generated for each response,
there is no privacy loss.

It is important to note that these signatures do _not_ enable nodes to prove
they did _not_ give a particular response since the `did:key` objects are not
themselves authenticated (so signed messages can be repudiated).  As such, they
cannot be used for auditing purposes, only for nodes to claim responses provided
nodes in the supply chain include the signatures and keys in the
`partialResponses` field of their own responses.  Being able to identify nodes
from signatures would potentially be a privacy concern and would have to be
carefully thought through.

Nodes are incentivised to allow their children to lay claim to responses because
this allows them to shift blame in the event of faulty information being
revealed. However, because the keys are not authenticated, child nodes can
always deny they gave a response.

If a node refuses to admit wrongdoing, an auditor does not know whether the node
or its parent was dishonest: either the child node was dishonest and will not
claim responsibility, or the parent was dishonest and changed the child node's
honest response.  From the auditor's point of view, it has identified
approximately where in the supply chain the false information was provided. From
the parent and child nodes' points of view, one of them has caused the auditor
to blame the other instead of themselves so there are likely to be implications
for the business relationship (such as the parent ceasing to use the child as a
supplier).

Note that the `did:key` identifier is in scope of the signature as proof of
possession.

### Response caching
Some query responses may be statements of immutable fact, or may be applicable
for long periods of time (e.g. ISO 9001 compliance could be re-certified on an
annual basis).  Future versions of Veritable may support caching responses
to reduce network traffic.

## Other considerations
### CBOR
DIDComm does not support CBOR natively, so will not be supported in Veritable.