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
in future Veritable iterations.

For each query response, the responding node generates a single-use `did:key`
DID and signs the query response body. The signatures are included in the query
response of parents (at the same level as the `data` field).  The `id` field in
the objects inside the `partialQueries` array is no longer populated as a UUID,
but as the `did:key` identifier, e.g.
`did:key:DRFlgWZAAG5SVQ8dWSjuS8y-Pz3bqgc0_QmrGqcg9SI`.

At a later point in time, the node can prove they signed the message by signing
a random challenge value.  Since new `did:key`s are generated for each response,
this is no less anonymous than the current DIDComm message `id` approach.

It is important to note that these signatures do _not_ enable nodes to prove
they did _not_ give a particular response since the `did:key` objects are not
themselves authenticated (so signed messages can be repudiated).  As such, they
cannot be used for auditing purposes, only for nodes to claim responses provided
nodes in the supply chain include the signatures and keys in the
`partialResponses` field of their own responses.  Being able to identify nodes
from signatures would potentially be a privacy concern and would have to be
carefully thought through.

### Response caching
Some query responses may be statements of immutable fact, or may be applicable
for long periods of time (e.g. ISO 9001 compliance could be re-certified on an
annual basis).  Future versions of Veritable may support caching responses
to reduce network traffic.

## Other considerations
### CBOR
DIDComm does not support CBOR natively, so will not be supported in Veritable.