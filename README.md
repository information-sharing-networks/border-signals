# Border Signals Protocol (draft)

[Principles](#principles) |
[Standards](#standards) |
[Use Cases](#use-cases) |
[Overview](#overview) |
[Tools](#tools) |
[Prototype](#prototype-implementation) |

[Interop examples](interop-examples)

This document describes the work to create a first draft of a protocol to allow trade and government systems to exchange signals indicating the status of events related to buying, selling and moving goods internationally.  This work is experimental.

The idea is to design an API specification that will meet the requirements of a couple of use cases (below) and to use the specification to create a prototype server and associated documentation that can be used by others as a way to collect feedback and improvements.

This README will be used to describe how to use the protocol and provide links to any technical guidance.   

All the work will be done in the open and hosted in github. Please feel free to start discussions and raise pull requests.

# Principles
* Signals should be simple: this protocol is intended to fill a gap, not replace existing APIs Trade systems use to exchange data, nor replace declaration APIs provided by the government.
* the protocol should use open standards
* It should be possible to create a server/client for prototyping and testing purposes with minimal technical investment
* The protocol should deal with version control so that new signals can be created without disrupting existing implementations.

# Standards
* The pub/sub API will be specified using [AsyncAPI](https://www.asyncapi.com/)
* Per gov [standards](https://www.gov.uk/government/publications/recommended-open-standards-for-government/describing-restful-apis-with-openapi-3), if REST APIs are needed they will be specified using [OpenAPI](https://www.openapis.org/)
* Payloads (the content of the signal message) will be exchanged as JSON.  Content can be validated using [JSON Schema](https://json-schema.org/).  There are tools available to automatically create the schemas based on the API specs, see [tools](#tools) below.


OpenApi and AsyncAPI are open standards managed by the Linux Foundation.  The two standards are closely related: AsyncAPI builds on OpenAPI and allows description of things like channels and messages.  

The specs can be written as JSON or YAML, but we prefer YAML for the parts on the protocol that will be of most interest to the business domain experts (for instance the signal payloads) as they will not necessarily have a technical background and YAML is easier to read/amend than JSON.
  
# Use Cases
TODO

# Overview

## Underlying protocol
RESTful APIs will use HTTP
event based APIS TODO 

## signal payloads
A signal payload definition describes the structure of a signal being exchanged between systems.   A simple payload definition might look something like this:
```yaml
signalType:
  type: string
  description: the type of message being sent
  example:  Business document issued
documentType:
  type: string
  description: the kind of document that was issued
  example: Bill of lading
issuedByEORI:
  type: string
  description: EORI of issuer
  example: GB123456789012
location:
  type: string
  description:  url to collect a copy of the document
  example: https://api.example.com/docs/XYZ
  ```

## signal lifecycle
TODO
* priority
* expiry
* revocation
* cancelation
* status

## Publish/Subscribe
the proposed protocol will adopt a "publish/subscribe" model where participating systems will publish signals related to events as they happen and other systems will subscribe to the message channels that they are interested in.

Although using this type of architecture creates some additional technical challenges compared to services that connect synchronously we think that in the long run the benefits will make the extra work worthwhile.  Potential benefits include:

* Improved transparency: signals can go in any direction, creating the potential for new workflows.  For instance new workflows could be created around gov controls that allowed signals to be generated alerting traders about pending inspections or where more data is required before goods can be cleared.
* Improved data quality and timeliness: because information about trade events can be published as they occur, data can be delivered in stages rather than waiting for all the data needed for a document to be complete.   Because the data can be published directly by the originating organisation, rather than via some intermediary consolidation/transformation process,  there will be fewer opportunities for data to be mangled/incorrectly mapped/dropped etc
* Improved cooperation between government agencies (both UK and international) who could also use the protocol to share information with one another without the need to organise complicated data integration projects, for instance by sharing  results of decisions related to trade, or the results of due dilligence related to checks and inspections.

it should be possible to implement the publisher API using a variety of pub/sub messaging solutions (RabbitMQ, Kafka, Firestore etc)

## signal orchestration
TODO
* How we record an event/decision/opinion 
* How we store status/entity/org information (eg a wallet)
* How end users monitor changes (alerting systems/Dashboards etc)
* How we route messages (broker)

## Identity and access management
TODO
* registration (?)
* authentication
* permissions  (can we reuse an entity/org access control concept from one of the EOT solutions?) 

# tools
Using the AsyncAPI and OpenAPI standards to specify the API used by the protocol means that sine work can be automated, including:

* creation of documentation that describes the technical details of what the API does
* creation of mock servers that can help with testing the design without lots of development effort
* generation of boilerplate code for language of choice (there are tools available for all the popular backend languages).
* security audits on authentication approach, identification of vulnerabilities etc.

The spec will be linted to check it is developed according to a set of design guidelines (tbd)

# API Lifecycle
TODO semantic versioning/git tags, initial thoughts on governance options
For the protocol to be useful it should be possible for trade and gov organisations to introduce new payloads without disrupting existing implemenations.  It might thereire be useful to look at making payload definitions part of arolling specification ala HTML and with an equivalent to *caniuse* being available to determine if a publisher supports a particular payload.

# Prototype Implementation
We aim to create a rudimentary prototype publisher implmemenation as part of the EOT pilot.   The sole purpose of the protototype is to help test whether the emerging protocol specfication is useful or not. It is unlikely to be maintained beyond the pilot (although any technical/implemenation detials will still be available on this github repo for reference). It is not be used for production purposes and will not publish any private or sensitive data.

For the prototype we'll borrow from the [DCSA information model](https://github.com/dcsaorg/DCSA-Information-Model) for payload definitions wherever possible.  The model, and assosciated APIs, are well documented and are largely based on international data standards (note the model was designed around  container traffic so we might need to look elsewhere for RORO specifics).

we'll follow the gov [service manual guidance](https://www.gov.uk/guidance/gds-api-technical-and-data-standards) on APIs

# interop examples
other interop implemenations to use or borrow/learn from:

borders
* [DCSA event subscription](https://app.swaggerhub.com/apis/dcsaorg/DCSA_EBL/2.0.0-Beta-2#/Shipping%20Instructions/get_v2_shipping_instructions__shippingInstructionReference_) approach (webhook)  
* NOTN (discussion pending)

Finance
* FIX - electronic trading  - very simple, pre web, well adopted but possible challenges around multiple dialects
* PSD2 (c.f open banking exchange)  provision of payment services , interesting because (EU) legislation-lead

Industry led examples
* [OpenTravel](https://opentravel.org/) - does not appear to be a web based presence for the spec, but available on request.
* [OpenReferral](https://openreferraluk.org/about-standard) - local council data exchange 
* [GTFS](https://github.com/google/transit/tree/master/gtfs-realtime/spec/en) - public transportation (an example feed - from Connecticut Department of Transportation - is hosted on the [Ably hub](https://ably.com/hub/cttransit/gtfsr)

