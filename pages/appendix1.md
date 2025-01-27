[Home](../readme.md)

[Next - Appendix 2: Signposting (technical detail)](appendix2.md)

# Appendix 1: Authentication (technical detail)

<a name="background"></a>
## Background / business process
Users of the NHS App need to be able to use MyChart with NHS login authentication, without another authentication challenge between the two systems.

NHS login implements [OpenID](https://openid.net/) Connect 1.0 OpenID Provider role to assert the identity of the End-User to a Partner Service, as well as enabling the Service to obtain basic profile information about the End-User in an interoperable manner.

<a name="triggers"></a>
## Trigger events
NHS App Authenticated User navigates to MyChart.

## Specifications
[Identity Federation External Interface Specification](https://nhsconnect.github.io/nhslogin/interface-spec-doc/)