---
title: "DNS Queries over CoAPS (DoC)"
abbrev: DoC
docname: draft-lenders-dns-over-coaps-latest
category: std

ipr: trust200902
area: Applications
workgroup: CoRE
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -  name: Martine Sophie Lenders
    org: Freie Universität Berlin
    abbrev: FU Berlin
    email: m.lenders@fu-berlin.de
 -  name: Christian Amsüss
    email: christian@amsuess.com
 -  name: Cenk Gündoğan
    org: HAW Hamburg
    email: cenk.guendogan@haw-hamburg.de
 -  name: Thomas C. Schmidt
    org: HAW Hamburg
    email: t.schmidt@haw-hamburg.de
 -  name: Matthias Wählisch
    org: Freie Universität Berlin
    abbrev: FU Berlin
    email: m.waehlisch@fu-berlin.de

normative:

informative:


--- abstract

The Constrained Application Protocol (CoAP) is the de-facto transfer protocol
for the Internet of Things (IoT). Besides providing REST capabilities for
constrained nodes it provides an encrypted transport utilizing DTLS known as
CoAPS. This transport can be used for encrypted DNS communication within the
IoT. This documents defines how to transport DNS queries and responses via
CoAPS.


--- middle

Introduction
============

With DNS over HTTPS {{?RFC8484}} (DoH) a confidential transport for DNS messages
was specified. However, the use cases provided in that document, preventing
on-path devices to interfere with DNS operations and getting safe and consistent
access to DNS information within the application space also apply for
constrained devices. The constraints on these types devices {{?RFC7228}}
typically do not allow for a HTTPS deployment. The Constrained Application
Protocol (CoAP) {{!RFC7252}}, on the other hand, provides RESTful APIs for such
devices, including a secure version over DTLS {{!RFC6347}}, CoAPS.

This document defines a protocol, DNS over CoAPS (DoC), to send DNS {{!RFC1035}}
queries and get DNS responses via CoAPS (including DTLS security for integrity
and confidentiality). Each DNS query-response pair is mapped into a CoAP message
exchange.

Usage of additional feature sets of CoAP/CoRE to deploy and gather DNS
information such as caching, block-wise transfer, and resource discovery are
discussed.

Terminology
-----------

A server that provides the service specified in this document is called a "DoC
server" to differentiate it from a classic "DNS server". Correspondingly, a
client using this protocol to retrieve the DNS information is called a "DoC
client".

The term "constrained nodes" is used as defined in {{?RFC7228}}.

{::boilerplate bcp14-tagged}

Selection of a DoC server
=========================
To be able to reuse configuration mechanisms provided for DoH, a DoC client is
configured with a URI Template {{!RFC6570}} as well. Likewise, MAY a DoC server
support more than one URI Template to provide different properties.

The URI Template SHOULD provide a variable "dns" so that GET requests can be
used to retrieve the DNS information. If the "dns" variable is not provided in
the URI Template, GET requests can not be used for DoC exchanges.

TBD DoC server identity, key exchange, ...

URI template alternatives
-------------------------
TBD:

- CRI {{?I-D.ietf-core-href}} or CoRAL {{?I-D.ietf-core-coral}}

CoAP Messaging
==============

Queries
-------

A DoH client encodes a single DNS query in one or more CoAP request messages
using either the CoAP GET, FETCH {{!RFC8132}}, or POST method. More than one
CoAP request message MAY be used if the FETCH or POST method is used and
block-wise transfer {{!RFC7959}} is supported by the client. If more than one
CoAP request message is used to encode the DNS query must be chained together
using the Block1 option in those CoAP requests. To make use of the recovery
mechanism of CoAP, the CoAP request SHOULD be carried in a Confirmable (CON)
message.

For a POST or FETCH request the URI Template specified in
[](#selection-of-a-doc-server) is processed without any variables set. For a GET
request the URI Template is extended with the "dns" variable set to the content
of the DNS query, encoded with `base64url` {{!RFC4648}}.

If new Content Formats are specified in the future, the specification MUST
define the variable used in the URI Template with that new format.

For POST and FETCH methods, the DNS query is included in the payloads of the
CoAP request messages in the binary format as specified in {{!RFC1035}} and
Content Format option MUST be included to indicate the message type as
"application/dns-message". Due to the lack of encoding requirements, both FETCH
and POST methods are generally smaller than GET requests.

A DoH server MUST implement both the GET and POST method and MAY implement the
FETCH method.

Using GET enables CoAP proxies en-route to the DoC server to cache a successful
response.
However, as the DNS query is carried in the URI and thus in one of the URI-\*
options within a GET request, block-wise transfer can not be used with that
method.
As a cache-friendly alternative, the FETCH method can be used, which is an
extension to legacy CoAP, specified in {{!RFC8132}}.

Requests of either method type SHOULD include an Accept option to indicate what
type of content can be parsed in the response. A client MUST be able to parse
messages of Content Format "application/dns-message" regardless of the provided
Accept option. Messages of that Content Format are DNS responses in binary
format as specified in {{!RFC1035}}.

To simplify cache-key calculations at the CoAP proxies en-route, DoH clients
using Content Formats that include the ID field from the DNS message, such as
"application/dns-message", SHOULD use DNS ID 0 in every DNS query. The CoAP
message ID takes the same function on the CoAP layer. Dedicated identification
of DNS message exchanges on the wire is thus not necessary.

TBD: ETag option?

### Examples

TBD

Responses
---------
This document specifies responses of Content Format "application/dns-message"
which encodes the DNS response in the binary format, specified in {{!RFC1035}}.
For this type of responses, the Content Format option indicating the
"application/dns-message" format MUST be included.
A DoC server MUST be able to parse requests of Content Format
"application/dns-message".

Each DNS query-response pair is mapped to a train one or more of CoAP
request-response pairs. If supported, a DoC server MAY transfer the DNS response
in more than one CoAP response using the Block2 option {{!RFC7959}}.

### Response Codes and Handling DNS and CoAP errors

A DNS response indicates either success or failure for the DNS query. As such,
SHOULD CoAP responses carrying any valid DNS response, use a 2.xx Success
response code. GET and FETCH requests SHOULD be responded to with a 2.05 Content
response. POST requests SHOULD be responded to with a 2.01 Created response.

CoAP responses with non-successful response codes MUST NOT contain any payload
and may only be used on errors in the CoAP layer or when a request does not
fulfill the requirements of the DoC protocol.

For consistency, communications errors with an upstream DNS server such as
timeouts SHOULD be indicated with a SERVFAIL DNS response in a successful CoAP
response.

A DoC client might try to repeat a non-successful exchange unless otherwise
prohibited. For instance, a FETCH request MUST NOT be repeated with a URI
Template for which the DoC server already responded with a 4.05 Method Not
Allowed, as the server might only implement legacy CoAP and does not support the
FETCH method. The DoC client might also elect to repeat a non-successful
exchange with a different URI Template, for instance, when the response
indicates an unsupported content format.

### Examples

- ...
- MUSTs for DoC not full-filled by CoAP request: `4.00 Bad Request`
- Content Format in Accept option can not be generated: `4.06 Not Acceptable`
- Content Format of payload is not as expected: `4.15 Unsupported
  Content-Format`

CoAP/CoRE Integration
=====================

Proxies and caching
-------------------
- responses SHOULD have Max-Age option (set to minimum TTL from DNS response)
- Cache SHOULD support FETCH ("the request body is part of the cache key"
  {{!RFC8132}})

OBSERVE (modifications)?
------------------------
- DoH has considerations on Server Push to deliver additional, potentially
  outstanding requests + response to the DoC client for caching
- OBSERVE does not include the request it would have been generated from ==>
  cannot be cached without corresponding request having been send over the wire.
- If use case exists: extend OBSERVE with option that contains "promised" request
  (see {{?RFC7540}}, section 8.2)?
- Other caveat: clients can't cache, only proxys

OSCORE
------
- TBD

URI template configuration
==========================
- Maybe out-of-scope?
- DHCP and RA options to deliver? {{?I-D.peterson-doh-dhcp}}
- CoRE-RD {{?I-D.ietf-core-resource-directory-28}}...

Considerations for Unencrypted Use
==================================
- DTLS-transport should be used
- Non-DTLS can have benefits: Blockwise-transfer for IEEE 802.15.4, additional
  layer of caching, ...

Security Considerations
=======================

TODO Security


IANA Considerations
===================

IANA is requested to assign CoAP Content-Format ID for the DNS message media
type in the "CoAP Content-Formats" sub-registry, within the "CoRE Parameters"
registry {{!RFC7252}}, corresponding the "application/dns-message" media
type from the "Media Types" registry:

Media-Type: application/dns-message

Encoding: -

Id: TBD

Reference: [TBD-this-spec]

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
