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

- DoH ensures encrypted DNS {{?RFC8484}}
- DoH for IoT: CoAP + DTLS {{!RFC7252}} + DNS


Terminology
-----------

{::boilerplate bcp14-tagged}

CoAP Messaging
==============

- Request SHOULD be CON {{!RFC7252}} to reuse CoAP retransmissions
- Request are sent to a resource defined by a URI template {{!RFC6570}} (see
  [](#uri-template-alternatives) for potential alternatives)
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

URI template alternatives
-------------------------
- URI templates {{RFC6570}} have advantage that DoH config mechanisms can be
  reused (see also {#uri-template-configuration})
- CRI {{?I-D.ietf-core-href}} or CoRAL {{?I-D.ietf-core-coral}}

OBSERVE (modifications)?
------------------------

OSCORE
------

URI template configuration
==========================
- Maybe out-of-scope?
- DHCP and RA options to deliver? {{?I-D.peterson-doh-dhcp}}
- CoRE-RD {{?I-D.ietf-core-resource-directory-28}}...

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
