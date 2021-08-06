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

- Request SHOULD be CON {{!RFC7252}} to reuse CoAP retransmissions
- DNS message ID SHOULD be 0 in query (cache key variance...)
- Allowed methods: GET, FETCH {{!RFC8132}} (“POST but cacheable”, blockwise
  transfer...), and POST

Queries
-------

### GET
- Encode query as `?dns=...` URI query in `base64url` {{!RFC4648}}
- MUST have Accept option (set to `application/dns-message` content-format ID)
- Expected successful response code: 2.05 Content
- Considerations: Cacheable, but no blockwise transfer (URI-Query option can
  become comparatively large), ...

### POST and FETCH
- Query carried in binary "wire" DNS message format {{!RFC1035}} in payload
- MUST have Content Format option (set to `application/dns-message`
  content-format ID)
- MUST have Accept option (set to `application/dns-message` content-format ID)
- Expected successful response code:
    - POST: 2.01 Created
    - FETCH: 2.05 Content

Responses
---------

- On success (see [](#queries) for response code):
    - DNS response carried as binary "wire" DNS message format {{!RFC1035}} in
      payload
    - MUST have Content Format option (set to `application/dns-message`)
    - MUST have Max-Age option (set to minimum TTL from DNS response)
- Only report CoAP layer errors with CoAP error messages.
  Examples:
  - MUSTs for DoC not full-filled by CoAP request: `4.00 Bad Request`
  - Content Format in Accept option can not be generated: `4.06 Not Acceptable`
  - Content Format of payload is not as expected: `4.15 Unsupported
    Content-Format`
- DNS layer errors MUST be responded with successful CoAP message (cmp.
  {{?RFC8484}}, section 4.2.1)
    - On communication error with upstream DNS server (e.g. timeout): respond
      to query with SERVFAIL message

CoAP/CoRE Integration
=====================

Proxies and caching
-------------------
- Cache SHOULD support FETCH ("the request body is part of the cache key"
  {{!RFC8132}})

OBSERVE (modifications)?
------------------------

OSCORE
------

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
registry {{!RFC7252}}, corresponding the `application/dns-message` media
type from the "Media Types" registry:

Media-Type: application/dns-message

Encoding: -

Id: TBD

Reference: [TBD-this-spec]

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
