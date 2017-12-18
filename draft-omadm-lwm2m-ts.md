---
stand_alone: true
ipr: trust200902
docname: draft-omadm-lwm2m-ts
cat: std
pi:
  toc: 'yes'
  symrefs: 'yes'
title: Lightweight Machine to Machine Technical Specification - Transport Bindings
abbrev: LWM2M TS
area: Applications
wg: OMA
author:
- ins: J. Jimenez
  role: editor
  name: Jaime Jimenez
  org: Ericsson
  street: My Great Street
  city: Madrid
  region: Spain
- ins: A. One
  name: Author One
  org: Company
  email: author@company.org
- ins: A. Two
  name: Author Two
  org: Company
  email: author@company.org
- ins: A. Three
  name: Author Three
  org: Company
  email: author@company.org

normative:
  RFC2119:
  RFC4648:
  RFC5277:
  RFC5246:
  RFC6243:
  RFC7049:
  RFC7252:
  RFC7950:
  RFC7959:
  RFC7641:
  RFC8132:
  RFC8040:

informative:
  RFC6655:
  RFC6066:
  RFC4086:
  RFC7251:
  RFC6125:
  RFC5280:
  RFC4279:
  RFC4293:
  RFC6347:
  RFC6690:
  RFC7159:
  RFC7223:
  RFC7925:
  RFC7317:
  I-D.ietf-core-interfaces:
  XML:
    title: Extensible Markup Language (XML)
    author:
    - org: W3C
    date: false
    seriesinfo:
      Web: http://www.w3.org/xml

--- abstract

This document specifies version 1.0 of the Lightweight Machine-to-Machine (LwM2M) protocol. This Lightweight M2M 1.0 enabler introduces the following features:

*   Simple resource model with the core set of objects and resources defined in this specification. The full list of registered objects can be found at (TODO OMNA).

*   Operations for creation, update, deletion, and retrieval of resources.

*   Asynchronous notifications of resource changes.

*   Support for several serialization formats, namely TLV, JSON, Plain Text and binary data formats and the core set of LightweightM2M Objects.

*   UDP and SMS transport support.

*   Communication security based on the DTLS protocol supporting different types of credentials.

*   Queue Mode offers functionality for a LwM2M Client to inform the LwM2M Server that it may be disconnected for an extended period and when it becomes reachable again.

*   Support for use of multiple LwM2M Servers.

*   Provisioning of security credentials and access control lists by a dedicated LwM2M bootstrap-server.

--- note_Note

Discussion and suggestions for improvement are requested,
and should be sent to omaemail@oma.com.

--- middle

# Copyright Notice {#copyright}

Use of this document is subject to all of the terms and conditions of the Use Agreement located at http://www.openmobilealliance.org/UseAgreement.html.

Unless this document is clearly designated as an approved specification, this document is a work in process, is not an approved Open Mobile Alliance™ specification, and is subject to revision or removal without notice.

You may use this document or any part of the document for internal or educational purposes only, provided you do not modify, edit or take out of context the information in this document in any manner. Information contained in this document may be used, at your sole risk, for any purposes. You may not use this document in any other manner without the prior written permission of the Open Mobile Alliance. The Open Mobile Alliance authorizes you to copy this document, provided that you retain all copyright and other proprietary notices contained in the original materials on any copies of the materials and that you comply strictly with these terms. This copyright permission does not constitute an endorsement of the products or services. The Open Mobile Alliance assumes no responsibility for errors or omissions in this document.

Each Open Mobile Alliance member has agreed to use reasonable endeavors to inform the Open Mobile Alliance in a timely manner of Essential IPR as it becomes aware that the Essential IPR is related to the prepared or published specification. However, the members do not have an obligation to conduct IPR searches. The declared Essential IPR is publicly available to members and non-members of the Open Mobile Alliance and may be found on the “OMA IPR Declarations” list at http://www.openmobilealliance.org/ipr.html. The Open Mobile Alliance has not conducted an independent IPR review of this document and the information contained herein, and makes no representations or warranties regarding third party IPR, including without limitation patents, copyrights or trade secret rights. This document may contain inventions for which you must obtain licenses from third parties before making, using or selling the inventions. Defined terms above are set forth in the schedule to the Open Mobile Alliance Application Form.

NO REPRESENTATIONS OR WARRANTIES (WHETHER EXPRESS OR IMPLIED) ARE MADE BY THE OPEN MOBILE ALLIANCE OR ANY OPEN MOBILE ALLIANCE MEMBER OR ITS AFFILIATES REGARDING ANY OF THE IPR’S REPRESENTED ON THE “OMA IPR DECLARATIONS” LIST, INCLUDING, BUT NOT LIMITED TO THE ACCURACY, COMPLETENESS, VALIDITY OR RELEVANCE OF THE INFORMATION OR WHETHER OR NOT SUCH RIGHTS ARE ESSENTIAL OR NON-ESSENTIAL.

THE OPEN MOBILE ALLIANCE IS NOT LIABLE FOR AND HEREBY DISCLAIMS ANY DIRECT, INDIRECT, PUNITIVE, SPECIAL, INCIDENTAL, CONSEQUENTIAL, OR EXEMPLARY DAMAGES ARISING OUT OF OR IN CONNECTION WITH THE USE OF DOCUMENTS AND THE INFORMATION CONTAINED IN THE DOCUMENTS.

© 2017 Open Mobile Alliance All Rights Reserved. Used with the permission of the Open Mobile Alliance under the terms set forth above.

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{RFC7252}} is designed for
Machine to Machine (M2M) applications such as smart energy, smart city and building control.
Constrained devices need to be managed in an automatic fashion to handle
the large quantities of devices that are expected in
future installations. Messages between devices need to be as small and
infrequent as possible. The implementation
complexity and runtime resources need to be as small as possible.

This specification defines the transport bindings for the LwM2M messaging protocol used between the LwM2M Client, the LwM2M Bootstrap Server and with the LwM2M Server. Figure 1 shows the relationships between the transport bindings and the messaging protocol. In particular, this specification defines the following transport bindings.

*   CoAP over UDP
*   CoAP over DTLS over UDP
*   CoAP over SMS
*   CoAP over DTLS over SMS


## Terminology {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in {{RFC2119}}.

# LwM2M Architecture {#lwm2m-architecture}


A general architecture can be seen below {{archit}}.

~~~~
    +----------------------------------+  +---------+
    |              LWM2M               |  | Objects |
    +----------------------------------+  +---------+
    +----------------------------------+
    |              CoAP                |
    +----------------------------------+
    +----------------+ +---------------+
    |      DTLS      | |    SMS        |
    +----------------+ |               |
    +-------+ +------+ |  on Smartcard |
    |  UDP  | |SMS on| |               |
    |       | |device| |               |
    +-------+ +------+ +---------------+
~~~~
{: #archit title='The Protocol Stack of the LwM2M Enabler' artwork-align="left"}


#Transport Layer Binding and Encodings

This section defines the CoAP transport binding used by LwM2M interfaces.

##Required Features

For realization of the LwM2M interfaces, only the basic binary CoAP message header, and a small subset of options are required. This section explicitly defines the features of the CoAP standard that are required for LwM2M.

* The 4-byte binary CoAP message header is defined in Section 3 of {{RFC7252}}. This same base message is used for Request and Response interactions.

* Confirmable, Acknowledgement and Reset messages MUST be supported. The Reset message is used as a message layer error message in response to a malformed Confirmable message. Non-Confirmable messages MAY be used by a Client for sending Information Reporting notifications as per (TODO Observe).

* GET, PUT, POST and DELETE methods MUST be supported. LwM2M Operations map to these methods.

* A subset of Response Codes MUST be supported for LwM2M response message mapping.

* The Uri-Path Option MUST be supported to indicate the identifier of the interface, Object Instance and Resource being requested.

* The Location-Path Option MUST be supported to indicate the handle of a registration for future update and delete operations.

* The Uri-Query Option MUST be supported.

* The Content-Format Option MUST be used to indicate the data format of the payload.

* The Accept Option MAY be included in a LwM2M Server data request, to specify the payload Content-Format this Server prefers to receive. The Client returns the preferred Content-Format if available. If this Accept option is not given or if the LwM2M Client doesn’t support that option, the LwM2M Client will use its own preferred data format reported in the Content-Format of the response message. If the preferred Content-Format cannot be returned, then a 4.06 “Not Acceptable” value MUST be sent as a response.

* The Token Option MAY be used to enable multiple requests in parallel with an endpoint, and MUST be supported for the Information Reporting interface.

* CoAP Blockwise transfer (TODO add REF) for CoAP MUST be supported by the LwM2M Client when the Firmware Update Object (ID:5) is implemented by the client and MUST be supported by the LwM2M Server. This functionality is motivated by limitations of CoAP, as defined in [RFC7252] since CoAP was not designed for transmission of large payloads. Because the CoAP header itself does not contain length information the UDP length header is used instead. The maximum UDP datagram size is limited to ~64 KiB and transmitting data beyond the (path) maximum transmission (MTU) size will additionally lead to inefficiency because of fragmentation at lower layers (IP layer, adaptation layer, and link layer). Blockwise Transfer for CoAP (TODO CoAP_Blockwise) was specifically designed to lift this limitation in order to transfer large payloads larger than ~64 KiB via CoAP, such as firmware images. Note: (TODO CoAP_Blockwise) is also beneficial for use with firmware images smaller than 64 KiB since the block-wise transmission allows the server to deliver firmware images in chunks suitable to the MTU and thereby avoiding fragmentation at lower layers. A LwM2M client may choose to support block-wise transfer for objects other than the Firmware Update object. This may, for example, be useful with objects that are larger in size, such as the security object which may contain certificates. The specifics of how this functionality is utilized by a LwM2M Server are out of scope for this release of LwM2M.

##URI Identifier & Operation Mapping

Although CoAP supports a URI in requests, it is not used in the same way as in HTTP. The URI in CoAP is broken down into binary parts, minimizing overhead and complexity. In LwM2M only path segment and query string URI components are needed. The URI path is used to simply identify the interface, Object Instance or Resource that the request is for, and is encoded in Uri-Path options. The LwM2M Registration interface also makes use of query string parameters to pass on meta-data with the request separately from the payload. Each query parameter is encoded in a Uri-Query Option. Likewise, the LwM2M operations for each interface are mapped to CoAP Methods. All the LwM2M operations using CoAP layer MUST be Confirmable CoAP messages, except as follows:

* “Notify” which may be a Non-Confirmable CoAP message.
* UQT mode triggering an execution on a resource can be a Non-Confirmable CoAP message.

### Firewall/NAT

For a firewall to support LwM2M, it should be configured to allow outgoing UDP packets to destination port 5683 (other ports can be configured), and allow incoming UDP packets back to the source address/port of the outgoing UDP packet for a period of at least 240 seconds. These UDP packets may contain DTLS or CoAP payloads. When a firewall is configured as such any LwM2M Clients behind it should use Queue Mode.

For a firewall to support LwM2M it can be configured to allow both outgoing and incoming UDP packets to destination port 5683 (other ports can be configured). These UDP packets may contain DTLS or CoAP payloads. When a firewall is configured as such any LwM2M Clients behind it are not required to use Queued Mode, but may use it for other reasons (e.g., a battery powered sleeping device).

Any LwM2M Clients behind a NAT can use Queued Mode. There are other mechanisms to transverse a NAT, however they are out of scope for the LwM2M Enabler.

### Alternate Path

By default, the LwM2M Objects are located under the root path. However, devices might be hosting other CoAP Resources on an endpoint, and there may be the need to place LwM2M Objects under an alternate path.

When registering, or updating its registration, a LwM2M Client MAY include an OMA LwM2M link in addition to the Object links in the registration payload. The link is identified by {{RFC6690}} Resource Type parameter “oma.lwm2m”.

This link MUST NOT contain numerical URI segment.

For instance, the Example Client from Appendix F may place Objects under the “/lwm2m” path. The registration payload would be as follows:

~~~~
</lwm2m>;rt="oma.lwm2m",
</lwm2m /1/0>,</lwm2m /1/1>,
</lwm2m /2/0>,</lwm2m /2/1>,</lwm2m /2/2>,</lwm2m /2/3>,</lwm2m /2/4>,
</lwm2m /3/0>,
</lwm2m /4/0>,
</lwm2m /5>
~~~~
{: #sampletext title='Sample text'artwork-align="left"}

When using the Device Management & Service Enablement Interface and the Information Reporting Interface, the LwM2M Server MUST prepend the OMA LwM2M link to the path in the CoAP messages. Example: "GET /lwm2m/3/0/0."

When using the Bootstrap Interface, the LwM2M Bootstrap-Server MUST use CoAP paths only in the form:
~~~~
/{Object ID}/{Object Instance ID}/{Resource ID}
~~~~
{: #sampletext2 title='Sample text'artwork-align="left"}

It is the responsibility of the LwM2M Client to map these paths to its alternate path.

The Resource Type value “oma.lwm2m” is part of IANA registry.

TODO Continue from Bootstrap Interface

##Bootstrap Interface

##Registration Interface

##Device Management and Service Enablement Interface

##Information Reporting Interface

###Queue Mode Operation

###Update Trigger Mechanism


##Response Codes

This section lists available response codes of each operation. The codes are divided into each interface. These are the only valid response codes defined in for the LwM2M Enabler.

The different Content-formats usage is  summarized in the table below:

Table 1 example: Formatted

| Operations         | CoAP Responses     | Reason Phrase                               |
| Bootstrap Request  | 2.04 Changed       | Bootstrap-Request is completed successfully |
|                    | 4.00 Bad Request   | Unknown Endpoint Client Name                |
| Write              | 2.04 Changed       | Write operation is completed successfully   |
|                    | 4.00 Bad Request   | Data is malformed                           |
|                    | 4.15 Unsupported Content Format   | The specified format is not supported|
| Discover           | 2.05 Content       | Discover operation is completed successfully|
|                    | 4.00 Bad Request   | Undetermined error occurred                 |
|                    | 4.04 Not Found     | URI of “Discover” operation is not found    |
| Delete             | 2.02 Deleted       | Delete Operation is completed successfully  |
|                    | 4.00 Bad Request   | Bad or Unknown URI provided                 |
| Bootstrap-Finish   | 2.04 Changed       | Bootstrap-Finished is completed successfully|
|                    | 4.00 Bad Request   | Bad URI provided                            |
|                    | 4.04 Not Acceptable| Inconsistent loaded configuration           |
{: align="left"}

Table 1 example: Raw

~~~~
+--------------------------------------------------------------------------------+
| Operations  | CoAP Responses     | Reason Phrase                               |
+--------------------------------------------------------------------------------+
| Bootstrap   | 2.04 Changed       | Bootstrap-Request is completed successfully |
| Request     | 4.00 Bad Request   | Unknown Endpoint Client Name                |
+--------------------------------------------------------------------------------+
| Write       | 2.04 Changed       | Write operation is completed successfully   |
|             | 4.00 Bad Request   | Data is malformed                           |
|             | 4.15 Unsupported Content Format   | Unsupported Content Format   |
+--------------------------------------------------------------------------------+
| Discover    | 2.05 Content       | Discover operation is completed successfully|
|             | 4.00 Bad Request   | Undetermined error occurred                 |
|             | 4.04 Not Found     | URI of “Discover” operation is not found    |
+--------------------------------------------------------------------------------+
| Delete      | 2.02 Deleted       | Delete Operation is completed successfully  |
|             | 4.00 Bad Request   | Bad or Unknown URI provided                 |
+--------------------------------------------------------------------------------+
| Bootstrap   | 2.04 Changed       | Bootstrap-Finished is completed successfully|
| Finish      | 4.00 Bad Request   | Bad URI provided                            |
|             | 4.04 Not Acceptable| Inconsistent loaded configuration           |
+--------------------------------------------------------------------------------+
~~~~
{: #sampletable title='Sample Table' artwork-align="left"}



Table 1 example Object:

~~~~
 OID: 3304
 +--ro IPSO-humidity* [instance_number]
 +--ro instance_number                  uint16
 +--ro 5700     Sensor_Value            decimal64
 +--ro 5701?    Units                   string
 +--ro 5601?    Min_Measured_Value?     decimal64
 +--ro 5602?    Max_Measured_Value?     decimal64
 +--ro 5603?    Min_Range_Value?        decimal64
 +--ro 5604?    Max_Range_Value?        decimal64
 +---x 5605?    Reset_Min_and_Max_measured_values
~~~~
{: #sampletable2 title='Sample Table 2'artwork-align="left"}



# Security Considerations {#security-considerations}

For secure network management, it is important to restrict access to configuration variables
only to authorized parties. LwM2M re-uses the security mechanisms already available to CoAP,
this includes DTLS {{RFC6347}} for protected access to resources, as well suitable
authentication and authorization mechanisms.

The LwM2M protocol supports various transport bindings and credentials for securely communicating with LwM2M Servers. This configuration information can be provisioned during manufacturing or through the use of the bootstrap mechanism.

LwM2M supports three different types of credentials, namely

* Certificates.

* Raw public keys.

* Pre-shared secrets.

Since these credential types offer different properties the LwM2M specification offers support for all of them. {{RFC7925}} provides the necessary details about the use of each of these credentials with DTLS.

The LwM2M protocol specifies that authorization of LwM2M Servers to access Object Instances and Resources within the LwM2M Client is provided through Access Control Object Instances within the LwM2M Client.

## DTLS Security

**Requirements**

For authentication of communicating LwM2M entities, the LwM2M protocol requires that all communication between LwM2M Clients and LwM2M Servers as well as LwM2M Clients and LwM2M Bootstrap-Servers are authenticated using mutual authentication. This means that a:

* LwM2M Client MUST authenticate a LwM2M Server prior to exchange of any information.

* LwM2M Server MUST authenticate a LwM2M Client prior to exchange of any information.

* LwM2M Client MUST authenticate a LwM2M Bootstrap-Server prior to exchange of any information.

* LwM2M Bootstrap-Server MUST authenticate a LwM2M Client prior to exchange of any information.

For confidentiality and data integrity of information between communicating LwM2M entities, the LwM2M protocol requires that all communication between LwM2M Clients and LwM2M Servers as well as LwM2M Clients and LwM2M Bootstrap-Servers are encrypted and integrity protected. This means that a:

* LwM2M Client MUST encrypt and integrity protect data communicated to a LwM2M Server.

* LwM2M Server MUST encrypt and integrity protect data communicated to a LwM2M Client.

* LwM2M Client MUST encrypt and integrity protect data communicated to a LwM2M Bootstrap-Server.

* LwM2M Bootstrap-Server MUST encrypt and integrity protect data communicated to a LwM2M Client.

Due the sensitive nature of bootstrap information, a particular care has to be taken to ensure protection of that data.

The use of DTLS fulfils these requirements.

###DTLS Overview

CoAP {{RFC7252}} is secured using the Datagram Transport Layer Security (DTLS) 1.2 protocol {{RFC6347}}, which is based on TLS v1.2 {{RFC5246}}. The DTLS binding for CoAP is defined in Section 9 of {{RFC7252}}. DTLS is a communication security solution for datagram based protocols (such as UDP). It provides a secure handshake with session key generation, mutual authentication, data integrity and confidentiality.

This section provides information related to the use of DTLS for use with CoAP over DTLS over UDP as well as for use with CoAP over DTLS over SMS. Section "SMS Channel Security" provides additional information regarding the use of DTLS in an SMS context. This document uses the profiles defined in {{RFC7925}}, which includes not only ciphersuites but also extensions.

The DTLS client and the DTLS server SHOULD keep security state, such as session keys, sequence numbers, and initialization vectors, and other security parameters, established with DTLS for as long a period as can be safely achieved without risking compromise to the security context. If such state persists across sleep cycles where the RAM is powered off, secure storage SHOULD be used for the security context.

The credentials used for authenticating the DTLS client and the DTLS server to secure the communication between the LwM2M Client and the LwM2M Server are obtained using one of the bootstrap modes defined in Section 5.2.2. Appendix E.1.1 of (TODO add Link to LWM2M-CORE) defines the format of the keying material stored in the LwM2M Security Object Instances.

LwM2M Bootstrap-Servers, LwM2M Servers and LwM2M Clients MUST use different key pairs. LwM2M Clients MUST use keys, which are unique to each LwM2M Client. When a LwM2M Client is configured to utilize multiple LwM2M Servers then the LwM2M Bootstrap-Server may configure different credentials with these LwM2Ms Servers. Such configuration provides better unlinkability properties since each individual LwM2M Server cannot correlate request based on the credentials used by the LwM2M Client. Deployment and application specific considerations dictate what approach to use.

###Ciphersuites

DTLS supports the concept of ciphersuites and they are securely negotiated during the DTLS handshake. This specification recommends a limited number of ciphersuites based on {{RFC7925}}. The recommended ciphersuites have been chosen because of suitability for IoT devices, security reasons and to improve interoperability and depend on the type of credential being used since the ciphersuite concept also indicates the authentication and key exchange mechanism. LwM2M Clients and LwM2M Servers MAY support additional ciphersuites that conform to state-of-the-art security requirements.

###Bootstrapping

The Resources in the LwM2M Security Object (i.e., “Security Mode”, “Public Key or Identity”, “Server Public Key or Identity” and “Secret Key”) are used

1. for providing UDP channel security in “Client Registration”, “Device Management & Service Enablement”, and “Information Reporting” Interfaces if the LwM2M Security Object Instance relates to a LwM2M Server, or,

2. for providing channel security in Bootstrap Interface if the LwM2M Security Object instance relates to a LwM2M Bootstrap-Server.

3. for protecting the communication with a firmware repository server when the LwM2M Client receives a URI in the Package URI of the Firmware Update object.

The content and the interpretation of the Resources in the LwM2M Security Object depend on the type of credential being used.

Concerning Bootstrap from Smartcard a secure channel between the Smartcard and the LwM2M Client SHOULD be established, as described in Appendix G and defined in (TODO add links GLOBALPLATFORM 3 , GP SCP03). Using Smartcard with pre-shared secrets, raw public keys, and with certificates needs no pre-existing trust relationship between LwM2M Server(s) and LwM2M Client(s). The pre-established trust relationship is between the LwM2M Server(s) and the SmartCard(s).

LwM2M Clients MUST either be provisioned for use with a LwM2M Server (manufacturer pre-configuration bootstrap mode) or else be provisioned for use with an LwM2M Bootstrap-Server. Any LwM2M Client, which supports client or server initiated bootstrap mode, MUST support at least one of the following secure methods:

1. Bootstrapping with a strong (high-entropy) pre-shared secret, as described in Section "Pre-Shared Keys". The ciphersuites defined in Section 7.1.7 MUST NOT be used with a low-entropy secret or with a password.

2. Bootstrapping with a raw public key or certificate-based method (as described in TODO add links Section "Raw Public Keys" and Section "X509 Certificates".

In either case, the LwM2M Client MUST be provisioned with a credential that is unique to a device. For full interoperability, a LwM2M Bootstrap-Server MUST support bootstrapping via pre-shared secrets, raw public keys, and certificates.

NOTE: The above security methods can also be used by the LwM2M Bootstrap-Server to provision KIc and KID for the SMS Secured Packet Structure mode (see Section TODO add link 7.2.2 for SMS Secured Packet Structure mode).

Security credential dynamically provisioned to the LwM2M Client and the LwM2M Server MAY change at any time, even during the lifetime of an ongoing DTLS session. Since the DTLS protocol verifies the credentials only at the beginning of the session establishment (unless the re-negotiation feature is used) it is possible that a change in credential (for example, credentials for the use of a PSK-based ciphersuite) occurs after a DTLS handshake has already been completed and the DTLS session setup is already finalized. Hence, from a DTLS protocol point of view such a change is not recognized and the already established record layer security associations are in use. It is a policy decision for a DTLS client as well as a DTLS server implementation to tear down an already existing session when the credentials change. Such a decision will depend on various factors, such as the application domain in which LwM2M is used. The LwM2M specification does not mandate a specific behaviour in such a case since DTLS allows both communication parties to tear down an established DTLS session for any number of reasons.

###Endpoint Client Name

The LwM2M specification defines the use of the endpoint client name in the Bootstrap-Request and in the Register messages. Since the endpoint client name is not authenticated at the application layer the LwM2M Server MUST compare the received endpoint client name identifier with the identifier used at the DTLS handshake. This comparison may either be an equality match or may involve a dedicated lookup table to ensure that LwM2M Clients cannot intentionally or due to misconfiguration impersonate other LwM2M Clients. The LwM2M Server MUST respond with a “4.00 Bad Request” to the LwM2M Client if these fields do not match.


###LwM2M and DTLS Roles

The client-server roles of DTLS, which indicate who initiates the DTLS handshake, are independent from the client-server relationship of LwM2M. In client-initiated bootstrapping the LwM2M Client is also the DTLS client and the LwM2M Bootstrap Server acts as the DTLS server. For server-initiated bootstrapping, however, the roles are reversed: the LwM2M Client acts in the role of a DTLS server and the LwM2M Bootstrap Server is the DTLS client. Note that using a DTLS server on a LwM2M Client requires additional resources, such as RAM, and flash memory.

When the LwM2M Client acts in the role of a DTLS server then care has to be taken that the following four values are equal:

1. Value in the Server Name Indication (SNI) extension used in the DTLS exchange,

2. Endpoint Client Name,

3. Identifier used with the credential, such as the identifier contained in the DTLS server certificate, and

4. Value in the LwM2M Server URI Resource.

Note that the DTLS client (acting as the LwM2M Server) for the server-initiated bootstrapping has to be configured with the IP address of the LwM2M Client, an FQDN, and the certificate, raw public key or PSK for use with the LwM2M Client.

###Pre-Shared Keys

If a LwM2M Server supports the pre-shared key credentials it MUST support TLS_PSK_WITH_AES_128_CCM_8, as defined in {{RFC6655}} and mandated in {{RFC7925}}.

A LwM2M Client MUST support the Pre-Shared Key mode of DTLS with at least one of the ciphersuites specified for the LwM2M Server.

This mode requires the following resources of the Security Object defined in Appendix E.1 of TODO add link LWM2M-CORE) to be populated:

* The “Security Mode” Resource MUST contain the value 0.

* The "Public Key or Identity" Resource MUST be used to store the PSK identity, defined in {{RFC4279}}.

* The "Secret Key" Resource MUST be used to store the PSK, defined in {{RFC4279}}.

* The “Server Public Key” Resource MUST NOT be used in the Pre-Shared Key mode.

###Raw Public Keys

If a LwM2M Server supports the raw public key credentials it MUST support TLS_ECDHE_ECDSA_WITH_AES_128_CCM_8, as defined in {{RFC6655}} and mandated in {{RFC7925}}.

If a LwM2M Client supports the raw public key mode it MUST support at least one of the ciphersuites supported by the LwM2M Server.

This mode requires the following resources of the Security Object defined in Appendix E.1 of (TODO add link LWM2M-CORE) to be populated:

* The “Security Mode” Resource MUST contain the value 1.

* The "Public Key or Identity" Resource MUST be used to store the raw public key of the DTLS client.

* The "Secret Key" Resource MUST be used to store the private key of the DTLS client.

* The “Server Public Key” Resource MUST be used to store the raw public key of the DTLS server.

This security mode is appropriate for LwM2M deployments where the benefits of asymmetric cryptography are used but without the overhead of the public key infrastructure.

The DTLS client MUST check that the raw public key presented by the DTLS server exactly matches this stored public key.

The DTLS server MUST store its own private and public keys, and MUST have a stored copy of the expected client public key. The DTLS server MUST check that the raw public key presented by the DTLS client exactly matches this stored public key.

###X.509 Certificates

The X.509 Certificate mode requires the use of X.509v3 certificates {{RFC5280}}.

If a LwM2M Server supports X.509 Certificate mode it MUST support TLS_ECDHE_ECDSA_WITH_AES_128_CCM_8, as defined in {{RFC7251}} and mandated in {{RFC7925}}.

If a LwM2M Client supports X.509 Certificate mode it MUST support at least one of the above mentioned ciphersuites.

This mode requires the following resources of the Security Object defined in Appendix E.1 of (TODO add link LWM2M-CORE) to be populated:

* The “Security Mode” Resource MUST contain the value 2.

* The "Public Key or Identity" Resource MUST be used to store the X.509 certificate of the DTLS client, or optional its complete certification path (including client certificate but not the trust anchor).

* The "Secret Key" Resource MUST be used to store the private key of the DTLS client.

* The “Server Public Key” Resource MUST be used to store either a trust anchor certificate suitable for path validation of the certificate of the DTLS server, or directly the certificate of the DTLS server (“domain-issued certificate mode”). The use of it is explained in more detail below.

The "LwM2M Server URI", and the "Bootstrap Server" Resources are populated according to the description in Appendix E.1 of (TODO add link LWM2M-CORE).

The algorithm for verifying the service identity, as described in Section 4.4.1 of {{RFC7925}} and in {{RFC6125}}, is essential for ensuring proper security when certificates are used and MUST be implemented and used by the DTLS client. Terms like reference identifier and presented identifier are defined in {{RFC6125}}.

Comparing the reference identifier against the presented identifier obtained from the certificate is required to ensure the DTLS client is communicating with the intended DTLS server. To cater for the case that the Server Public Key Resource directly contains the certificate of the DTLS server, DTLS client does not compare the reference identifier against the presented identifier if the certificate from the Server Public Key Resource matches the certificate provided by the DTLS server during the DTLS handshake. If that is not the case, the DTLS client MUST compare the reference identifier against the presented identifier as described below, and perform path validation.

The algorithm description from RFC 6125 assumes that fully qualified DNS domain names (FQDN) are used. If a server node is provisioned with a FQDN, then the DTLS server certificate MUST contain this "FQDN". This FQDN is stored in the SubjectAltName or in Common Name (CN) component of the subject name, as explained in Section 9.1.3.3 of {{RFC7252}}, and used by the DTLS client to match it against the FQDN used during the lookup process, as described in {{RFC6125}}.

Note that the Server Name Indication (SNI) extension {{RFC6066}} allows a DTLS client to tell a DTLS server the name of the DTLS server it is contacting. This is an important feature when the server is part of a hosting solution where multiple virtual servers are using a single underlying network address. Section 3 of {{RFC6066}} only allows FQDN hostname of the DTLS server in the ServerName field. For the DTLS client running on a LwM2M Server the SNI extension allows the LwM2M Server to indicate what certificate it is expecting. The SNI extension MUST be set by the LwM2M client.

In some deployment scenarios DNS is not used and hence LwM2M Clients need to follow a different procedure.

If the CoAP URI stored in the "LwM2M Server URI" Resource contains an IP literal, such as `coaps://[2001:db8::2:1]/`, then certificate provided by the server MUST also contain this IP address in the Common Name (CN) component of the server certificate or in a field of URI type in the SubjectAltName set.

The procedure for a client using such certificates is as follows:

* The LwM2M Client uses the IP address from the LwM2M Server URI Resource to connect to the LwM2M Server using a DTLS handshake. The IP address becomes the reference identifier.

* The DTLS stack of the LwM2M Server returns a Certificate message as part of the handshake that contains a certificate. The IP address extracted from the server certificate becomes the presented identifier.

* The client matches the reference identifier against the presented identifier. If the two match, the client continues with the certificate verification according to RFC 5280, otherwise it aborts the handshake with a fatal alert.

Note: In case no FQDN but IP addresses are used ,and the IP address of the LwM2M server changes, the IP address contained in the LwM2M Server URI Resource will naturally also need to be updated. As a result, in case the “Server Public Key” contains a trust anchor certificate, a new certificate for that LwM2M Server needs to be created.

The use of certificates might require the DTLS client to understand the concept of time since it might need to check the validity of the server-provided certificate if required by the deployment. Different deployments may have different means of obtaining the current time and this specification does not mandate one mechanism. In general, the LwM2M Bootstrap-Server certificate is not expected to expire during the lifetime of the LwM2M Client since it has no easy possibility to recover from such an expired certificate. However, if the LwM2M Client determines that the LwM2M Server certificate is expired it MAY contact the LwM2M Bootstrap-Server to obtain new security credentials for use with the LwM2M Server.

Note that the LwM2M Device Object allows the LwM2M Bootstrap-Server to configure the current time for the LwM2M Client using the Current Time Resource.

###“NoSec” mode

It is highly recommended to always protect the LwM2M protocol with DTLS. There are, however, scenarios where the LwM2M protocol is deployed in environments where lower layer security mechanisms are provided.

The LwM2M Server MUST compare the endpoint client name identifier used during the Register and the Bootstrap-Request message with the identifier used for network access authentication (typically used to setup link layer security procedures).

The LwM2M protocol may use the NoSec mode with or without a lower-layer security mechanism and matching the endpoint client name identifier with any lower layer identifier may in the latter case not be possible.

###Certificate mode with EST

This mode uses the configuration of the certificate mode defined in Section 7.1.9 with the following changes; instead of generating the certificate and the private key for the client by the LwM2M Bootstrap Server and to provision it to the LwM2M Client the Bootstrap Server MUST set the “Security Mode” Resource to value 4 and provisions the certificate of the DTLS server to the “Server Public Key” Resource. This triggers the LwM2M Client to locally generate a public / private key pair on the LwM2M Client and to initiate an EST over CoAP protocol exchange (TODO add link to CoAP-EST) to obtain a certificate. The EST over CoAP specification (TODO add link to CoAP-EST) profiles the use of EST for use in constrained environments.

When generating a public / private key pair, the random generator used by the LwM2M Client MUST respect the characteristics of a sufficiently high quality random bit generator, such as defined for example by ISO/IEC 18031:2011, RFC 4086 {{RFC4086}} or NIST Special Publication 800-90a (TODO add link to SP800-90A).

Compared to the certificate mode additional over-the-air overhead is introduced by this mode since the LwM2M Client needs to convey the public key to the EST server and needs to demonstrate possession of the private key using the PKCS#10 defined mechanism, as referenced in the EST specification. Depending on the deployment environment this additional overhead needs to be compared against the added security benefit of not disclosing the private key to other parties.

The "Secret Key" and the "Public Key or Identity" Resources are not used by this mode. The "LwM2M Server URI", and the "Bootstrap Server" Resources are populated according to the description in Appendix E.1.

Enrollment over Secure Transport (EST) offers multiple features, including

* Simple PKI messages,

* CA certificate retrieval,

* CSR Attributes Request,

* Server-generated key request,

but only the first two are mandatory to implement.

In context of this specification functionality for server-generated key requests is already covered as part of the security mode (1 - Raw Public Key mode and 2 - Certificate mode). CSR Attributes Request is also not required for this specification either since the LwM2M Bootstrap Server is typically in possession of the required attributes for generating a certificate. The CA certificate retrieval, while mandatory to implement for EST, is not used by version 1.0 of this specification since only the domain issued certificate mode is supported, as described in Section 7.1.9. Hence, CA certificates are not utilized.

##SMS Channel Security

Channel security for [CoAP] has been defined for the UDP transport and is based on the Datagram Transport Layer Security (DTLS) {{RFC6347}}

This section defines the security modes for the transport of CoAP over SMS.

LwM2M Clients supporting SMS, when the SMS Channel is only used for debugging purposes MAY support the NoSec mode.

LwM2M Clients supporting UDP and SMS, when the SMS Channel is only used for triggering as defined in chapter 8.4 MUST support the adequate mechanism for securing UDP Channel as defined in chapter 7.1 UDP channel security. Those clients MAY use any SMS security mode. In particular SMS NoSec mode can be used for SMS triggering since all other communication will be secured by UDP channel security.

Using SMS NoSec for SMS triggering could induce issues as “Denial of Service” (DoS), SMS auto reply attacks (based on PoR:) and is strongly not recommended.

LwM2M Clients supporting SMS for communications other than triggering, or supporting only the SMS Channel MUST support SMS Secured Mode. In any security mode except for debugging purposes, when an SMS message is received from an MSISDN that is not recorded in the LwM2M Server SMS Number resource of the LwM2M Server Access Security, the SMS message MUST be silently ignored.

###SMS “NoSec” Mode

It is highly recommended to always use LwM2M with one of the security mechanisms described in this section. However, there are few scenarios and use cases where security is provided by lower layers. For example, LwM2M devices in a controlled environment behind a gateway, or, tests focussing first on other functions before performing end-to-end tests including security.

This security profile is also useful to support SMS triggering when all other exchanges run over UDP Channel.

###SMS Secured Mode

The SMS Secured mode specified in this section MUST be supported when the SMS binding is used.

A LwM2M Client which uses the SMS binding MUST either be directly provisioned for use with a target LwM2M Server (Factory Bootstrap or Bootstrap from Smartcard) or else be able to bootstrap via the UDP binding.

The end-point for the SMS channel (delivery of mobile terminated SMS, and sending of mobile originated SMS) MAY be either on the Smartcard or on the Device. When the LwM2M Client device doesn’t support a Smartcard, the end-point is on the LwM2M Client device.

A LwM2M Client, Server or Bootstrap-Server supporting SMS binding MUST discard SMS messages which are not correctly protected using the expected parameters stored in the “SMS Binding Key Parameters” Resource and the expected keys stored in the “SMS Binding Secret Keys” Resource, and MUST NOT respond with an error message secured using the correct parameters and keys.

###Device End-Point

The Secured Packet Structure is based on (TODO add ref to 3GPP TS 31 115 and ETSI TS 102 225) which was originally designed for securing packet structures for UICC based applications. However, for LwM2M it is suitable for securing the SMS payload exchanged between client and server. Usage of Secured Packet Structure Packet mode in LwM2M device needs evolution towards the introduction of a secure environment. The intention is to evolve the specifications in the next LwM2M release.

In LwM2M Enabler 1.0 if the SMS channel end-point is on the Device, the Channel security for [CoAP] is based on the Datagram Transport Layer Security (DTLS) {{RFC6347}}. For that reason the main lines of Section 7.1 on “DTLS-based Security” relative to DTLS binding on CoAP are also applicable to that section.

Appendix A of {{RFC7925}} describes how to bind CoAP/DTLS message to the SMS channel and specifies the restrictions on DTLS for fitting the SMS channel specific functioning and narrow bandwidth.

**Header Definitions (for one SMS)**

a) SMS Frame for basic Request/Response Interaction message (no Token field required) {{smsframe}}.

~~~~
+-------------------------------------------------------------------------------------+
|                              TPDU (140 bytes)                                       |
|---------------------------------+---------------------------------------------------|
| DTLS (29 bytes)                 | CoAP + Effective Payload                          |
|---------------------------------+---------------------------------------------------|
| Header (13), Nonce (8), IV (8)  |                                                   |
|---------------------------------+---------------------------------------------------|
|                                 | CoAP ( 4 bytes)  | Effective Payload ( 107 bytes) |
+---------------------------------+---------------------------------------------------+
~~~~
{: #smsframe title='Basic SMS Frame' artwork-align="left"}

Model calculation using these header definitions, overall TPDU : 140 bytes

* DTLS requires 29 bytes: 13 bytes header according to (RFC 6347 and Appendix B of {{RFC7925}} + 8 bytes for the explicit nonce and 8 bytes for the integrity check value when an AES-128-CCM-8 ciphersuite is used. This ciphersuite uses a short integrity check value.

* CoAP header of variable length with at least 4 bytes {{RFC7252}}.

* Available bytes for the effective LwM2M payload from one SMS: 107 bytes.

b) SMS Frame for messages of the Information Reporting Interface (Token field required) {{smsframe2}}.

~~~~
+-------------------------------------------------------------------------------------+
|                              TPDU (140 bytes)                                       |
|---------------------------------+---------------------------------------------------|
| DTLS (29 bytes)                 | CoAP + Effective Payload                          |
|---------------------------------+---------------------------------------------------|
| Header (13), Nonce (8), IV (8)  |                                                   |
|---------------------------------+---------------------------------------------------|
|                                 | CoAP ( 4+8 bytes) | Effective Payload ( 99 bytes) |
+---------------------------------+---------------------------------------------------+
~~~~
{: #smsframe2 title='SMS Frame for Information Reporting Interface' artwork-align="left"}

Model calculation using these header definitions,

* DTLS takes 29 bytes: 13 bytes {{RFC6347}} of header + 16 bytes of integrity check for CoAP in DTLS {{RFC6655}} . Cipher suite mandated by CoAP (AES-128).

* CoAP header 4+8 {{RFC7252}} (Token field required).

* Available bytes for the effective LwM2M Payload from one SMS: 99 bytes.

###Smartcard End-Point

If the SMS channel end-point is on the smart card, a CoAP message as defined in {{RFC7252}} MUST be encapsulated in (TODO 3GPP 31.115) Secured Packets, in implementing - for SMS Point to Point (SMS_PP) - the general (TODO ETSI 102 225) specification for UICC based applications.

The following settings MUST be applied:

Class 2 SMS as specified in (TODO 3GPP TS 23.038). The (TODO 3GPP TS 23.040) SMS header MUST be defined as below:

* TP-PID : 111111 (USIM Data Download) as specified in (TODO 3GPP TS 23.040)

* TP-OA : the TP-OA (originating address as defined in (TODO 3GPP 23.040) of an incoming command packet (e.g CoAP request) MUST be re-used as the TP-DA of the outgoing packet (e.g CoAP response)

**Secure SMS Transfer to UICC**

A SMS Secured Packet encapsulating a CoAP request received by the LwM2M device, MUST be – according to (TODO ETSI TS 102 225 and 3GPP TS 31.115) - addressed to the LwM2M UICC Application in the Smartcard where it will be decrypted, aggregated if needed, and checked for integrity.

If decryption and integrity verification succeed, the message contained in the SMS MUST be provided to the LwM2M Client.

If decryption or integrity verification failed, SMS MUST be discarded.

The mechanism for providing the decrypted CoAP Request to the LwM2M Client relies on basic GET_DATA commands of (TODO GP SCP03) .This data MUST follow the format as below:

~~~~
  data_rcv _ ::= <address> <coap_msg>

  address ::= TP_OA ; originated address

  coap_msg ::= CoAP_TAG <coap_request_length> <coap_request>

  coap_request_length ::= 16BITS_VALUE

  coap_request ::= CoAP message payload
~~~~
{: #sampledata title='Sample Data'artwork-align="left"}


NOTE: In current LwM2M release, the way the LwM2M Client Application is triggered for retrieving the available message from the Smartcard is device specific: i.e. a middle class LwM2M Device implementing (TODO ETSI TS 102 223) ToolKit with class “e” and “k” support could be automatically triggered by Toolkit mechanisms, whereas a simpler LwM2M device could rely on a polling mechanisms on Smartcard for fetching data when available.

**Secured SMS Transfer to LwM2M Server**

For sending a CoAP message to the LwM2M Server, the LwM2M Client prepares a data containing the right TP-DA to use, concatenated with the CoAP message and MUST provide that data to the LwM2M UICC Application in using the [GP SCP03] STORE-DATA command.

According to (TODO ETSI TS 102 225 and 3GPP TS 31.115) the Smartcard will be in charge to prepare (encryption / concatenation) the CoAP message before sending it as a SMS Secure Packet (TODO ETSI TS 102 223) SEND_SMS command).

The SMS Secured Packet MUST be formatted as Secured Data specified in section TODO 7.2.2.3.

The Secure Channel as specified in Appendix H of this document SHOULD be used to provide the prepared data to the Smartcard.

###SMS Secured Packet Binding for CoAP messages

In SMS Secured Packet Structure mode, a CoAP message as defined in {{RFC7252}} MUST be encapsulated in (TODO 3GPP 31.115) Secured Packets, in implementing - for SMS Point to Point (SMS_PP) - the general (TODO ETSI 102 225) specification for UICC based applications.

* The “Command Packet” command specified in (TODO 3GPP 31.115)/(TODO ETSI 102 225) MUST be used for both CoAP Request and Response message

* The Structure of the Command Packet contained in the Short Message MUST follow (TODO 3GPP 31.115) specification

* SPI MUST be set as follow (see coding of SPI in (TODO ETSI TS 102 225) section 5.2.1):

  * use of cryptographic checksum

  * use of ciphering

    * The ciphering and crypto graphic checksum MUST use either AES or Triple DES

    * Single DES MUST NOT be used

    * AES SHOULD be used

    * When Triple DES is used , then it MUST be used in outer CBC mode and 3 different keys MUST be used

    * When AES is used it MUST be used with CBC mode for ciphering (see coding of KIc in (TODO ETSI TS 102 225) section 5.2.2) and in CMAC mode for integrity (see coding of KID in (TODO ETSI TS 102 225) section 5.2.3).

  * process if and only if counter value is higher than the value in the RE

  * PoR depends on LwM2M Server Policy

* TAR MUST be set to ‘B2 02 03’ value for the LwM2M UICC Application as registered in (TODO ETSI TS 101 220) Appendix D

* Secured Data : contains the Secured Application Message which MUST be coded as a BER-TLV, the Tag (TBD : e.g 0x05) will indicate the type (e.g CoAP type) of that message.

##LPWA Security

Low Power Wide Area (LPWA) networks are dedicated networks for communications with very resource constrained devices. Such devices are often battery driven and have limited processing capabilities.

LWM2M can be deployed as a “thin” service layer providing security in LPWA scenarios since LWM2M has been designed to keep the requirements of constrained devices in mind. However, some of the LPWA radio technologies offer their own link layer security mechanisms, which need to be considered when offering additionally LWM2M security based on DTLS. For example, when deploying link layer security as well as DTLS together the resulting double encryption (for some part of communication path) will result in higher power consumption and additional transmission overhead, which might not be acceptable for a range of battery driven devices.

In case the LPWA network offers link layer security and the threat analysis concluded that no additional communication security at higher layers, such as with DTLS, is necessary, the LWM2M “NoSec” mode MAY be used. Protecting LWM2M communication using DTLS remains a deployment choice. Protocol designers may need to take into account that the link layer security mechanism typically terminates at a different node than security mechanisms offered at higher layers and solely relying on link layer security may leave some segment of the communication path unprotected.

Examples of LPWA network security mechanisms can, for example, be found in TS 33.401 “SAE, Security architecture” describes the keys and processes for Narrow band IoT (TODO NB-IoT) security based on what is called “end-to-middle (e2m) security” from the device to the 3GPP network.

When using LWM2M security based on DTLS in a LPWA environment it is recommended to consider the work done in IETF on “DTLS In Constrained Environments” (TODO DICE), see (TODO REF draft-ietf-dice-profile-17). (TODO REF draft-ietf-dice-profile-17) does not introduce any changes to DTLS and TLS but rather offers guidance for use of various extensions for increased interoperability, and gives recommendations for improving the handshake procedures.

(TODO REF draft-ietf-dice-profile-17) gives recommendations for three types of credentials, namely pre-shared keys, raw public keys, and X.509 certificates. LWM2M works with all three types of credentials but the performance and security trade-offs for these three mechanisms are different. As a summary, the three credential types have the following properties:

* The pre-shared key profile offers the most efficient solution for integration of DTLS into LWM2M since DTLS pre-shared ciphersuites recommended in (TODO REF draft-ietf-dice-profile-17) are computationally efficient (since they use the most efficient cryptographic primitives), and require a minimum amount of flash as well as RAM. The size of the exchanged messages is also kept at a minimum. There is, however, a downside as well: symmetric keys need to be available to both communication endpoints.

* The certificate-based profile re-uses widely used X.509 certificates. This allows both tools as well as existing infrastructure, such as Certification Authorities (CAs), to be re-used. Unlike the typical web browser use of certificates the DICE profile (TODO REF draft-ietf-dice-profile-17) uses certificates for clients and servers. The use of certificates comes at a price. The use of asymmetric cryptography is more complex to implement, requires more bandwidth for the exchanged messages, is computationally more demanding, and requires a larger code size as well as more RAM. The benefits are, in addition to the re-use of existing technologies, the need to only share the certificates (and the public key that is contained inside the certificate) with other communication partners and to keep the private key local to each party. This property of asymmetric cryptography reduces the risk of exposing private keying material.

* The raw public key profile offers features that sit between the pre-shared key and the certificate-based profile and combines the benefits of these two profiles. The use of asymmetric cryptography offers improved security but avoids the overhead associated with certificates and the PKI.

For purpose of DTLS usage with LWM2M over LWPAN this specification RECOMMENDs the implementation and use of the pre-shared key profile primarily due to the over-the-wire communication overhead. Deployments MAY implement other profiles as well. The subsequent text summarizes the key aspects of the pre-shared key profile described in (TODO REF draft-ietf-dice-profile-17) which is based on TLS_PSK_WITH_AES_128_CCM_8 that uses the AES-128-based without offering perfect forward secrecy:

* The Maximum Fragment Length extension described in Section 15 of (TODO  REF draft-ietf-dice-profile-17) allows a client to lower their RAM requirements and client implementations MUST implement this extension. Without this extension a client is required to maintain a maximum buffer size of 16KB.

* Session resumption, described in Section 7 of (TODO REF draft-ietf-dice-profile-17), offers slightly improved performance for a PSK-based ciphersuite and is RECOMMENDED. Session resumption allows a client to abbreviate the handshake based on session state established in an executed full handshake. This results in fewer messages and smaller message sizes. It is therefore RECOMMENDED to maintain session state information as long as possible (consistent with the security requirement to protect session key material on both Client and Server; e.g. a long-lived session key must be managed at least as securely as an underlying pre-shared key).

* Compression offered by DTLS is NOT RECOMMENDED due to security attacks, as described in Section 8 of (TODO REF draft-ietf-dice-profile-17). Compression functionality is better offered by higher layer protocols and various components used in LWM2M make use of compression techniques, such as CoAP with header compression, and the binary encoding of payloads.
The timeout recommendations provided in Section 11 of (TODO REF draft-ietf-dice-profile-17) MUST be followed since the modified timer settings prevent spurious retransmissions. Failure to increase the timeout value can lead to failed protocol exchanges.

* A number of DTLS extensions are not applicable or are not recommended for use with the PSK-based ciphersuite and the recommendations made throughout (TODO REF draft-ietf-dice-profile-17) have to be taken into account. Note that the use of False Start, described in Section 21 of (TODO REF draft-ietf-dice-profile-17), is not required since the ability to transmit application data earlier is less important with long-lived DTLS sessions.

The guidance for credential-based profile can be found in Section 4.4 of (TODO REF draft-ietf-dice-profile-17) and guidance for the raw public key profile can be found in Section 4.3 of (TODO REF draft-ietf-dice-profile-17). Both profiles use Elliptic Curve Cryptography algorithms and offer perfect forward secrecy, as described in Section 9 of (TODO REF draft-ietf-dice-profile-17).

# IANA Considerations

## Resource Type (rt=) Link Target Attribute Values Registry

TODO This document should add the various (rt=) Link Attributes that it uses. It adds the following resource type to the "Resource Type (rt=) Link Target Attribute Values", within the "Constrained RESTful Environments (CoRE) Parameters" registry.

| Value               | Description          | Reference |
| omadm.valuexxx      | OMA value            | RFC XXXX  |
| omadm.valuexxx      | OMA value            | RFC XXXX  |
| omadm.valuexxx      | OMA value            | RFC XXXX  |
{: align="left"}

// RFC Ed.: replace RFC XXXX with this RFC number and remove this note. OMA should request that to IANA.

## CoAP Content-Formats Registry

TODO This document adds the following Content-Format to the "CoAP Content-Formats", within the "Constrained RESTful Environments (CoRE) Parameters" registry.

| Media Type                        | Excoding ID  | Reference |
| application/oma+JSON              | XXX          | RFC XXXX  |
| application/oma+CBOR              | XXX          | RFC XXXX  |
| application/oma+TLV               | XXX          | RFC XXXX  |
| application/oma+SenML             | XXX          | RFC XXXX  |
{: align="left"}

// RFC Ed.: replace XXX with assigned IDs and remove this note.
// RFC Ed.: replace RFC XXXX with this RFC number and remove this note.


# Acknowledgements

The authors would like to thank Jaime Jiménez for his work as editor of this document.
They would also like to thank the OMA organization for providing support to make this work possible.

--- back

# Appendix 1: TITLE {#appendix1}

askdjhasdkjhaskdjhaskjdhkajshdkjahsd
asdlkjasldkjasd
alskdjlaksjdlkajsd
aslkdaslkdj

# OMA DM example Objects {#example-objects}

This appendix shows .....
