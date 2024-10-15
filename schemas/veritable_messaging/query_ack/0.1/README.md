# Veritable Acknowledgement
The Veritable acknowledgement message is used to acknowledge receipt of some
request message, usually a DRPC request message.


## Schema
```json
{
  "$id": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "The Veritable acknowledgement message",
  "type": "object",
  "properties": {
    "returnCode": {
      "type": "number"
    }
  },
  "required": [ "returnCode" ]
}
```

## Example
```json
{
  "jsonrpc": "2.0",
  "method": "submit_query_request",
  "params": {
    "id": "eb8e942f-45f6-45b1-8cf4-2e4e665491f6",
    "type": "https://github.com/digicatapult/veritable-documentation/tree/main/schemas/veritable_messaging/query_ack/0.1",
    "created_time": 1704067200,
    "expires_time": 1704067260,
    "data": {
      "returnCode": 0
    }
  }
}
```

The return codes currently defined are as follows:

| Number | Meaning |
| - | - |
| 0 | No error |
| 1 | Query unlikely to be answered before specified `expires_time` |