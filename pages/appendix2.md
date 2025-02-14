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
* ID is taken from the DocumentReference resource retrieved from an NRL lookup (whether [through BaRS Proxy](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_2_0#get-/DocumentReference) or [direct from NRL](https://digital.nhs.uk/developer/api-catalogue/national-record-locator-fhir/v3/consumer#get-/DocumentReference)) the ID can be taken from either (in both cases 8c63d621-4d86-4f57-8699-e8e22d49935d in the example):
  * The end of the content[0].attachment.url attribute.
  * The value  of the identifier with system of: https://fhir.nhs.uk/Id/BaRS-Identifier

Example: `https://api.service.nhs.uk/booking-and-referral/FHIR/R4/Appointment/8c63d621-4d86-4f57-8699-e8e22d49935d`

A __GET__ request is sent to this URL

Further actions to 'manage' the appointment are out of scope for MVP and are therefore TBC

<a name="authentication"></a>
## Authentication
* Between Wayfinder Aggregator and BaRS Proxy is OAuth 2.0 Token Exchange
* Between BaRS Proxy and Epic is Mutual TLS. Details of the Root and SubCA that issue the BaRS proxy client certificate in the "int" environment are [published on the NHS England Path to Live Environments pages](https://digital.nhs.uk/services/path-to-live-environments/integration-environment#rootca-and-subca-certificates)

<a name="payload"></a>
## Response Payload
FHIR UK Core R4 Appointment as defined [here](https://simplifier.net/packages/hl7.fhir.r4.core/4.0.1/files/83384)

<a name="headers"></a>
## Request Headers
| Name | Description | Type |
| --- | --- | --- |
| `X-Request-Id` | An ID for the request, this is mirrored back in the response. | UUID |
| `X-Correlation-Id` | An ID that can be used internally by the client to correlate to internal business processes. | UUID |
| `NHSD-Target-Identifier` | This is the JSON of the Identifier with system of https://fhir.nhs.uk/Id/dos-service-id - this is obtained from the `identifier` array of the DocumentReference that points to the Appointment | Base64 encoded JSON object |
| `NHSD-End-User-Organisation` | Requesting Organization described in an object based on a FHIR 'Organization' resource (Base64 encoded JSON). This header SHALL be populated when the request is not initiated by the App user / Patient. | Base64 encoded JSON object |
| `NHSD-ID-Token` | This is the ID token of the App user / Patient. The BaRS proxy does NOT validate the token and passes it through to the target system | JWT |


Example (unencoded) `NHSD-Target-Identifier`:
```json
{
  "value": "2000072491",
  "system": "https://fhir.nhs.uk/Id/dos-service-id"
}
```

Example (Base64 encoded) `NHSD-Target-Identifier`:

`ewrCoCAidmFsdWUiOsKgIjIwMDAwNzI0OTEiLArCoCAic3lzdGVtIjrCoCJodHRwczovL2ZoaXIubmhzLnVrL0lkL2Rvcy1zZXJ2aWNlLWlkIgp9`



Example (unencoded) `NHSD-End-User-Organisation`:
```json
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

![Integration via BaRS Proxy](images/Figure4.svg)
Figure 4. Integration via BaRS Proxy 


<a name="details"></a>
## Further low-level technical detail

Before returning data, Epic MUST [validate the supplied ID Token](token_validation.md).

Appointment resource [field by field assessment](appointment_fields.md).

[See Attached](pdfs/3.pdf)
