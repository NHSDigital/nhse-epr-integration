[Home](../readme.md)

[Back to Appendix 2](appendix2.md)

# Appointment Fields

## id
__Required__

__Description__

A unique identifier of the resource within the source system

## resourceType
__Required__

__Description__

FHIR Appointment resource type, part of the FHIR spec.

## identifiers
__Optional__

__Description__

Identifiers used within the source system.
NB: The identifier with system of `urn:oid:1.2.840.114350.1.13.0.1.7.3.698084.8` holds the Epic __CSN__.
e.g:
```json
"identifier": [
  {
    "system": "urn:oid:1.2.840.114350.1.13.5325.1.7.3.698084.8",
    "value": "10000209577"
  }
],
```

## status
__Required__

__Description__

Restricted by FHIR R4 to a value from [this set](https://hl7.org/fhir/R4/valueset-appointmentstatus.html):
- proposed
- pending
- booked
- arrived
- fulfilled
- cancelled
- noshow
- entered-in-error
- checked-in
- waitlist

Further constrained by Wayfinder to this set:
- booked
- arrived
- fulfilled
- cancelled
- noshow

## serviceCategory
__Optional__

__Description__

A broad categorization of the service that is to be performed during this appointment

__Not used__

## serviceType
__Optional__

__Description__

The specific service that is to be performed during this appointment

__Not used__

## Specialty

__Optional BUT Required by Wayfinder__

__Description__

The specialty of a practitioner that would be required to perform the service requested in this appointment

* Wayfinder uses https://fhir.nhs.uk/STU3/CodeSystem/Specialty-1 which is based on NHS Data Dictionary.
* Epic example payload doesn't include this field

## appointmentType
__Optional__

__Description__

The style of appointment or patient that has been booked in the slot (not service type)

__Expansion of this [is here](https://build.fhir.org/ig/HL7/UTG/CodeSystem-v3-ActCode.html)__

* Not used in Wayfinder
* Present in the Epic example

e.g:
```json
"appointmentType": {
  "coding": [
    {
      "system": "urn:oid:2.16.840.1.113883.5.4",
      "code": "AMB",
      "display": "Ambulatory"
    }
  ]
}
```

## reasonCode
__Optional__

__Description__

Coded reason this appointment is scheduled

* Not used in Wayfinder
* In the Epic example below, one or more SNOMED coded 'reasons'

e.g:
```json
{
  "coding": [
    {
      "system": "http://snomed.info/sct",
      "code": "316931000119104"
    }
  ],
  "text": "Knee pain, right"
}
```

## reasonReference
__Optional__

__Description__

Reason the appointment is to take place (resource)

* Not used in Wayfinder
* In the Epic example below a relative reference to one or more Condition resources

e.g:
```json
"reasonReference": [
{
  "reference": "Condition/eMIcE44Y.K0BWJcJ.QI13kIddolPVrQtCRadjUBUtusXiVGaC07vvUp85vvtcpU1l3"
}
]
```

## priority
__Optional__

__Description__

An [Unsigned Int](https://hl7.org/fhir/R4/datatypes.html#unsignedInt) Used to make informed decisions if needing to re-prioritize

* Not used in Wayfinder
* Not present in the Epic example below

## description
__Optional BUT Required by Wayfinder__

__Description__

Shown on a subject line in a meeting request, or appointment list

* Required by Wayfinder
* Present in the Epic example below

e.g:
```json
"description": "Visit for routine care"
```

## supportingInformation
__Optional__

__Description__

Additional information to support the appointment

* Not used by Wayfinder
* Not present in the Epic example below

## start
__Optional BUT Required for Wayfinder__

__Description__

When appointment is to take place. An [instant](https://hl7.org/fhir/R4/datatypes.html#instant)
* Requires accuracy to include seconds
* Requires a timezone
* Required by Wayfinder
* Present in the Epic example

```json
"start": "2022-04-20T20:00:00Z"
```

## end
__Optional__

__Description__

When appointment is to conclude. An [instant](https://hl7.org/fhir/R4/datatypes.html#instant)
* Requires accuracy to include seconds
* Requires a timezone
* Not used by Wayfinder
* Present in the Epic example

## minutesDuration
__Optional__

__Description__

Can be less than start/end (e.g. estimate). A [positiveInt](https://hl7.org/fhir/R4/datatypes.html#positiveInt)
* Not used by Wayfinder
* Present in the Epic example

## slot
__Optional__

__Description__

Reference to the (zero to many) slots that this appointment is filling.

* Not used by Wayfinder
* Not present in the Epic example

## created
__Optional__

__Description__

The date that this appointment was initially created

* Not used in Wayfinder
* Present in the Epic example below

e.g:
```json
"created": "2022-03-24"
```

## comment
__Optional__

__Description__

Additional comments

* Not used by Wayfinder
* Present in the Epic example

e.g:
```json
"comment": "Patient has new, persistent knee pain during daily activities."
```

## patientInstruction
__Optional__

__Description__

Detailed information and instructions for the patient

* Not used by Wayfinder
* Present in the Epic example

e.g:
```json
"patientInstruction": "Please make sure patient arrives 5 minutes early to fill out any necessary paperwork before the appointment."
```

## basedOn
__Optional__

__Description__

The service request this appointment is allocated to assess

* Not used in Wayfinder
* Not present in the Epic example below

## participant
__Required - See rule below__

__Description__

Participants involved in appointment
+ Rule: Either the type or actor on the participant SHALL be specified
----
* Patient is used by Wayfinder
* Wayfinder expects one of them to be the Patient
* For the Patient, Wayfinder expects Participant[].identifier.system to be `https://fhir.nhs.uk/Id/nhs-number`
* For the Patient, Wayfinder expects Participant[].identifier.value to be the NHS Number
* As seen below, Epic Patient Participant doesn't include the NHS Number
* Epic example Participants don't include the `type` which is expected to come from [this ValueSet](https://hl7.org/fhir/R4/valueset-encounter-participant-type.html).


## Example Epic Appointment
Taken from [here](https://fhir.epic.com/Specifications?api=10468)

NB: The Location Participant at that ink is broken, with a missing `"` before `West Clinic"`
```json
{
    "resourceType": "Appointment",
    "id": "elmPBHPxEEEvVLKSSR6xGfQaaeOxoGVxtCt9FlmcwgQ03",
    "identifier": [
        {
            "system": "urn:oid:1.2.840.114350.1.13.5325.1.7.3.698084.8",
            "value": "10000209577"
        }
    ],
    "status": "booked",
    "serviceCategory": [
        {
            "coding": [
                {
                    "system": "http://open.epic.com/FHIR/StructureDefinition/appointment-service-category",
                    "code": "appointment",
                    "display": "Appointment"
                }
            ],
            "text": "appointment"
        }
    ],
    "serviceType": [
        {
            "coding": [
                {
                    "system": "urn:oid:1.2.840.114350.1.13.5325.1.7.2.808267",
                    "code": "6",
                    "display": "Office Visit"
                }
            ]
        }
    ],
    "appointmentType": {
        "coding": [
            {
                "system": "urn:oid:2.16.840.1.113883.5.4",
                "code": "AMB",
                "display": "Ambulatory"
            }
        ]
    },
    "reasonCode": [
        {
            "coding": [
                {
                    "system": "http://snomed.info/sct",
                    "code": "316931000119104"
                }
            ],
            "text": "Knee pain, right"
        }
    ],
    "reasonReference": [
        {
            "reference": "Condition/eMIcE44Y.K0BWJcJ.QI13kIddolPVrQtCRadjUBUtusXiVGaC07vvUp85vvtcpU1l3"
        }
    ],
    "description": "Visit for routine care",
    "start": "2022-04-20T20:00:00Z",
    "end": "2022-04-20T20:15:00Z",
    "minutesDuration": 15,
    "created": "2022-03-24",
    "comment": "Patient has new, persistent knee pain during daily activities.",
    "patientInstruction": "Please make sure patient arrives 5 minutes early to fill out any necessary paperwork before the appointment.",
    "participant": [
        {
            "actor": {
                "reference": "Patient/eNO3wqOfAltfnWMfWBQ1WmQ3",
                "display": "Doe, Jane"
            },
            "required": "required",
            "status": "accepted",
            "period": {
                "start": "2022-04-20T20:00:00Z",
                "end": "2022-04-20T20:15:00Z"
            }
        },
        {
            "actor": {
                "reference": "Practitioner/eb3d.4apzwRM33Q91Nm7wJA3",
                "display": "Amy Smith, MD"
            },
            "required": "required",
            "status": "accepted",
            "period": {
                "start": "2022-04-20T20:00:00Z",
                "end": "2022-04-20T20:15:00Z"
            }
        },
        {
            "actor": {
                "reference": "Location/ezECB8TyTDQJuuly5Byn.lQ3",
                "display": "West Clinic"
            },
            "required": "required",
            "status": "accepted",
            "period": {
                "start": "2022-04-20T20:00:00Z",
                "end": "2022-04-20T20:15:00Z"
            }
        }
    ]
}
```