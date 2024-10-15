# Audit Query

## Query purpose
Query response audit is a core feature of Veritable.  A Veritable audit cannot
necessarily definitively identify the source of incorrect information.  This is
in part because Veritable aims to provide a degree of anonymity, since
information about supply chain structure is potentially sensitive from both
security and commercial perspectives.

Additionally, Veritable cannot deal with the case that a supplier conceals which
of their own suppliers is to blame for incorrect information.  They could choose
to do so if they know the malicious supplier is (unfortunately for the OEM) an
irreplaceable part of the supply chain.  The OEM may be able and willing to take
the reputational damage in place of their supplier for the sake of maintaining
their manufacturing capability.

Nevertheless, Veritable query responses are designed to allow auditors to
identify _some_ party to blame if query responses are found to be incorrect. The
auditor determines that the blame lies with the identified node or with one of
its suppliers; it is up to the identified node to take the blame or to provide
evidence that one of their suppliers is to blame.

Audit messages in Veritable are just queries of a specific type.  The responses
of these queries may necessitate further investigation by the auditor but this
is out of scope once the parties have been identified.

## Query request

### Schema
The schema for the `data` field of an audit query request is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query response schema for ISO 9001 compliance",
  "type": "object",
  "properties": {
    "path": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uuid"
      }
    },
  },
  "required": [ "path" ]
}
```

### Example
An audit query request for a subject with ID `39C6F7EF` is as follows:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/request/0.1",
    "createdTime": 1704067200,
    "data": {
      "path": [
        "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
        "81660576-653f-433a-9eae-c42a5164575e",
        "8825b9f9-f136-4ca6-aab0-62a50d128d33",
      ]
    }
  }
}
```

## Query response
### Schema
The schema for the `data` field of an audit query response is as follows:
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable query response schema for ISO 9001 compliance",
  "type": "object",
  "properties": {
    "nodeId": {
      "type": "string"
    },
  },
  "required": [ "nodeId" ]
}
```

The `nodeId` field is populated in such a way that an auditor can establish a
connection with the node.  For example, this could be a DID, or a company name,
number and address.  The details are not specified in the first iteration of
Veritable.

### Example
An audit query response reporting a node with ID
`did:web:example:com:well_known` is as follows:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "dbbab830-d2f9-4b6c-8168-98250e5c09b7",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/response/0.1",
    "createdTime": 1704067200,
    "data": {
      "nodeId": "did:web:example:com:well_known"
    }
  }
}
```

## Audit process

### Initiating the audit
An audit is initiated when information discovered out of band determines that
information received in a query response is incorrect.

For example, suppose a component is reported in Veritable to be able
to support a particular shear stress tolerance but is found to break
during routine testing of a vehicle that integrates that component.

The OEM looks up the Veritable query response that contains information
pertaining to the stress tolerance and initiates the audit.  The OEM must
have stored the query response to do this, e.g. 
alongside the invoice/procurement forms.

The OEM may itself audit query responses.  The remainder of this document
assumes the OEM is the auditor.  If this is not the case, the auditor either
needs to control its own Veritable node, or will require (e.g.) an OEM to submit
audit queries on its behalf.

Suppose the ISO 9001 response from the [ISO 9001 message type
documentation](../../iso_9001/0.1/README.md) is being audited as it turns out
that subject with ID `7CA09D81` was not actually certified.  The audit uses the
top-level response message to look up the `id` for the message containing
information for `7CA09D81` and determines the path of `id`s from the top-level
to this partial query response.  This is determined to be:

- `dbbab830-d2f9-4b6c-8168-98250e5c09b7`
  - `81660576-653f-433a-9eae-c42a5164575e`
    - `8825b9f9-f136-4ca6-aab0-62a50d128d33`

### Processing the query
The OEM receives the message, retrieves the query response message with `id`
`dbbab830-d2f9-4b6c-8168-98250e5c09b7` and looks up the identity of the
Veritable node that sent the message with `id`
`81660576-653f-433a-9eae-c42a5164575e`.  In this case, it was Bob.  Recall in [Veritable
messaging](../../../0.1/README.md#state) that it was required that nodes
maintain state to record which Veritable nodes were associated with which `id`s.

It then sends a partial query to that node:
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "256b5adc-effe-4e40-b471-848dd0ee557f",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_types/audit/request/0.1",
    "createdTime": 1704067230,
    "data": {
      "path": [
        "81660576-653f-433a-9eae-c42a5164575e",
        "8825b9f9-f136-4ca6-aab0-62a50d128d33",
      ]
    }
  }
}
```

The receiving node does the same processing to obtain the ID of the node that
sent the message `8825b9f9-f136-4ca6-aab0-62a50d128d33`, say, Charlie.  It then
sends this ID to the auditor, since it is the parent node of the node that sent
the message being targeted by the audit.

### Resolving the dispute
At this point the auditor can establish a connection to a node they believe to
be cheating.  The auditor and node in question can now communicate over DIDComm
to resolve the dispute.

This resolution could be 'technical', e.g. by requesting the same information as
was requested in the original query and comparing it, or non-technical, like
requesting an out-of-band conversation about the discrepancy, and investigating
database content manually.  The resolution process is out of scope of Veritable.

If for some reason the parent node is unwilling to divulge its child node's
identity, then the auditor will 'blame' the parent node and must resolve the
dispute with the auditor.

If the identity of the child node is revealed and the auditor successfully
establishes a connection, then further queries can be initiated rooted at that
node depending on the nature of the problem is (e.g. whether data is incorrect
or just invalid).

## State
Note that the 'top-level' querier (i.e. the node that submits a query request
that does not have a parent query) should store query responses for audit
purposes (since they contain the paths of message `id`s), but 'intermediate'
nodes do not need to store messages since the tree of responses is provided in
the top-level query response.

Intermediate nodes do, however, need to maintain state to match message `id`
values to Veritable node IDs so they can forward partial audit query requests
and/or identify nodes that gave particular responses for the auditor.  This
requirement was discussed in [Veritable
messaging](../../../0.1/README.md#state).

## Dealing with malicious behaviour

### Repudiating messages
There is no application-layer authentication on Veritable messages (see
[Future work](../../../0.1/README.md#future-work)) so any information in query
responses can be changed by malicious nodes.

If a node changes values its children provided in partial query responses when
providing a response upstream, during dispute resolution of the audit process,
the child nodes may deny they provided different information.  There is no way
for them to assert that they provided accurate information that was changed by a
parent since messages are repudiable.

However, falsely accusing a parent may mean they are removed as a supplier, so
they are incentivised to be honest.  Additionally, a parent node found to be
using a supplier node that provides false information may be receive punitive
action administered by the auditor, and may therefore be dropped as a supplier
by its own suppliers, incentivising nodes not to change partial query responses.

### Loss of node ID / query ID database
When collating partial query responses, each node is responsible for storing the
mapping between query `id` and Veritable node ID so that audit query can trace
previous responses to Veritable nodes.  If this database is lost (intentionally
or otherwise), then there is no direct way to trace the source of a query.

If a node honestly loses state and cannot identify the child node that gave a
particular response `id`, the node that lost its state can broadcast the `id`
'below' the missing `id` in the path to the audit subject to all of its
children, who can claim the response and continue the audit.  If none of its
children claim the response, the node is treated as malicious.  (It is unlikely
that two nodes will lose state at the same time.  State recovery is left as
future work.)

If the parent node of the node that provided information being audited loses
state, this node is treated as malicious since its supplier provided false
information and cannot be identified. 

Note that the dispute resolution process in this situation is equivalent to the
resolution required for nodes who might choose to change `id`s in message
responses to obfuscate query responses (rather than values of responses as was
the focus in the previous subsection), and there is no clear value in doing so.

## Future work
### State recovery
If a node loses its database mapping Veritable nodes to messages they have sent,
it should be possible for them to reconstruct their state by communicating with
their parents and children.