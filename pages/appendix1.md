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
The actual endpoints [^2] are defined in [the BaRS documentation](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0) under the Document Reference section. __NB: That documentation currently contains some known errors and is under active review.__

On creating a new Appointment, send a POST request as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#post-/DocumentReference)

On change of an existing Appointment, send a PUT request as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#put-/DocumentReference/-id-) (the ID used is the ID returned when the appointment was created).

On cancellation of an existing Appointment either (Preferred) an update as above can be made (to change status to cancelled) or the DocumentReference can be deleted by making a DELETE call as described [here](https://digital.nhs.uk/developer/api-catalogue/booking-and-referral-fhir/v1_1_0#delete-/DocumentReference/-id-) (the ID used is the ID returned when the appointment was created)

<a name="authentication"></a>
## Authentication
Uses Application Restricted as defined [here](https://digital.nhs.uk/developer/guides-and-documentation/security-and-authorisation/application-restricted-restful-apis-signed-jwt-authentication)

<a name="payload"></a>
## Payload
The payload for POST and PUT is a FHIR R4 DocumentReference with some specific restrictions defined [here](https://digital.nhs.uk/developer/api-catalogue/national-record-locator-fhir/v3/producer#post-/DocumentReference)

The restrictions will be determined by the BaRS programme, but in general are:
- [identifier](appendix1.md#identifier)
- optional [identifier](appendix1.md#optionalidentifier)
- [status](appendix1.md#status)
- [type](appendix1.md#type)
- [category](appendix1.md#category)
- [subject](appendix1.md#subject)
- [date](appendix1.md#date)
- [author](appendix1.md#author)
- [custodian](appendix1.md#custodian)
- [content](appendix1.md#content)
- [content extension](appendix1.md#extension)
- [content.format](appendix1.md#format)
- [context.period](appendix1.md#period)
- [context.practicesetting](appendix1.md#practicesetting)
- [Full example](appendix1.md#example)


<a name="identifier"></a>
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


<a name="optionalidentifier"></a>
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


<a name="status"></a>
__status MUST reflect the status of the DocumentReference. This is expected to always be current__
```json
"status": "current"
```

<a name="type"></a>
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

<a name="category"></a>
__category MUST indicate the broader class of the Document Type as agreed during onboarding.e.g.__
```json
"category": [
  {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "419891008",
        "display": "Record artifact (record artifact)"
      }
    ]
  }
]
```


<a name="subject"></a>
__subject MUST be a Patient reference using a valid NHS number. e.g.__
```json
"subject": {
  "identifier": {
    "system": "https://fhir.nhs.uk/Id/nhs-number",
    "value": "3495456481"
  }
}
```


<a name="date"></a>
__date MUST be set to the instant the DocumentReference was created__
```json
"date": "2025-02-05T08:25:13.136693+00:00"
```


<a name="author"></a>
__author MUST have an entry with an Organization reference using a valid ODS code.__

This refers to __the Trust__ that the Appointment is at.
```json
"author": [
  {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/ods-organization-code",
      "value": "RGT"
    }
  }
]
```


<a name="custodian"></a>
__custodian MUST be an Organization reference using a valid ODS code, agreed during onboarding. e.g.__

This refers to __the Trust__ that the Appointment is at.
```json
"custodian": {
  "identifier": {
    "system": "https://fhir.nhs.uk/Id/ods-organization-code",
    "value": "RGT"
  }
}
```




<a name="content"></a>
__content MUST have exactly one entry - content[0].attachment.url MUST contain the direct URL of the appointment being registered and content[0].attachement.contentType MUST be populated__
```json
"content": [
  {
    "attachment": {
      "url": "https://server.fire.ly/r4/Appointment/GL0-DZOqD39",
      "contentType": "application/fhir+json"
    }
  }
]
```


<a name="extension"></a>
__content[0] MUST include the Extension: `https://fhir.nhs.uk/England/StructureDefinition/Extension-England-ContentStability` with the code/display values of `static/Static` or `dynamic/Dynamic`__
```json
"extension": [
  {
    "url": "https://fhir.nhs.uk/England/StructureDefinition/Extension-England-ContentStability",
    "valueCodeableConcept": {
      "coding": [
        {
          "system": "https://fhir.nhs.uk/England/CodeSystem/England-NRLContentStability",
          "code": "static",
          "display": "Static"
        }
      ]
    }
  }
]
```


<a name="format"></a>
__content[0].format[] MUST be as follows__
```json
"format": {
  "system": "https://fhir.nhs.uk/England/CodeSystem/England-NRLFormatCode",
  "code": "urn:nhs-ic:structured",
  "display": "Structured Document"
}
```


<a name="period"></a>
__context MUST include a period set to the Appointment start and end times, e.g.__
```json
"context": {
  "period": {
    "start": "2025-01-15T09:50:00Z",
    "end": "2025-01-15T10:00:00Z"
  }
}
```


<a name="practicesetting"></a>
__context.practiceSetting MUST include a coded Clinical Specialty for the Appointment, This code must be a child of `394658006|Clinical speciality` e.g.__
```json
"context": {
  "practiceSetting": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "394802001",
        "display": "General medicine (qualifier value)"
    }
    ]
  }
}
```
<a name="example"></a>
## Example Payload
```json
{
  "resourceType": "DocumentReference",
  "identifier": [
    {
      "system": "https://fhir.nhs.uk/Id/BaRS-Identifier",
      "value": "GL0-DZOqD39"
    },
    {
      "system": "https://fhir.nhs.uk/Id/dos-service-id",
      "value": "2000072491"
    },
    {
      "system": "https://fhir.nhs.uk/id/product-id",
      "value": "P.GH7-4TY"
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
          "system": "http://snomed.info/sct",
          "code": "419891008",
          "display": "Record artifact (record artifact)"
        }
      ]
    }
  ],
  "subject": {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/nhs-number",
      "value": "9693893123"
    }
  },
  "date": "2025-02-05T08:25:13.136693+00:00",
  "author": [
    {
      "identifier": {
        "system": "https://fhir.nhs.uk/Id/ods-organization-code",
        "value": "RGT"
      }
    }
  ],
  "custodian": {
    "identifier": {
      "system": "https://fhir.nhs.uk/Id/ods-organization-code",
      "value": "RGT"
    }
  },
  "content": [
    {
      "extension": [
        {
          "url": "https://fhir.nhs.uk/England/StructureDefinition/Extension-England-ContentStability",
          "valueCodeableConcept": {
            "coding": [
              {
                "system": "https://fhir.nhs.uk/England/CodeSystem/England-NRLContentStability",
                "code": "static",
                "display": "Static"
              }
            ]
          }
        }
      ],
      "attachment": {
        "contentType": "application/fhir+json",
        "url": "https://server.fire.ly/r4/Appointment/GL0-DZOqD39"
      },
      "format": {
        "system": "https://fhir.nhs.uk/England/CodeSystem/England-NRLFormatCode",
        "code": "urn:nhs-ic:structured",
        "display": "Structured Document"
      }
    }
  ],
  "context": {
    "period": {
      "start": "2025-02-01T06:45:00+00:00",
      "end": "2025-02-01T07:00:00+00:00"
    },
    "practiceSetting": {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "394802001",
          "display": "General medicine (qualifier value)"
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
| `NHSD-End-User-Organisation-ODS` | Requesting Organization ODS Code. | String |
| `accept` | Must be `application/fhir+json;version=1.1.0` | String |
| `content-type` | Must descibe the payload e.g. `application/fhir+json;version=1.1.0` | String |


![Uploading pointers using BaRS Proxy](images/Figure3.svg)
Figure 3. Uploading pointers to NRL using BaRS Proxy

<a name="lookup"></a>
## Appointment Look-up

[See Attached](pdfs/2b.pdf)

[^1]: As this is a BaRS Proxy API, the consumer system (i.e. the requestor) will need to be formally onboarded with the BaRS team

[^2]: Some provided examples (request body) are incorrect (more specifically for the POST and PUT calls). The examples also need to be FHIR R4 compliant. All API pages are currently under review by the BaRS team who will shortly make corrections and updates.
