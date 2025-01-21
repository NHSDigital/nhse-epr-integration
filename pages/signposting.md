[Home](../readme.md)

# Signposting

The strategic solution MUST have Epic/EPR integrate with the NRL ([National Record Locator](https://digital.nhs.uk/services/national-record-locator)) service. In the future, most NHS systems of record will be required to upload pointers (or, record locators) to patient data, thus enabling discovery and subsequent retrieval (and in some cases, the management) of those patient data.

Integration with NRL can be done directly (using NRL APIs), or indirectly via BaRS (using BaRS APIs for NRL). The latter approach is recommended because compliance with BaRS, integration with the BaRS Proxy, as well as integration with NRL can all be achieved by engaging with only one team (the BaRS team). 

Whilst the existing approach used in Wayfinder has some benefits, it is not part of the NHS England’s strategic direction, for two key reasons:
1.	The pointers currently stored in Wayfinder are relationship-based, meaning that the pointer only indicates that “a system has some data for a particular patient”, but it can neither provide information about how much data, nor can it allow a level of filtering (based on some additional metadata) during the look-up / discovery step. This generally leads to query-based data retrievals that (can) have unpredictable load profiles and is an approach discouraged by NHS England.
2.	The pointers currently stored in Wayfinder are not available to any other system/service/component as they are kept in a ‘silo’ and not aligned to NHS England recommended standards.

The low level technical detail is provided below in [Appendix 1](appendix1.md): Signposting (technical detail).
