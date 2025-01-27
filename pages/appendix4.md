[Home](../readme.md)

# Appendix 4: Transfer (technical detail)

<a name="background"></a>
## Background / business process
When an NHS App user has found an appointment provided at an Epic trust in their NHS App, and wants to carry out more detailed actions on that appointment, then can 'smoothly' link from the NHS App to the details of that appointment in MyChart.

<a name="triggers"></a>
## Trigger events
User clicks on 'Manage this Appointment' (working to be confrmed) button or link in the NHS App.

<a name="requester"></a>
## Requester
NHS App

<a name="responder"></a>
## Responder
MyChart appointment page

<a name="endpoint"></a>
## Endpoint
It is assumed that there are two options for determining the endpoint for this:

* The appointment resource fetched from Epic contains a URL for the transfer. This is the 'normal' mechanism implemented by Wayfinder. An Extension [(ExtensionWayfinderPortalLink)](https://simplifier.net/wayfinder-patient-care-aggregator/extensionwayfinderportallink)  is used to carry the value.
* The URL can be constructed by concatenating:
  * `"https://"` +
  * `[A Base FHIR server URL for the Epic instance, looked up somehow from the ODS code of the Trust]` +
  * `"/Appointment/"` +
  * `[The ID, or an identifier value from the Appointment resource]`

<a name="authentication"></a>
## Authentication
Is covered in Appendix 1: Authentication (technical detail).
