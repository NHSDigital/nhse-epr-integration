[Home](../readme.md)

[Next - Appendix 2: Record retrieval (technical detail)](appendix2.md)

# Appendix 1: Signposting (technical detail)

<a name="background"></a>
## Background / business process
When an appointment is created or changed for a patient, this information needs to be discoverable by the NHS App. To support this, a FHIR DocumentReference resource needs to be added to NRL. This is achieved via a supporting BaRS endpoint.

<a name="triggers"></a>
## Trigger events
* Creation of a new appointment for a Patient
* Change to an existing appointment
* Cancellation of an existing appointment

<a name="requester"></a>
## Requester
Trust TIE or EPR

<a name="responder"></a>
## Responder
NHS England BaRS API [^1]

<a name="endpoint"></a>
## Endpoint
The actual endpoints [^2] are defined in [the BaRS documentation](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0) under the Document Reference section.

On creating a new Appointment, send a POST request as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#post-/DocumentReference)

On change of an existing Appointment, send a PUT request as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#put-/DocumentReference/-id-) (the ID used is the ID returned when the appointment was created).

On cancellation of an existing Appointment either (Preferred) an update as above can be made (to change status to cancelled) or the DocumentReference can be deleted by making a DELETE call as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#delete-/DocumentReference/-id-) (the ID used is the ID returned when the appointment was created)

<a name="authentication"></a>
## Authentication
Uses Application Restricted as defined [here](https://digital.nhs.uk/developer/guides-and-documentation/security-and-authorisation/application-restricted-restful-apis-signed-jwt-authentication)

<a name="payload"></a>
## Payload
The payload for POST and PUT is a standard FHIR R4 DocumentReference with some specific restrictions defined [here](https://digital.nhs.uk/developer/api-catalogue/national-record-locator-fhir/v3/producer#post-/DocumentReference)

The restrictions will be determined by the BaRS programme, but in general are:

__subject MUST be a Patient reference using a valid NHS number. e.g.__
```json
"subject": {
  "identifier": {
    "system": "https://fhir.nhs.uk/Id/nhs-number",
    "value": "3495456481"
  }
}
```

__custodian MUST be an Organization reference using a valid ODS code, agreed during onboarding. e.g.__
```json
"custodian": {
  "identifier": {
    "system": "https://fhir.nhs.uk/Id/ods-organization-code",
    "value": "Y05868"
  }
}
```

__type MUST match one of the Document Types agreed during onboarding. e.g.__
```json
"type": {
  "coding": [
    {
      "system": "http://snomed.info/sct",
      "code": "749001000000101",
      "display": "Appointment (record artifact)"
    }
  ]
}
```

__category MUST indicate the broader class of the Document Type as agreed during onboarding.e.g.__
```json
"category": [
  {
    "coding": [
      {
        "system": "http://ihe.net/xds/connectathon/classCodes",
        "code": "History and Physical",
        "display": "History and Physical"
      }
    ]
  }
]
```

__content MUST have at least one entry.__
__Specific to this 'direct' use case - content.attachment.url MUST contain the direct URL of the appointment being registered__
```json
"content": [
  {
    "attachment": {
      "url": "http://example.org/api/FHIR/R4/Appointment/{ID}"
    }
  }
]
```

__content[].format[] SHOULD be as follows, e.g.__
```json
"format": {
  "system": "https://fhir.nhs.uk/CodeSystem/message-events-bars",
  "code": "booking-request",
  "display": "Booking Request - Request"
}
```

__author SHOULD have an entry with an Organization reference using a valid ODS code.__
```json
"author": [
  {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/ods-organization-code",
      "value": "Y05868"
    }
  }
]
```

__context MUST include a related entry containing the ASID if the document is to be accessed via SSP, e.g. - NOT REQUIRED IN THIS CASE__
```json
"context": {
  "related": [
    {
      "identifier": {
        "system": "https://fhir.nhs.uk/Id/nhsSpineASID",
        "value": "230811201350"
      }
    }
  ]
}
```

__context SHOULD include a period set to the Appointment start and end times, e.g.__
```json
"context": {
  "period": {
    "start": "2025-01-15T09:50:00Z",
    "end": "2025-01-15T10:00:00Z"
  }
}
```

__In order to be BaRS compliant, the DocumentReference MUST include identifiers for the following systems:__
* https://fhir.nhs.uk/Id/BaRS-Identifier
  * This is the identifier of the BaRS resource this pointer pertains to, in this example an appointment with an id of 8c63d621-4d86-4f57-8699-e8e22d49935d
* https://fhir.nhs.uk/Id/dos-service-id
  * The value of the identifier with this system is determined at the time of BaRS onboarding, and allows the BaRS proxy to lookup the service in the Endpoint Catalogue.
```json
"identifier": [
  {
    "system": "https://fhir.nhs.uk/Id/BaRS-Identifier",
    "value": "8c63d621-4d86-4f57-8699-e8e22d49935d"
  },
  {
    "value": "2000072491",
    "system": "https://fhir.nhs.uk/Id/dos-service-id"
  }
]
```

__In order to be BaRS compliant, the DocumentReference SHOULD include identifiers for this system:__
* https://fhir.nhs.uk/id/product-id
  * __For Future Consideration__ The product id is a forthcoming feature of the NRL API that we are currently building out. It allows for a different level of granularity from ODS Code. It is some months away from announcing formally, more information can be found in [the attached file](pdfs/Products%20and%20Product%20IDs.pdf) - this is an export of an internal NHS England design document, so will include broken links and is included only to give some context to the use of Product ID.
```json
"identifier": [
  {
    "system": "https://fhir.nhs.uk/id/product-id",
    "value": "P.GH7-4TY"
  }
]
```


<a name="example"></a>
## Example Payload
```json
{
  "resourceType": "DocumentReference",
  "id": "Yorkshire Ambulance Service|423456781055",
  "masterIdentifier": [
    {
      "system": "urn:ietf:rfc:3986",
      "value": "urn:uuid:27e2b1c8-ecd8-48f8-9958-8e614cc7ad73"
    }
  ],
  "identifier": [
    {
      "system": "https://fhir.nhs.uk/Id/BaRS-Identifier",
      "value": "8c63d621-4d86-4f57-8699-e8e22d49935d"
    },
    {
      "value": "2000072491",
      "system": "https://fhir.nhs.uk/Id/dos-service-id"
    },
    {
      "system": "https://fhir.nhs.uk/id/product-id",
      "value": "0B475C"
    }
  ],
  "status": "current",
  "type": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "749001000000101",
        "display": "Appointment (record artifact)"
      }
    ]
  },
  "category": [
    {
      "coding": [
        {
          "system": "http://ihe.net/xds/connectathon/classCodes",
          "code": ""History and Physical",
          "display": "History and Physical"
        }
      ]
    }
  ],
  "subject": {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/nhs-number",
      "value": "9876543210"
    }
  },
  "custodian": {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/ods-organization-code",
      "value": "X26",
      "display": "NHS ENGLAND - X26"
    }
  },
  "content": [
    {
      "attachment": {
        "contentType": "application/fhir+json",
        "language": "en-UK",
        "url": "https://api.example.com/booking-and-referral/FHIR/R4/Appointment/8c63d621-4d86-4f57-8699-e8e22d49935d"
      },
      "format": {
        "system": "https://fhir.nhs.uk/CodeSystem/message-events-bars",
        "code": "booking-request",
        "display": "Booking Request - Request"
      }
    }
  ],
  "context": {
    "period": {
      "start": "2025-01-15T09:50:00Z",
      "end": "2025-01-15T10:00:00Z"
    },
    "practiceSetting": {
      "coding": [
        {
          "system": "http://www.ihe.net/xds/connectathon/practiceSettingCodes",
          "code": "General Medicine",
          "display": "General Medicine"
        }
      ]
    }
  }
}
```

<a name="headers"></a>
## Request Headers

| Name | Description | Type |
| --- | --- | --- |
| `X-Request-Id` | An ID for the request, this is mirrored back in the response. | UUID |
| `X-Correlation-Id` | An ID that can be used internally by the client to correlate to internal business processes. | UUID |
| `NHSD-End-User-Organisation` | Requesting Organization described in an object based on a FHIR 'Organization' resource (Base64 encoded JSON). | Base64 encoded JSON object |

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
Example (Base64 encoded) `NHSD-End-User-Organisation`:
`ewrCoCAicmVzb3VyY2VUeXBlIjogIk9yZ2FuaXphdGlvbiIsCsKgICJpZGVudGlmaWVyIjogW
wrCoCDCoCB7CsKgIMKgIMKgICJodHRwczovL2ZoaXIubmhzLnVrL0lkL29kcy1vcmdhbml6YXRpb24tY29kZSIsCsKgIMK
gIMKgICJ2YWx1ZSI6ICJYMjYiCgoKwqAgwqAgfSwKwqAgIm5hbWUiOiAiTkhTIEVOR0xBTkQgLSBYMjYiCsKgIF0KfQ==` 

![Uploading pointers using BaRS Proxy](images/Figure3.svg)
Figure 3. Uploading pointers to NRL using BaRS Proxy

<a name="lookup"></a>
## Appointment Look-up

[See Attached](pdfs/2b.pdf)

[^1]: As this is a BaRS Proxy API, the consumer system (i.e. the requestor) will need to be formally onboarded with the BaRS team

[^2]: Some provided examples (request body) are incorrect (more specifically for the POST and PUT calls). The examples also need to be FHIR R4 compliant. All API pages are currently under review by the BaRS team who will shortly make corrections and updates.