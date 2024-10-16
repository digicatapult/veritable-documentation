# Veritable Documentation

Documentation for the Veritable Framework.

`Veritable` is an open-source framework for implementing and deploying enterprise SSI (Self-Sovereign Identity) solutions. It is designed to be modular, extensible, and easy to use.  SSI solutions allow for the creation of digital identities, which can be used to authenticate and authorize entities, and to exchange credential data with other users and organizations.  SSI solutions are based on the [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/) and [Decentralized Identifiers](https://www.w3.org/TR/did-core/) standards.

This solution has features such as:
* A modular architecture, allowing for the use of different DID storage methods.
* OpenAPI-based RESTful APIs for easy integration with other systems.

## READMEs
This repo includes READMEs that explain concepts within Veritable:
* [Architecture](docs/architecture.md)

It also provides details of the Veritable messaging protocol and schemas for
Veritable messages:
* [Veritable messaging protocol](schemas/veritable_messaging/0.1/README.md)
* [Veritable query types]
  * [ISO 9001](schemas/veritable_messaging/query_types/iso_9001/0.1/README.md)
  * [Total Carbon Embodiment](schemas/veritable_messaging/query_types/total_carbon_embodiment/0.1/README.md)
  * [Audit](schemas/veritable_messaging/query_types/audit/0.1/README.md)

## Repositories
### Active Veritable Repositories
These repositories contain code being actively maintained as a part of the Veritable project.

| Repository | Description |
| --- | --- |
| [veritable-documentation](https://github.com/digicatapult/veritable-documentation) | Documentation for the Veritable Framework. |
| [veritable-cloudagent](https://github.com/digicatapult/veritable-cloudagent) | A cloud agent and cloud wallet API for the Veritable framework. |
| [veritable-portal](https://github.com/digicatapult/veritable-portal) | A web portal for the Veritable framework. |

### Inactive Veritable Repositories
These repositories contain code that is no longer being actively maintained as a part of the Veritable project.

| Repository | Description |
| --- | --- |
| [veritable-acapy-proxy](https://github.com/digicatapult/veritable-acapy-proxy) | Proxy API for aca-py Cloud Wallet |
| [veritable-verifier](https://github.com/digicatapult/veritable-verifier) | Front-end for the Verifier persona in Veritable |
| [veritable-holder](https://github.com/digicatapult/veritable-holder) | Front-end for the Holder persona in Veritable |
| [veritable-issuer](https://github.com/digicatapult/veritable-issuer) | Front-end for the Issuer persona in Veritable |
| [veritable-authority](https://github.com/digicatapult/veritable-authority) | Front-end for the Authority persona in Veritable |
| [veritable-poc](https://github.com/digicatapult/veritable-poc) | Proof of concept demonstrator for the Veritable SSI solution |
