[Home](../readme.md)

# Appendix 2: Record retrieval (technical detail)

<a name="background"></a>
## Background / business process
When a request is made for an Appointment resource for a patient, that Appointment is returned.

<a name="triggers"></a>
## Trigger events
Request from NHS App (actually sent by Wayfinder Aggregator) for details (the actual FHIR Resource) of a specific Appointment.

<a name="requester"></a>
## Requester
Wayfinder Aggregator via API-M/BaRS Proxy.

<a name="responder"></a>
## Responder
NHS England BaRS Proxy API acts as an intermediary, FHIR API on Trust EPR is the actual Responder.

<a name="endpoint"></a>
## Endpoint
URL is constructed from the BaRS base URL + ID:
* BaRS base URL, [see details](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_2_0#overview--environments-and-testing)
* ID is taken from the DocumentReference resource retrieved in step 2b Appointment Lookup the ID can be taken from either (in both cases 8c63d621-4d86-4f57-8699-e8e22d49935d in the example):
  * The end of the content[0].attachment.url attribute.
  * The value  of the identifier with system of: https://fhir.nhs.uk/Id/BaRS-Identifier

Example: `https://api.service.nhs.uk/booking-and-referral/FHIR/R4/Appointment/8c63d621-4d86-4f57-8699-e8e22d49935d`

A __GET__ request is sent to this URL

Further actions to 'manage' the appointment are out of scope for MVP and are therefore TBC

<a name="authentication"></a>
## Authentication
* Between Wayfinder Aggregator and BaRS Proxy is OAuth 2.0 Token Exchange
* Between BaRS Proxy and Epic is Mutual TLS

<a name="payload"></a>
## Response Payload
FHIR UK Core R4 Appointment as defined [here](https://simplifier.net/packages/hl7.fhir.r4.core/4.0.1/files/83384)

<a name="headers"></a>
## Request Headers
| Name | Description | Type |
| --- | --- | --- |
| `X-Request-Id` | An ID for the request, this is mirrored back in the response. | UUID |
| `X-Correlation-Id` | An ID that can be used internally by the client to correlate to internal business processes. | UUID |
| `NHSD-Target-Identifier` | This is the JSON of the Identifier with system of https://fhir.nhs.uk/Id/dos-service-id | Base64 encoded JSON object |
| `NHSD-End-User-Organisation` | Requesting Organization described in an object based on a FHIR 'Organization' resource (Base64 encoded JSON). | Base64 encoded JSON object |



Example (unencoded) `NHSD-Target-Identifier`:
```
{
  "value": "2000072491",
  "system": "https://fhir.nhs.uk/Id/dos-service-id"
}
```

Example (Base64 encoded) `NHSD-Target-Identifier`:

`ewrCoCAidmFsdWUiOsKgIjIwMDAwNzI0OTEiLArCoCAic3lzdGVtIjrCoCJodHRwczovL2ZoaXIubmhzLnVrL0lkL2Rvcy1zZXJ2aWNlLWlkIgp9`



Example (unencoded) `NHSD-End-User-Organisation`:
```
{
  "resourceType": "Organization",
  "identifier": [
    {
      "https://fhir.nhs.uk/Id/ods-organization-code",
      "value": "X26"
    },
  "name": "NHS ENGLAND - X26"
  ]
}
```
Example (Base64 encoded) `NHSD-End-User-Organisation`: ewrCoCAicmVzb3VyY2VUeXBlIjogIk9yZ2FuaXphdGlvbiIsCsKgICJpZGVudGlmaWVyIjogW
wrCoCDCoCB7CsKgIMKgIMKgICJodHRwczovL2ZoaXIubmhzLnVrL0lkL29kcy1vcmdhbml6YXRpb24tY29kZSIsCsKgIMK
gIMKgICJ2YWx1ZSI6ICJYMjYiCgoKwqAgwqAgfSwKwqAgIm5hbWUiOiAiTkhTIEVOR0xBTkQgLSBYMjYiCsKgIF0KfQ==	

<a name="dataflows"></a>
## Data Flows

![Integration via BaRS Proxy](images/Figure4.png)
Figure 4. Integration via BaRS Proxy 


<a name="details"></a>
## Further low-level technical detail

[See Attached](pdfs/3.pdf)