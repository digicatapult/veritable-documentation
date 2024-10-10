# Veritable messaging
A Veritable message is a type of DIDComm message that allows a user to ask a
supply chain question and obtain a response.

This document describes the Veritable message structure and flow.  Separate
documents describe query and response formats, which are the Veritable message
payloads.  Different query types use the same standard Veritable message
structure and flow, but the message payloads is different for different query
types (compare a compliance query with a numerical query).

## Transport
Veritable is built atop DIDComm and the reader is assumed to be familiar with
the [DIDComm
specifications](https://identity.foundation/didcomm-messaging/spec).

The DIDComm Protocol Identifier URI (PIURI) for Veritable is
`https://github.com/digicatapult/veritable-documentation/tree/main/docs/veritable_messaging/0.1`.  

Veritable uses
[DRPC](https://github.com/openwallet-foundation/credo-ts/tree/main/packages/drpc),
which is a request/response communication architecture for DIDComm based on
JSON-RPC.

## Query flow
A Veritable query consists of two messages: a query request message (asking the
question) and a query response (giving the answer):
```
  A                                         B
      query request (DRPC request)
      ---------------------------------->
      query response (DRPC response)
      <----------------------------------
```

Future iterations of Veritable may support an asynchronous exchange. See [Future
Work](#future-work).

The query request and query response are tied together using a [DIDComm
thread](https://identity.foundation/didcomm-messaging/spec/#threads).

## Veritable query message structure
Veritable currently defines three types of query: ISO 9001, Scope 3 Emissions,
and Audit.  The DIDComm Message Type URIs (MTURIs) are as follows:
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/iso_9001/response/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/scope_3/response/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/request/0.1`
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1`

This subsection provides templates for different kinds of query request and
response messages.  All DIDComm headers not in the templates are optional for
Veritable messages.

### Query request template
A query request message follows the following query request template:
```json
{
  "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/<query type>/request/0.1",
  "from": "did:example:alice",
  "to": ["did:example:bob"],
  "created_time": 1704067200,
  "expires_time": 1704067260,
  "body": {
    // Veritable query as described in the query_type document
  }
}
```
The `body` field is populated according to the query type (identified by the
MTURI value in the `type` field).  These data structures are defined in the
MTURIs listed above.

The `expires_time` field should be set to a 'reasonable' timeframe in which the
responder is expected to be able to process the query request message (including
decryption and validation of any signatures, if used) and return a query response.  By
default, the `expires_time` is set to 60 seconds after the `created_time`.
Asynchronous logic is left as future work: see [Future
work](#asynchronous-communication).

If the sender does re-send the query request message before `expires_time` has
passed, the querier must attach the new query request to the existing query
request thread by populating the `thid` field with `id` of the original query
request message in the new query request. This prevents the responder from
attempting to answer multiple copies of the query: the responder should consider
new query requests that are _not_ part of an existing thread to be separate
queries.

### Query response template
A query response message follows the following query response template:
```json
{
  "id": "ff63852a-1dc9-4795-a796-79d89cb1b56c",
  "thid":  "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/<query type>/response/0.1",
  "from": "did:example:bob",
  "to": [ "did:example:alice" ],
  "created_time": 1704067203,
  "body": {
    // Veritable message as described in this document set
    ...
    "partialResponses": []
  }
}
```
The `body` field is populated according to the query type (identified by the
MTURI value in the `type` field).  These data structures are defined in the
MTURIs listed above.

Note the thread ID field `thid`, which is populated using the query request
message's `id` field.  This ties the query request to the query response.

The `partialResponses` field is described below.

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
done by the 'top-level' querier however they find useful, and it reduces the
amount of state each node has to store for audits.

The general process for constructing a query response from partial queries is to
extract the `id` and the `body` fields from the DIDComm message and create an
object
```json
{
  "id": ...,
  "body": ...
}
```
and then add this object to the `partialResponses` array.

## State
For auditing purposes (in the first version of Veritable), each node should
store a database holding pairs (Veritable node ID, message ID).  The Veritable
node ID is a 'permanent' identifier (such as a company name and number) for the
message sender, and the message ID is the value of the `id` field in the DIDComm
message.  The reasoning is explained in more detail in the [Audit
query](../query_types/audit/0.1/README.md) documentation.

## Future work
### Asynchronous communication
Asynchronous communication may be supported in the future by using _query
acknowledgement_ messages.  This section gives an overview of how they are
expected to work.

When a responder receives a query request, if it can answer immediately
then it replies with a query response, and otherwise it should reply with a
query acknowledgement.

The `expires_time` DIDComm header can be used in query request messages to
indicate the response timeout by which a query acknowledgement or query response
should be sent.  The timeout as the difference between the `created_time` and
`expires_time` should not be significantly larger than the round-trip time,
since acknowledgements should be sent immediately.

If the sender does not receive a query response or acknowledgement before
`expires_time` passes, the sender may send a new query request (in the same
message thread).  If the responder receives multiple query requests with the
same `thid` (e.g. due to some buffering/cache/network latency issues), it need
only respond to the latest one.

The query acknowledgement message could use the `expires_time` field to indicate when a query
response should be expected.  This may be important where a responder must
decompose a query into a large number of partial queries for which the responder
must itself wait for responses.

The MTURI for a query acknowledgement is:
- `https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1`

The `query_ack` message has the following form (regardless of the query type)
```json
{
  "id": "ff63852a-1dc9-4795-a796-79d89cb1b56c",
  "thid": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
  "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1",
  "from": "did:example:bob",
  "to": [ "did:example:alice" ],
  "created_time": 1704067265,
  "body": {}
}
```

Note that the query acknowledgement message is only ever sent within a query
request thread.  As such, the use of the `thid` field implies that this message
is an acknowledgement of the query request with the `thid` as its `id`, so there
is no need to include the acknowledgement header `ack`, or to use the
corresponding `please_ack` header in the query request message.

A responder should always send an acknowledgement message in response to a query
request message even if it has expired, since it may be that network latency is
greater than the default timeout the querier sets.  If the querier
receives an acknowledgement for a 'previous' query request, it should consider
this acknowledgement of the query.

### Adding signatures
DIDComm allows messages to be signed to provide non-repudiation for messages.
This would be helpful when auditing messages since there is cryptographic proof
that a node provided a given response.  However, it possibly introduces privacy
concerns, so doing so requires careful thought and is not planned for Veritable
at this time.

### Claiming responses
It may be useful for nodes to be able to lay claim to responses.  They may wish
to do this to prove (at a later date) that they provided a query response at a
given point in time.  The challenge is to do this without compromising the
node's privacy. 

We now describe one possible mechanism using `did:key`.

For each query response, the responding node generates a single-use `did:key`
DID and signs the query response body. The signatures are included in the query
response of parents (along with the `body` field).  The `id` field in the objects inside the `partialQueries` array is no longer populated as a UUID, but as the `did:key` identifier, e.g. `did:key:DRFlgWZAAG5SVQ8dWSjuS8y-Pz3bqgc0_QmrGqcg9SI`.

At a later point in time, the node can prove they signed the message˜ by signing
a random challenge value.  Since new `did:key`s are generated for each response,
this is no less anonymous than the current DIDComm message `id` approach.

It is important to note that these signatures do _not_ enable nodes to prove
they did _not_ give a particular response since the `did:key` objects are not
themselves authenticated (so signed messages can be repudiated).  As such, they
cannot be used for auditing purposes, only for nodes to claim responses.  Being
able to identify nodes from signatures would potentially be a privacy concern
and would have to be carefully thought through.

## Other considerations
### CBOR
DIDComm does not support CBOR natively, so will not be supported in Veritable.