---
v: 3

title: "DNS over CoAP (DoC)"
abbrev: DoC
docname: draft-ietf-core-dns-over-coap-latest
category: std
submissiontype: IETF

v3xml2rfc:
  silence:
  - Found SVG with width or height specified

area: Applications
workgroup: CoRE
keyword:
  - Internet-Draft
  - CoRE
  - CoAP
  - DNS
venue:
    group: CoRE
    type: Working Group
    mail: core@ietf.org
    arch: "https://mailarchive.ietf.org/arch/browse/core/"
    github: "core-wg/draft-dns-over-coap"
    latest: "https://core-wg.github.io/draft-dns-over-coap/draft-ietf-core-dns-over-coap.html"

author:
 -  name: Martine Sophie Lenders
    org: TUD Dresden University of Technology
    abbrev: TU Dresden
    street: Helmholtzstr. 10
    city: Dresden
    code: D-01069
    country: Germany
    email: martine.lenders@tu-dresden.de
 -  name: Christian Amsüss
    email: christian@amsuess.com
 -  name: Cenk Gündoğan
    org: NeuralAgent GmbH
    street: Mies-van-der-Rohe-Straße 6
    city: Munich
    code: D-80807
    country: Germany
    email: cenk.gundogan@neuralagent.ai
 -  name: Thomas C. Schmidt
    org: HAW Hamburg
    street: Berliner Tor 7
    city: Hamburg
    code: D-20099
    country: Germany
    email: t.schmidt@haw-hamburg.de
 -  name: Matthias Wählisch
    org: TUD Dresden University of Technology & Barkhausen Institut
    abbrev: TU Dresden & Barkhausen Institut
    street: Helmholtzstr. 10
    city: Dresden
    code: D-01069
    country: Germany
    email: m.waehlisch@tu-dresden.de

normative:
  RFC1035: dns
  RFC6347: dtls12
  RFC7228: constr-nodes
  RFC7252: coap
  RFC7641: coap-observe
  RFC7959: coap-blockwise
  RFC8132: coap-fetch
  RFC8613: oscore
  RFC8742: cborseq
  RFC8949: cbor
  RFC9147: dtls13
  I-D.ietf-core-coap-dtls-alpn: coap-dtls-alpn
  I-D.ietf-cbor-edn-literals: edn

informative:
  RFC3986: uri
  RFC6690: core-link-format
  RFC8765: dns-push
  RFC8094: dodtls
  RFC8484: doh
  RFC9176: core-rd
  RFC9250: doq
  RFC9364: dnssec
  RFC8499: dns-terminology
  RFC9460: svcb
  RFC9461: svcb-dns
  RFC9462: ddr
  RFC9463: dnr
  I-D.ietf-core-href: cri
  I-D.ietf-core-transport-indication: transport-indication
  I-D.lenders-core-dnr: core-dnr
  I-D.amsuess-core-cachable-oscore: cachable-oscore
  DoC-paper: DOI.10.1145/3609423
  I-D.ietf-core-corr-clar: core-corrclar
--- abstract

This document defines a protocol for sending DNS messages over the
Constrained Application Protocol (CoAP). These CoAP messages can be protected
by DTLS-Secured CoAP (CoAPS) or Object Security for Constrained RESTful
Environments (OSCORE) to provide encrypted DNS message exchange for
constrained devices in the Internet of Things (IoT).

--- middle

Introduction
============

This document defines DNS over CoAP (DoC), a protocol to send DNS
{{-dns}} queries and get DNS responses over the Constrained Application
Protocol (CoAP) {{-coap}}. Each DNS query-response pair is mapped into a
CoAP message exchange. Each CoAP message can be secured by DTLS {{-dtls12}} {{-dtls13}} or
Object Security for Constrained RESTful Environments (OSCORE) {{-oscore}}
to ensure message integrity and confidentiality.

The application use case of DoC is inspired by DNS over HTTPS {{-doh}}
(DoH). DoC, however, aims for the deployment in the constrained Internet of
Things (IoT), which usually conflicts with the requirements introduced by
HTTPS.
Constrained IoT devices may be restricted in memory, power consumption,
link layer frame sizes, throughput, and latency. They may
only have a handful kilobytes of both RAM and ROM. They may sleep for long
durations of time, after which they need to refresh the named resources they
know about. Name resolution in such scenarios must take into account link
layer frame sizes of only a few hundred bytes, bit rates in the magnitude
of kilobits per second, and latencies of several seconds {{-constr-nodes}}.

In order not to be burdened by the resource requirements of TCP and HTTPS, constrained IoT devices could use DNS over DTLS {{-dodtls}}.
In contrast to DNS over DTLS, DoC
utilizes CoAP features to mitigate drawbacks of datagram-based
communication. These features include: block-wise transfer, which solves
the Path MTU problem of DNS over DTLS (see {{-dodtls, Section 5}}); CoAP
proxies, which provide an additional level of caching; re-use of data
structures for application traffic and DNS information, which saves memory
on constrained devices.

To avoid resource requirements of DTLS or TLS on top of UDP (e.g., introduced by DNS over DTLS {{-dodtls}} or DNS over QUIC {{-doq}}), DoC allows for lightweight payload encryption based on OSCORE.

~~~ aasvg

              . FETCH coaps://[2001:db8::1]/
             /
            /
           CoAP request
+------+   [DNS query]   +------+   DNS query     .---------------.
| DoC  |---------------->| DoC  |--- --- --- --->|      DNS        |
|Client|<----------------|Server|<--- --- --- ---| Infrastructure  |
+------+  CoAP response  +------+  DNS response   '---------------'
          [DNS response]
   \                        /\                                 /
    '-----DNS over CoAP----'  '--DNS over UDP/HTTPS/QUIC/...--'

~~~
{: #fig-overview-arch title="Basic DoC architecture"}


The most important components of DoC can be seen in {{fig-overview-arch}}: A DoC
client tries to resolve DNS information by sending DNS queries carried within
CoAP requests to a DoC server.
That DoC server is a DNS client (i.e., a stub or recursive resolver) that resolves DNS information by using other DNS transports such
as DNS over UDP {{-dns}}, DNS over HTTPS {{-doh}}, or DNS over QUIC {{-doq}} when communicating with the upstream
DNS infrastructure.
Using that information, the DoC server then replies to the queries of the DoC client with DNS
responses carried within CoAP responses.

Note that this specification is distinct from DoH since the CoRE-specific FETCH method is used.
This was done to take benefit from having the DNS query in the payload as with POST, but still
having the caching advantages we would gain with GET.
Having the DNS query in the payload means we do not need extra base64 encoding, which would increase
code complexity and message sizes.
We are also able to transfer a query block-wise.


Terminology
===========

A server that provides the service specified in this document is called a "DoC
server" to differentiate it from a classic "DNS server".
A DoC server acts either as a DNS stub resolver {{-dns-terminology}} or a DNS recursive resolver {{-dns-terminology}}.
As such, the DoC server communicates with an "upstream DNS infrastructure" or a single "upstream DNS server".

A client using the service specified in this document to retrieve the
DNS information is called a "DoC client".

The term "constrained nodes" is used as defined in {{-constr-nodes}}.

The terms "CoAP payload" and "CoAP body" are used as defined in {{-coap-blockwise, Section 2}}.

{::boilerplate bcp14-tagged}

Selection of a DoC Server   {#sec:doc-server-selection}
=========================

While there is no path specified for the DoC resource, it is RECOMMENDED to use the root path "/"
to keep the CoAP requests small.

In this document, it is assumed that the DoC client knows the DoC server and the DNS resource at the
DoC server.
Possible options could be manual configuration of a URI {{-uri}} or CRI {{-cri}},
or automatic configuration, e.g., using a CoRE resource directory
{{-core-rd}}, DHCP or Router Advertisement options {{-dnr}} or discovery of designated resolvers
{{-ddr}}.
Automatic configuration SHOULD only be done from a trusted source.

## Discovery by Resource Type
When discovering the DNS resource through a link mechanism that allows describing a resource type
(e.g., the Resource Type Attribute in {{-core-link-format}}), the resource type "core.dns" can be
used to identify a generic DNS resolver that is available to the client.

## Discovery using SVCB Resource Records or DNR
A DoC server can also be discovered using SVCB Resource Records (RR) {{-svcb}}, {{-svcb-dns}} or DNR
Service Parameters {{-dnr}}.
{{-coap-dtls-alpn}} provides solutions to discover CoAP over (D)TLS servers using the "alpn" SvcParam.
{{-core-dnr}} provides a problem statement for service bindings discovery for OSCORE and EDHOC.
This document specifies "docpath" as
a single-valued SvcParamKey whose value MUST be a CBOR sequence of 0 or more text strings (see
{{-cborseq}} and {{-cbor}}), delimited by the length of the SvcParamValue field (in octets). If the
SvcParamValue ends within a CBOR text string, the SVCB RR MUST be considered as malformed.
As a text format, e.g., in DNS zone files, the CBOR diagnostic
notation (see {{Section 8 of -cbor}} and {{-edn}})
of that CBOR sequence can be used.

Note, that this specifically does not surround the text string sequence with a CBOR array or a
similar CBOR data item. This path format was chosen to coincide with the path representation in CRIs
({{-cri}}). Furthermore, it is easily transferable into a sequence of CoAP Uri-Path options by
mapping the initial byte of any present CBOR text string (see {{-cbor, Section 3}}) into the Option
Delta and Option Length of the CoAP option, provided these CBOR text strings are all of a length
between 0 and 12 octets (see {{-coap, Section 3.1}}). Likewise, it can be transferred into a URI
path-abempty form (see {{-uri, Section 3.3}}) by replacing the initial byte of any present CBOR text
string with the "/" character, provided these CBOR text strings are all of a length less than 24
octets and do not contain bytes that need escaping.

To use the service binding from a SVCB RR, the DoC client MUST send a DoC request constructed from the SvcParams including "docpath".
A rough construction algorithm could be as follows, going through the provided records in order of their priority.

- If the "alpn" SvcParam value for the service is "coap", construct a CoAP request for CoAP over TLS,
  if it is "co", construct a CoAP request for CoAP over DTLS. Any other SvcParamKeys specifying a
  CoAP transport are out of scope of this document.
- The destination address for the request should be taken from additional information about the
  target, e.g., from an AAAA record associated to the target name or from an "ipv6hint" SvcParam
  value, or, as a fallback, by querying an address for the target name of the SVCB record.
- The destination port for the address is taken from the "port" SvcParam value, if present.
  Otherwise, take the default port of the CoAP transport.
- Set the target name of SVCB record in the URI-Host option.
- For each element in the CBOR sequence of the "docpath" SvcParam value, add a Uri-Path option to
  the request.
- If a "port" SvcParam value is provided or if a port was queried, and if either differs from
  the default port of the transport or the destination port selected above, set that port in the
  URI-Port option.
- If the request constructed this way receives a response, use the same SVCB record for construction
  of future DoC queries.
  If not, or if the endpoint becomes unreachable, repeat with the SVCB record with the next highest
  priority.

A more generalized construction algorithm can be found in {{-transport-indication}}.


Basic Message Exchange
======================

The "application/dns-message" Content-Format    {#sec:content-format}
--------------------------------------------
This document defines a CoAP Content-Format number for the Internet
media type "application/dns-message" to be the mnemonic 553 — based on the port assignment of DNS.
This media type is defined as in {{Section 6 of -doh}}, i.e., a single DNS message encoded in the DNS on-the-wire format {{-dns}}.
Both DoC client and DoC server MUST be able to parse contents in the "application/dns-message" format.
For the purposes of this document, only OPCODE 0 (Query) is supported for DNS messages.
Future work might provide specifications and considerations for other values of OPCODE.
Unless another error takes precedence, a DoC server uses RCODE = 4, NotImp {{-dns}}, in its response when it receives a query with an OPCODE it does not implement (see also {{sec:resp-examples}}).

DNS Queries in CoAP Requests    {#sec:queries}
----------------------------

A DoC client encodes a single DNS query in one or more CoAP request
messages that use the CoAP FETCH {{-coap-fetch}} method.
Requests SHOULD include an Accept option to indicate the type of content that can be parsed in the response.

Since CoAP provides reliability of the message layer (e.g., CON) the retransmission mechanism of the
DNS protocol as defined in {{-dns}} is not needed.

### Request Format

When sending a CoAP request, a DoC client MUST include the DNS query in the body of the CoAP request.
As specified in {{Section 2.3.1 of -coap-fetch}}, the type of content of the body MUST be indicated using the Content-Format option.
This document specifies the usage of Content-Format "application/dns-message" (details see {{sec:content-format}}).
A DoC server MUST be able to parse requests of Content-Format "application/dns-message".

### Support of CoAP Caching {#sec:req-caching}

The DoC client SHOULD set the ID field of the DNS header always to 0 to enable a CoAP cache (e.g., a CoAP proxy en-route) to respond to the same DNS queries with a cache entry.
This ensures that the CoAP Cache-Key (see {{-coap-fetch, Section 2}}) does not change when multiple DNS queries for the same DNS data, carried in CoAP requests, are issued.

### Examples {#sec:req-examples}

The following example illustrates the usage of a CoAP message to
resolve "example.org. IN AAAA" based on the URI "coaps://\[2001:db8::1\]/". The
CoAP body is encoded in "application/dns-message" Content Format.
For better readability, we provide the payload in a human-readable format.
In the actual message, however, it would be encoded in the binary message format defined in {{-dns}}.

    FETCH coaps://[2001:db8::1]/
    Content-Format: application/dns-message
    Accept: application/dns-message
    Payload (human-readable):
      ;; ->>Header<<- opcode: QUERY, status: NOERROR, id: 0
      ;; flags: rd ad; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ARCOUNT: 0

      ;; QUESTION SECTION:
      ;example.org.             IN      AAAA

As such, the message would actually look as follows (bytes printed in hexadecimal representation).

    FETCH coaps://[2001:db8::1]/
    Content-Format: application/dns-message
    Accept: application/dns-message
    Payload (binary):
      00 00 01 20 00 01 00 00 00 00 00 00 07 65 78 61
      6d 70 6c 65 03 6f 72 67 00 00 1c 00 01


DNS Responses in CoAP Responses
-------------------------------

Each DNS query-response pair is mapped to a CoAP request-response
operation. DNS responses are provided in the body of the CoAP response.
A DoC server MUST be able to produce responses in the "application/dns-message"
Content-Format (details see {{sec:content-format}}) when requested.
A DoC client MUST understand responses in "application/dns-message" format
when it does not send an Accept option.
Any other response format than "application/dns-message" MUST be indicated with
the Content-Format option by the DoC server.

### Response Codes and Handling DNS and CoAP errors

A DNS response indicates either success or failure in the RCODE of the DNS header (see {{Section 4.1.1 of -dns}}).
It is RECOMMENDED that CoAP responses that carry a parseable DNS response use a "2.05 Content" response code.

CoAP responses using non-successful response codes MUST NOT contain a DNS response
and MUST only be used on errors in the CoAP layer or when a request does not
fulfill the requirements of the DoC protocol.

Communication errors with an upstream DNS server (e.g., timeouts) MUST be indicated by including a DNS response with the appropriate RCODE in a successful CoAP response, i.e., using a 2.xx response code.

A DoC client might try to repeat a non-successful exchange unless otherwise prohibited.
The DoC client might also decide to repeat a non-successful exchange with a different URI, for instance, when the response indicates an unsupported Content-Format.

### Support of CoAP Caching {#sec:resp-caching}

For reliability and energy saving measures, content decoupling, i.e., en-route caching on proxies, takes a far greater role than it does, e.g., in HTTP.
Likewise, CoAP utilizes cache validation to refresh stale cache entries to reduce the amount of large response messages.
For cache validation, CoAP implementations regularly use hashing over the message content for ETag generation.
As such, the approach to guarantee the same cache key for DNS responses as proposed in DoH ({{-doh, Section 5.1}}) is not sufficient and needs to be updated so that the TTLs in the response are more often the same regardless of query time.

The DoC server MUST ensure that any sum of the Max-Age value of a CoAP response and any TTL in the
DNS response is less or equal to the corresponding TTL received from an upstream DNS server.
This also includes the default Max-Age value of 60 seconds (see {{Section 5.10.5 of -coap}}) when no Max-Age option is provided.
The DoC client MUST then add the Max-Age value of the carrying CoAP response to all TTLs in a DNS response on reception and use these calculated TTLs for the associated records.

The RECOMMENDED algorithm to meet the requirement for DoC is as follows:
Set the Max-Age option of a response to the minimum TTL of a DNS response and subtract this value from all TTLs of that DNS response.
This prevents expired records unintentionally being served from an intermediate CoAP cache.
Additionally, it allows for the ETag value for cache validation, if it is based on the content of the response, not to change even if the TTL values are updated by an upstream DNS cache.
If only one record set per DNS response is assumed, a simplification of this algorithm is to just set all TTLs in the response to 0 and set the TTLs at the DoC client to the value of the Max-Age option.

If shorter caching periods are plausible, e.g., if the RCODE of the message indicates an error that should only be cached for a minimal duration, the value for the Max-Age option SHOULD be set accordingly.
This value might be 0, but if the DoC server knows the error will persist, greater values are also conceivable, depending on the projected duration of the error.
Same goes for DNS responses, that for any reason do not carry any records with a TTL.

### Examples {#sec:resp-examples}

The following example illustrates the response to the query "example.org. IN
AAAA record", recursion turned on. Successful responses carry one answer
record including address 2001:db8:1::1:2:3:4 and TTL 79689.
As in {{sec:req-examples}}, we use a human-readable format for the payload, but skip the binary conversion this time.

A successful response:

    2.05 Content
    Content-Format: application/dns-message
    Max-Age: 58719
    Payload (human-readable):
      ;; ->>Header<<- opcode: QUERY, status: NOERROR, id: 0
      ;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ARCOUNT: 0

      ;; QUESTION SECTION:
      ;example.org.                 IN      AAAA
      ;; ANSWER SECTION:
      ;example.org.         79689   IN      AAAA    2001:db8:1::1:2:3:4

When a DNS error—NotImp (RCODE = 4) in response to a DNS Update (OPCODE = 5) for "example.org" in this case—is noted in the DNS response, the CoAP response still indicates success.

    2.05 Content
    Content-Format: application/dns-message
    Payload (human-readable):
      ;; ->>Header<<- opcode: UPDATE, status: NOTIMP, id: 0
      ;; flags: qr ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ARCOUNT: 0

      ;; ZONE SECTION:
      ;example.org.                 IN      SOA

When an error occurs on the CoAP layer, the DoC server SHOULD respond with
an appropriate CoAP error, for instance "4.15 Unsupported Content-Format"
if the Content-Format option in the request was not set to
"application/dns-message" and the Content-Format is not otherwise supported by
the server.

    4.15 Unsupported Content-Format
    [no payload]

CoAP/CoRE Integration
=====================

DNS Push
--------
DNS Push requires additional overhead, which conflicts with constrained resources,
This is the reason why it is RECOMMENDED to use CoAP Observe {{-coap-observe}} instead of DNS Push
in the DoC domain.

If the CoAP request indicates that the DoC client wants to observe a resource record, a DoC server
MAY use a DNS Subscribe message {{-dns-push}} instead of a classic DNS query to fetch the
information on behalf of a DoC client.
If this is not supported by the DoC server, it MUST act as if the resource were not observable.

Whenever the DoC server receives a DNS Push message {{-dns-push}} from the DNS
infrastructure for an observed resource record, the DoC server sends an appropriate Observe response
to the DoC client.

If no more DoC clients observe a resource record for which the DoC server has an open subscription,
the DoC server MUST use a DNS Unsubscribe message {{-dns-push}} to close its subscription to the
resource record as well.

OSCORE
------
It is RECOMMENDED to carry DNS messages encrypted using OSCORE {{-oscore}} between the DoC client
and the DoC server. The establishment and maintenance of the OSCORE Security Context is out of the
scope of this document.

If cache retrieval of OSCORE responses is desired, it can be achieved, for instance, by using the
method defined in {{-cachable-oscore}}. This has, however, implications on message sizes and
security properties, which are compiled in that document.

Mapping DoC to DoH
------------------
This document provides no specification how to map between DoC and DoH, e.g., at a CoAP-HTTP-proxy.
In fact, such a direct mapping is NOT RECOMMENDED:
Rewriting the FETCH method ({{sec:queries}}) and the TTL rewriting ({{sec:resp-caching}}) as
specified in this draft would be non-trivial.
It is RECOMMENDED to use a DNS forwarder to map between DoC and DoH, as would be the case for
mapping between any other pair of DNS transports.

Considerations for Unprotected Use {#sec:unprotected-coap}
==================================
The use of DoC without confidentiality protection is NOT RECOMMENDED.
Without confidential communication, many possible attacks need to be evaluated in the context of
the application's threat model.
This includes threats that are mitigated even by DNS over UDP:
For example, the random ID of the DNS header afford some protection against off-path cache poisoning
attacks—a threat that might be mitigated by using random large token values in the CoAP request.

Implementation Status
=====================

{::boilerplate rfc7942}

DoC Client
----------
The authors of this document provide a [DoC client implementation available
in the IoT operating system RIOT][gcoap_dns].

Level of maturity:
: production

Version compatibility:
: draft-ietf-core-dns-over-coap-09

License:
: LGPL-2.1

Contact information:
: `Martine S. Lenders <martine.lenders@tu-dresden.de>`

Last update of this information:
: September 2024

DoC Server
----------
The authors of this document provide a [DoC server implementation in
Python][aiodnsprox].

Level of maturity:
: production

Version compatibility:
: draft-ietf-core-dns-over-coap-09

License:
: MIT

Contact information:
: `Martine S. Lenders <martine.lenders@tu-dresden.de>`

Last update of this information:
: September 2024

Security Considerations
=======================

General CoAP security considerations in {{Section 11 of RFC7252}} apply to DoC.
Additionally, DoC uses request patterns that require the maintenance of long-lived security
contexts.
{{Section 2.6 of -core-corrclar}} goes into more detail on what needs to be done
when those are resumed from a new endpoint.

When using unprotected CoAP (see {{sec:unprotected-coap}}), setting the ID of a DNS message to 0 as
specified in {{sec:req-caching}} opens the DNS cache of a DoC client to cache poisoning attacks
via response spoofing.
This document requires an unpredictable CoAP token in each DoC query from the client when CoAP is
not secured to mitigate such an attack over DoC (see {{sec:unprotected-coap}}).

For confidential communication via DTLS or OSCORE the impact of a fixed ID on security is limited, as both
harden against injecting spoofed responses.
Consequently, it is of little concern to leverage the benefits of CoAP caching by setting the ID to
0.

A user of DoC must be aware that the DoC server
may communicate unprotected with the upstream DNS infrastructure, e.g., using DNS over UDP.
DoC can only guarantee confidential communication and integrity between parties for which the
security context is exchanged.
The DoC server may use another security context to communicate confidentially and with integrity
upstream (e.g., DNS over QUIC {{-doq}}) or just integrity (e.g., DNSSEC {{-dnssec}}), but, while
recommended, this is opaque to the DoC client on the protocol level.

A DoC client may not be able to perform DNSSEC validation,
e.g., due to code size constraints, or due to size of the responses.
It may trust its DoC server to perform DNSSEC validation;
how that trust is expressed is out of scope of this document.
A DoC client may be, for instance, configured to use a particular credential by which it recognizes an eligible DoC server.
That information can also imply trust in the DNSSEC validation by that server.

IANA Considerations
===================

[^replace-xxxx]

[^replace-xxxx]: RFC Ed.: throughout this section, please replace
    RFC-XXXX with the RFC number of this specification and remove this
    note.

New "application/dns-message" Content-Format
--------------------------------------------

IANA is requested to assign a CoAP Content-Format ID for the DNS message media
type in the "CoAP Content-Formats" sub-registry, within the "CoRE Parameters"
registry {{-coap}}, corresponding to the "application/dns-message" media
type from the "Media Types" registry (see {{-doh}}).

Content Type: application/dns-message

Content Coding: -

Id: 553 (suggested)

Reference: {{-doh}}\[RFC-XXXX, {{sec:content-format}}\]

New "docpath" SVCB Service Parameter
------------------------------------

This document adds the following entry to the Service Parameter Keys (SvcParamKeys) registry in the
DNS Service Bindings (SVCB) registry group.
The definition of this parameter can be found in {{sec:doc-server-selection}}.

| Number  | Name           | Meaning                            | Reference       |
| ------- | -------------- | ---------------------------------- | --------------- |
| 10 (suggested)     | docpath        | DNS over CoAP resource path        | \[RFC-XXXX, {{sec:doc-server-selection}}\] |
{: #tab-svc-param-keys title="Values for SvcParamKeys"}

New "core.dns" Resource Type
----------------------------

IANA is requested to assign a new Resource Type (rt=) Link Target Attribute, "core.dns" in the
"Resource Type (rt=) Link Target Attribute Values" sub-registry, within the "CoRE Parameters"
register {{-core-link-format}}.

Attribute Value: core.dns

Description: DNS over CoAP resource.

Reference: \[RFC-XXXX, {{sec:doc-server-selection}}\]


--- back

Evaluation {#sec:evaluation}
==========
The authors of this document presented the design, implementation, and analysis of DoC in their
paper "Securing Name Resolution in the IoT: DNS over CoAP" {{DoC-paper}}.

Change Log
==========

[^remove-changelog]

[^remove-changelog]: RFC Ed.: Please remove this section before publication.

Since [draft-ietf-core-dns-over-coap-10]
----------------------------------------
- Replace imprecise or wrong terms:
    - disjunct => distinct
    - unencrypted CoAP => unprotected CoAP
    - security mode => confidential communication
- Pull in definition of CBOR sequences and their EDN
- Fix broken external section references
- Define terminology for "upstream DNS infrastructure" and "upstream DNS server"
- Fix wording on DNS error handling
- Clarify that any OpCode beyond 0 is not supported for now and remove now redundant DNS Upgrade
  section as a consequence
- Clarify that the DoC/DoH mapping is what is NOT RECOMMENDED
- Avoid use of undefined term “CoAP resource identifier”
- Discuss Max-Age option value in an error case
- Add human-readable format to examples
- General language check pass

Since [draft-ietf-core-dns-over-coap-09]
----------------------------------------
- Update SVCB SvcParamKey
- Update corr-clar reference
- Add reference to DNS Update {{?RFC2136}}, clarify that it is currently not considered
- Add to security considerations: unprotected upstream DNS and DNSSEC

Since [draft-ietf-core-dns-over-coap-08]
----------------------------------------
- Update Cenk's Affiliation

Since [draft-ietf-core-dns-over-coap-07]
----------------------------------------
- Address IANA early review #1368678
- Update normative reference to CoAP over DTLS alpn SvcParam
- Add missing DTLSv1.2 reference
- Security considerations: Point into corr-clar-future
- Implementation Status: Update to current version

Since [draft-ietf-core-dns-over-coap-06]
----------------------------------------
- Add "docpath" SVCB ParamKey definition
- IANA fixes
  + Use new column names (see Errata 4954)
  + Add reference to RFC 8484 for application/dns-message Media Type
  + IANA: unify self references

Since [draft-ietf-core-dns-over-coap-05]
----------------------------------------
- Add references to relevant SVCB/DNR RFCs and drafts

Since [draft-ietf-core-dns-over-coap-04]
----------------------------------------
- Add note on cacheable OSCORE
- Address early IANA review

Since [draft-ietf-core-dns-over-coap-03]
----------------------------------------
- Amended Introduction with short contextualization of constrained environments
- Add {{sec:evaluation}} on evaluation

Since [draft-ietf-core-dns-over-coap-02]
----------------------------------------

- Move implementation details to Implementation Status (in accordance with {{RFC7942}})
- Recommend root path to keep the CoAP options small
- Set Content-Format for application/dns-message to 553
- SVCB/DNR: Move to Server Selection Section but leave TBD based on DNSOP discussion for now
- Clarify that DoC and DoH are distinct
- Clarify mapping between DoC and DoH
- Update considerations on unprotected use
- Don't call OSCORE end-to-end encrypted


Since [draft-ietf-core-dns-over-coap-01]
----------------------------------------

- Specify DoC server role in terms of DNS terminology
- Clarify communication of DoC to DNS infrastructure is agnostic of the transport
- Add subsection on how to implement DNS Push in DoC
- Add appendix on reference implementation

Since [draft-ietf-core-dns-over-coap-00]
----------------------------------------

- SVGify ASCII art
- Move section on "DoC Server Considerations" (was Section 5.1) to its own draft
  ([draft-lenders-dns-cns])
- Replace layer violating statement for CON with statement of fact
- Add security considerations on ID=0

Since [draft-lenders-dns-over-coap-04]
--------------------------------------

- Removed change log of draft-lenders-dns-over-coap

# Acknowledgments
{:unnumbered}

The authors of this document want to thank Carsten Bormann, Ben Schwartz, Marco Tiloca, Tim
Wicinski, Thomas Fossati, and Esko Dijk for their feedback and comments.

[draft-ietf-core-dns-over-coap-10]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-10
[draft-ietf-core-dns-over-coap-09]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-09
[draft-ietf-core-dns-over-coap-08]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-08
[draft-ietf-core-dns-over-coap-07]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-07
[draft-ietf-core-dns-over-coap-06]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-06
[draft-ietf-core-dns-over-coap-05]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-05
[draft-ietf-core-dns-over-coap-04]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-04
[draft-ietf-core-dns-over-coap-03]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-03
[draft-ietf-core-dns-over-coap-02]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-02
[draft-ietf-core-dns-over-coap-01]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-01
[draft-ietf-core-dns-over-coap-00]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-00
[draft-lenders-dns-cns]: https://datatracker.ietf.org/doc/draft-lenders-dns-cns/
[draft-lenders-dns-over-coap-04]: https://datatracker.ietf.org/doc/html/draft-lenders-dns-over-coap-04
[gcoap_dns]: https://doc.riot-os.org/group__net__gcoap__dns.html
[aiodnsprox]: https://github.com/anr-bmbf-pivot/aiodnsprox
