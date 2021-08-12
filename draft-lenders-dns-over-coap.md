---
title: "DNS Queries over CoAP (DoC)"
abbrev: DoC
docname: draft-lenders-dns-over-coap-latest
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

This document defines a protocol for sending DNS messages over the
Constrained Application Protocol (CoAP). These CoAP messages are protected
by DTLS-Secured CoAP (CoAPS) or Object Security for Constrained RESTful
Environments (OSCORE) to provide encrypted DNS message exchange for
constrained devices in the Internet of Things (IoT).

--- middle

Introduction
============

This document defines DNS over CoAP (DoC), a protocol to send DNS
{{!RFC1035}} queries and get DNS responses over the Constrained Application
Protocol (CoAP) {{!RFC7252}}. Each DNS query-response pair is mapped into a
CoAP message exchange. Each CoAP message is secured by DTLS {{!RFC6347}} or
Object Security for Constrained RESTful Environments (OSCORE) {{!RFC7252}}
to ensure message integrity and confidentiality.

The application use case of DoC is inspired by DNS over HTTPS {{?RFC8484}}
(DoH). DoC, however, aims for the deployment in the constrained Internet of
Things (IoT), which usually conflicts with the requirements introduced by
HTTPS.

To prevent TCP and HTTPS resource requirements, constrained IoT devices
could use DNS over DTLS {{?RFC8094}}. In contrast to DNS over DTLS, DoC
utilizes CoAP features to mitigate drawbacks of datagram-based
communication. These features include: block-wise transfer, which solves
the Path MTU problem of DNS over DTLS (see {{?RFC8094}}, section 5); CoAP
proxies, which provide an additional level of caching; re-use of data
structures for application traffic and DNS information, which saves memory
on constrained devices.

~~~ drawing

                - GET coaps://[2001::db8::1]/?dns=example.org
               /- POST/FETCH coaps://[2001::db8::1]/
              /
             CoAP request
+--------+   [DNS query]   +--------+  DNS query   +--------+
|  DoC   |---------------->|  DoC   |.............>|  DNS   |
| Client |<----------------| Server |<.............| Server |
+--------+  CoAP response  +--------+ DNS response +--------+
            [DNS response]

~~~
{: #fig-overview-arch title="Basic DoC architecture"}


The most important components of DoC can be seen in {{fig-overview-arch}}: A DoC
client tries to resolve DNS information by sending DNS queries carried within
CoAP requests to a DoC server. That DoC server may or may not resolve that DNS
information itself by using other DNS transports with an upstream DNS server.
The DoC server then replies to the DNS queries with DNS responses carried within
CoAP responses.

TBD: additional feature sets of CoAP/CoRE

- resource directory for DoC service discovery,
- ...

Terminology
===========

A server that provides the service specified in this document is called a "DoC
server" to differentiate it from a classic "DNS server". Correspondingly, a
client using this protocol to retrieve the DNS information is called a "DoC
client".

The term "constrained nodes" is used as defined in {{?RFC7228}}.

{::boilerplate bcp14-tagged}

Selection of a DoC Server
=========================
A DoC client is configured with a URI Template {{!RFC6570}}. This allows us to
reuse configuration mechanisms provided for DoH.

The URI Template SHOULD provide a variable "dns" so that GET requests can be
used to retrieve the DNS information. If the "dns" variable is not provided in
the URI Template, GET requests can not be used for DoC exchanges.

TBD:

- Support for more than one URI Template by DoC server.
- DoC server identity, key exchange, ...

URI Template Alternatives
-------------------------
TBD:

- CRI {{?I-D.ietf-core-href}} or CoRAL {{?I-D.ietf-core-coral}}

Basic Message Exchange
======================

DNS Queries in CoAP Requests
----------------------------

A DoC client encodes a single DNS query in one or more CoAP request messages
using either the CoAP GET, FETCH {{!RFC8132}}, or POST method. More than one
CoAP request message MAY be used if the FETCH or POST method are used and
block-wise transfer {{!RFC7959}} is supported by the client. If more than one
CoAP request message is used to encode the DNS query, it must be chained
together using the Block1 option in those CoAP requests. To make use of the
recovery mechanism of CoAP, the CoAP request SHOULD be carried in a Confirmable
(CON) messages.

For a POST or FETCH request the URI Template specified in
[](#selection-of-a-doc-server) is processed without any variables set. For a GET
request the URI Template is extended with the "dns" variable set to the content
of the DNS query, encoded with `base64url` {{!RFC4648}}.

If new Content Formats are specified in the future, the specification MUST
define the variable used in the URI Template with that new format.

For POST and FETCH methods, the DNS query is included in the payloads of the
CoAP request messages in the binary format as specified in {{!RFC1035}}. The
Content Format option MUST be included to indicate the message type as
"application/dns-message". Due to the lack of encoding requirements, both FETCH
and POST methods are generally smaller than GET requests.

A DoC server MUST implement both the GET and POST method and MAY implement the
FETCH method.

Requests of either method type SHOULD include an Accept option to indicate what
type of content can be parsed in the response. A client MUST be able to parse
messages of Content Format "application/dns-message" regardless of the provided
Accept option. Messages of that Content Format are DNS responses in binary
format as specified in {{!RFC1035}}.

### Support of CoAP Caching

The DoC client SHOULD set the ID field of the DNS header always to 0 to
enable a CoAP cache (e.g., a CoAP proxy en-route) to respond to the same
DNS queries with a cache entry. This ensures that the CoAP Cache-Key (for
GET see {{!RFC7252}} Section 5.6, for FETCH see {{!RFC8132}} Section 2)
does not change when multiple DNS queries for the same DNS data, carried in
CoAP requests, are issued. Technically, using the POST method does not
require the DNS ID set to 0 because the payload of a POST message is not
part of the Cache-Key. For consistency reasons, however, it is RECOMMENDED
to use the same constant DNS ID.

### Examples

The following examples illustrate the usage of different CoAP messages to
resolve "example.org. IN AAAA" based on the URI template
"coaps://[2001:db8::1]/{?dns}". The CoAP body is encoded in
"application/dns-message" Content Format.

GET request:

    GET coaps://[2001::db8::1]/
    URI-Query: dns=AAABIAABAAAAAAAAB2V4YW1wbGUDb3JnAAAcAAE
    Accept: application/dns-message

POST request:

    POST coaps://[2001::db8::1]/
    Content-Format: application/dns-message
    Accept: application/dns-message
    Payload: 00 00 01 20 00 02 00 00 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             01 00 01                                        [binary]

FETCH request:

    FETCH coaps://[2001::db8::1]/
    Content-Format: application/dns-message
    Accept: application/dns-message
    Payload: 00 00 01 20 00 02 00 00 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             01 00 01                                        [binary]


DNS Responses in CoAP Responses
-------------------------------
This document specifies responses of Content Format "application/dns-message"
which encodes the DNS response in the binary format, specified in {{!RFC1035}}.
For this type of responses, the Content Format option indicating the
"application/dns-message" format MUST be included.
A DoC server MUST be able to parse requests of Content Format
"application/dns-message".

Each DNS query-response pair is mapped to a train of one or more of CoAP
request-response pairs. If supported, a DoC server MAY transfer the DNS response
in more than one CoAP response using the Block2 option {{!RFC7959}}.

### Response Codes and Handling DNS and CoAP errors

A DNS response indicates either success or failure for the DNS query. As such,
it is RECOMMENDED that CoAP responses that carry any valid DNS response, use a
2.xx Success response code. GET and FETCH requests SHOULD be responded to with a
2.05 Content response. POST requests SHOULD be responded to with a 2.01 Created
response.

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

### Support of CoAP Caching

It is RECOMMENDED to set the Max-Age option of a response to the minimum TTL in the Answer section of a DNS response. This prevents expired records unintentionally being served from a CoAP cache.

It is RECOMMENDED that DoC servers set an ETag option on large responses (TBD: more concrete guidance) that have a short Max-Age relative to the expected clients' caching time.
Thus, clients that need to revalidate a response can do so using the established ETag mechanism.
<!--
With short responses, a usable ETag might be almost as long as the response.
With long-lived responses, the client does not need to revalidate often.
-->
With responses large enough to be fragmented,
it's best practice for servers to set an ETag anyway.

### Examples

The following examples illustrate the replies to the query "example.org. IN
AAAA record", recursion turned on. Successful responses carry one answer
record including address 2001:db8:1::1:2:3:4 and TTL 58719.

A successful response to a GET or FETCH request:

    2.05 Content
    Content-Format: application/dns-message
    Max-Age: 58719
    Payload: 00 00 81 a0 00 01 00 01 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             1c 00 01 00 01 37 49 00 10 20 01 0d b8 00 01 00 [binary]
             00 00 01 00 02 00 03 00 04                      [binary]

A successful response to a POST request uses a different response code:

    2.03 Created
    Content-Format: application/dns-message
    Max-Age: 58719
    Payload: 00 00 81 a0 00 01 00 01 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             1c 00 01 00 01 37 49 00 10 20 01 0d b8 00 01 00 [binary]
             00 00 01 00 02 00 03 00 04                      [binary]

When a DNS error (SERVFAIL in this case) is noted in the DNS response, the CoAP
request still indicates success:

    2.05 Content
    Content-Format: application/dns-message
    Payload: 00 00 81 a2 00 01 00 00 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01          [binary]

When an error occurs on the CoAP layer, the DoC server SHOULD respond with
an appropriate CoAP error, for instance "4.15 Unsupported Content-Format"
if the Content Format option in the request was not set to
"application/dns-message".

CoAP/CoRE Integration
=====================

Proxies and caching
-------------------

TBD:

- [TTL vs. Max-Age](https://github.com/anr-bmbf-pivot/draft-dns-over-coaps/issues/5)
- Responses that are not globally valid
- General CoAP proxy problem, but what to do when DoC server is a DNS proxy,
  response came not yet in but retransmission by DoC client was received (see
  {{rt-problem}})
    - send empty ACK ([maybe move to best practices appendix](https://github.com/anr-bmbf-pivot/draft-dns-over-coaps/issues/6#issuecomment-895880206))

~~~ drawing
      DoC client           DoC proxy           DNS server
           |  CoAP req [rt 1]  |                    |
           |------------------>|  DNS query [rt 1]  |
           |                   |------------------->|
           |  CoAP req [rt 2]  |                    |
           |------------------>|      DNS resp      |
           |     CoAP resp     |<-------------------|
           |<------------------|                    |
           |                   |                    |
~~~
{: #rt-problem title="CoAP retransmission (rt) is received before DNS query could have been
fulfilled."}

OBSERVE (modifications)?
------------------------
- TBD
- DoH has considerations on Server Push to deliver additional, potentially
  outstanding requests + response to the DoC client for caching
- OBSERVE does not include the request it would have been generated from ==>
  cannot be cached without corresponding request having been send over the wire.
- If use case exists: extend OBSERVE with option that contains "promised" request
  (see {{?RFC7540}}, section 8.2)?
- Other caveat: clients can't cache, only proxys so value needs to be evaluated
- Potential use case: {{?RFC8490}} Section 4.1.2

OSCORE
------
- TBD
- With OSCORE DTLS might not be required

URI template configuration
==========================
- TBD
- Maybe out-of-scope?
- DHCP and RA options to deliver? {{?I-D.peterson-doh-dhcp}}
- CoRE-RD {{?I-D.ietf-core-resource-directory}} (...; can not express URI templates)
- When no actual templating is involved: regular resource discovery (`rt=core.dns`?) through .well-known/core

Considerations for Unencrypted Use
==================================
- TBD
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

# Change Log

TBD:

- [Reconsider usage of GET/POST](https://github.com/anr-bmbf-pivot/draft-dns-over-coaps/issues/2)?
- [Request text duplication](https://github.com/anr-bmbf-pivot/draft-dns-over-coaps/issues/4)

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
