# Tactical approach

The tactical approach is to perpetuate the Wayfinder __Record Service__ approach.

This contains a record to represent each time a patient is registered in a Trust.

In the below, some assumptions are:

* 

## Lookup and Retrieval via BaRS
```mermaid
sequenceDiagram
    box NHS England Internal
        participant NHS App
        participant Aggregator
        participant RecordService
    end
    box Trust
        participant Epic
    end
    NHS App->>Aggregator: Get Appointments
    activate NHS App
    activate Aggregator
    Aggregator->RecordService: Query
    activate RecordService
    RecordService-->>Aggregator: References
    deactivate RecordService
    loop Per Reference
        Aggregator->>Epic: /Patient$match
        activate Epic
        Epic->>Epic: Find Patient
        Epic-->>Aggregator: 0..1 Patients
        Aggregator->>Aggregator: Get Patient.id
        Aggregator->>Epic: Appointment search
        Epic->>Epic: Find Appointments
        Epic-->>Aggregator: 0..* Appointments
        deactivate Epic
    end
    Aggregator-->>NHS App: Appointments
    deactivate Aggregator
    deactivate NHS App
```
