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
Object Security for Constrained RESTful Environments (OSCORE) {{!RFC8613}}
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

To prevent resource requirements of DTLS or TLS on top of UDP (e.g.,
introduced by DNS over QUIC {{?I-D.ietf-dprive-dnsoquic}}), DoC allows
for lightweight end-to-end payload encryption based on OSCORE.

~~~ drawing

                - FETCH coaps://[2001:db8::1]/
               /
              /
             CoAP request
+--------+   [DNS query]   +--------+   DNS query    +--------+
|  DoC   |---------------->|  DoC   |...............>|  DNS   |
| Client |<----------------| Server |<...............| Server |
+--------+  CoAP response  +--------+  DNS response  +--------+
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

The terms "CoAP payload" and "CoAP body" are used as defined in [RFC7959].

{::boilerplate bcp14-tagged}

Selection of a DoC Server
=========================

It is assumed in this document that the DoC server, as well as the DNS query resource at the DoC
server, is known to the DoC client. Possible ways to configure could be manual configuration with a
URI {{?RFC3986}} or CRI {{?I-D.ietf-core-href}} or via a CoRE resource directory
{{?I-D.ietf-core-resource-directory}}. Automatic configuration SHOULD only be done from a trusted
source.

When discovering the URI through a link mechanism such as where the {{?RFC6690}} attribute "resource type" is available, the resource type "core.dns" can be used to identify a generic DNS resolver that is available to the client.

Basic Message Exchange
======================

The "application/dns-message" Content-Format    {#sec:content-format}
--------------------------------------------
This document defines the Internet media type "application/dns-message" for
the CoAP Content-Format. This media type is defined as in {{?RFC8484}}
Section 6, i.e., a single DNS message encoded in the DNS on-the-wire format
{{!RFC1035}}.

DNS Queries in CoAP Requests
----------------------------

A DoC client encodes a single DNS query in one or more CoAP request messages the CoAP FETCH {{!RFC8132}} method.
Requests SHOULD include an Accept option to indicate the type of content that can be parsed in the response.

To enable reliable message exchange, the CoAP request SHOULD be carried in a Confirmable (CON) message.

### Request Format

When sending a CoAP request, a DoC client MUST include the DNS query in the body (i.e. the payload, or the concatenated payloads) of the CoAP request.
As specified in {{!RFC8132}} Section 2.3.1, the type of content of the body MUST be indicated using the Content-Format option.
This document specifies the usage of Content-Format "application/dns-message" (details see {{sec:content-format}}).

If block-wise transfer {{!RFC7959}} is supported by the client, more than one CoAP request message MAY be used.
If more than one CoAP request message is used to encode the DNS query, it
must be chained together using the Block1 option in those CoAP requests.

The FETCH request is sent to the URI specified in {{selection-of-a-doc-server}}.

A DoC server MUST be able to parse requests of Content-Format
"application/dns-message".

### Support of CoAP Caching {#sec:req-caching}

The DoC client SHOULD set the ID field of the DNS header always to 0 to enable a CoAP cache (e.g., a CoAP proxy en-route) to respond to the same DNS queries with a cache entry.
This ensures that the CoAP Cache-Key (see {{!RFC8132}} Section 2) does not change when multiple DNS queries for the same DNS data, carried in CoAP requests, are issued.

### Examples

The following example illustrates the usage of a CoAP message to
resolve "example.org. IN AAAA" based on the URI "coaps://\[2001:db8::1\]/". The
CoAP body is encoded in "application/dns-message" Content Format.

    FETCH coaps://[2001:db8::1]/
    Content-Format: application/dns-message
    Accept: application/dns-message
    Payload: 00 00 01 20 00 02 00 00 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             01 00 01                                        [binary]


DNS Responses in CoAP Responses
-------------------------------

Each DNS query-response pair is mapped to a CoAP REST request-response
operation, which may consist of several CoAP request-response pairs if
block-wise transfer is involved.  DNS responses are provided in the body (i.e. the
payload, or the concatenated payloads) of the CoAP response. A DoC server MUST
indicate the type of content of the body using the Content-Format option,
and MUST be able to produce responses in the "application/dns-message"
Content-Format (details see {{sec:content-format}}) when requested.
A DoC client MUST understand responses in "application/dns-message" format
when it does not send an Accept option.

If supported, a DoC server MAY transfer the DNS response in more than one
CoAP responses using the Block2 option {{!RFC7959}}.

### Response Codes and Handling DNS and CoAP errors

A DNS response indicates either success or failure in the Response code of
the DNS header (see {{!RFC1035}} Section 4.1.1). It is RECOMMENDED that
CoAP responses that carry any valid DNS response use a "2.05 Content"
response code.

CoAP responses use non-successful response codes MUST NOT contain any payload
and may only be used on errors in the CoAP layer or when a request does not
fulfill the requirements of the DoC protocol.

Communication errors with a DNS server (e.g., timeouts) SHOULD be indicated
by including a SERVFAIL DNS response in a successful CoAP response.

A DoC client might try to repeat a non-successful exchange unless otherwise prohibited.
The DoC client might also decide to repeat a non-successful exchange with a different URI, for instance, when the response indicates an unsupported Content-Format.

### Support of CoAP Caching {#sec:resp-caching}

It is RECOMMENDED to set the Max-Age option of a response to the minimum TTL in the Answer section of a DNS response. This prevents expired records unintentionally being served from a CoAP cache.

It is RECOMMENDED that DoC servers set an ETag option on large responses (TBD: more concrete guidance) that have a short Max-Age relative to the expected clients' caching time.
Thus, clients that need to revalidate a response can do so using the established ETag mechanism.
<!--
With short responses, a usable ETag might be almost as long as the response.
With long-lived responses, the client does not need to revalidate often.
-->
With responses large enough to be fragmented,
it's best practice for servers to set an ETag anyway.
As specified in {{!RFC7252}} and {{!RFC8132}}, if the response associated with
the ETag is still valid, the response uses the "2.03 Valid" code and consequently
carries no payload.

### Examples

The following examples illustrate the replies to the query "example.org. IN
AAAA record", recursion turned on. Successful responses carry one answer
record including address 2001:db8:1::1:2:3:4 and TTL 58719.

A successful response:

    2.05 Content
    Content-Format: application/dns-message
    Max-Age: 58719
    Payload: 00 00 81 a0 00 01 00 01 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01 c0 0c 00 [binary]
             1c 00 01 00 01 37 49 00 10 20 01 0d b8 00 01 00 [binary]
             00 00 01 00 02 00 03 00 04                      [binary]

When a DNS error (SERVFAIL in this case) is noted in the DNS response, the CoAP
response still indicates success:

    2.05 Content
    Content-Format: application/dns-message
    Payload: 00 00 81 a2 00 01 00 00 00 00 00 00 07 65 78 61 [binary]
             6d 70 6c 65 03 6f 72 67 00 00 1c 00 01          [binary]

When an error occurs on the CoAP layer, the DoC server SHOULD respond with
an appropriate CoAP error, for instance "4.15 Unsupported Content-Format"
if the Content-Format option in the request was not set to
"application/dns-message" and the Content-Format is not otherwise supported by
the server.

CoAP/CoRE Integration
=====================

Proxies and caching
-------------------

TBD:

- [TTL vs. Max-Age](https://github.com/anr-bmbf-pivot/draft-dns-over-coap/issues/5)
- Responses that are not globally valid
- General CoAP proxy problem, but what to do when DoC server is a DNS proxy,
  response came not yet in but retransmission by DoC client was received (see
  {{rt-problem}})
    - send empty ACK ([maybe move to best practices appendix](https://github.com/anr-bmbf-pivot/draft-dns-over-coap/issues/6#issuecomment-895880206))

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

New "application/dns-message" Content-Format
--------------------------------------------

IANA is requested to assign CoAP Content-Format ID for the DNS message media
type in the "CoAP Content-Formats" sub-registry, within the "CoRE Parameters"
registry {{!RFC7252}}, corresponding the "application/dns-message" media
type from the "Media Types" registry:

Media-Type: application/dns-message

Encoding: -

Id: TBD

Reference: \[TBD-this-spec\]

New "core.dns" Resource Type
----------------------------

IANA is requested to assign an new Resource Type (rt=) Link Target Attribute, "core.dns" in the
"Resource Type (rt=) Link Target Attribute Values" sub-registry, within the "CoRE Parameters"
register {{?RFC6690}}.

Attribute Value: core.dns

Description: DNS over CoAP resource.

Reference: [TBD-this-spec] {{selection-of-a-doc-server}}


--- back

Change Log
==========

TBD:

- [Request text duplication](https://github.com/anr-bmbf-pivot/draft-dns-over-coap/issues/4)

Since [draft-lenders-dns-over-coap-01](https://datatracker.ietf.org/doc/html/draft-lenders-dns-over-coap-01)
---------------------------------------

- Remove GET and POST methods
- Add note on ETag and response codes
- Provide requirement conflict for DNS over QUIC
- Clarify Content-Format / Accept handling

Since [draft-lenders-dns-over-coap-00](https://datatracker.ietf.org/doc/html/draft-lenders-dns-over-coap-00)
---------------------------------------

- Soften Content-Format requirements in {{request-format}} and {{dns-responses-in-coap-responses}}
- Clarify "CoAP payload"/"CoAP body" terminology
- Fix nits and typos

Since [draft-lenders-dns-over-coaps-00](https://datatracker.ietf.org/doc/html/draft-lenders-dns-over-coaps-00)
---------------------------------------

- Clarification in abstract that both DTLS and OSCORE can be used as secure transport
- Restructuring of {{basic-message-exchange}}:
    - Add dedicated {{sec:content-format}} on Content-Format
    - Add overview table about usable and required features for request method
      types to {{dns-queries-in-coap-requests}}
    - Add dedicated {{sec:req-caching}} and {{sec:resp-caching}} on caching
      requirements for CoAP requests and responses
- Fix nits and typos

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
