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
  RFC3986: uri
  RFC5234: abnf
  RFC6347: dtls12
  RFC7252: coap
  RFC7641: coap-observe
  RFC7959: coap-blockwise
  RFC8132: coap-fetch
  RFC8323: coap-tcp
  RFC8484: doh
  RFC8613: oscore
  RFC8765: dns-push
  RFC8949: cbor
  RFC9147: dtls13
  I-D.ietf-core-coap-dtls-alpn: coap-dtls-alpn

informative:
  BCP219: dns-terminology
  RFC3833: dns-threats
  RFC6690: core-link-format
  RFC7228: constr-nodes
  RFC8094: dodtls
  RFC9076: dns-privacy
  RFC9176: core-rd
  RFC9250: doq
  RFC9364: dnssec
  RFC9460: svcb
  RFC9461: svcb-dns
  RFC9462: ddr
  RFC9463: dnr
  RFC9528: edhoc
  I-D.ietf-core-href: cri
  I-D.ietf-core-transport-indication: transport-indication
  I-D.ietf-iotops-7228bis: constr-nodes-bis
  I-D.lenders-core-dnr: core-dnr
  I-D.amsuess-core-cachable-oscore: cachable-oscore
  DoC-paper: DOI.10.1145/3609423
  I-D.ietf-core-corr-clar: core-corrclar
  REST:
    target: https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
    title: Architectural Styles and the Design of Network-based Software Architectures
    author:
      ins: R. Fielding
      name: Roy Thomas Fielding
      org: University of California, Irvine
    date: 2000
    seriesinfo:
      "Ph.D.": "Dissertation, University of California, Irvine"
    format:
      HTML: https://ics.uci.edu/~fielding/pubs/dissertation/top.htm
      PDF: https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
--- abstract

This document defines a protocol for exchanging DNS queries (OPCODE 0) over the
Constrained Application Protocol (CoAP). These CoAP messages can be protected
by (D)TLS-Secured CoAP (CoAPS) or Object Security for Constrained RESTful
Environments (OSCORE) to provide encrypted DNS message exchange for
constrained devices in the Internet of Things (IoT).

--- middle

Introduction
============

This document defines DNS over CoAP (DoC), a protocol to send DNS
{{-dns}} queries and get DNS responses over the Constrained Application
Protocol (CoAP) {{-coap}} using OPCODE 0 (Query). Each DNS query-response pair is mapped into a
CoAP message exchange. Each CoAP message can be secured by DTLS {{-dtls12}} {{-dtls13}} or
Object Security for Constrained RESTful Environments (OSCORE) {{-oscore}}
but also TLS {{-coap-tcp}} {{?RFC8446}}
to ensure message integrity and confidentiality.

The application use case of DoC is inspired by DNS over HTTPS {{-doh}}
(DoH). DoC, however, aims for deployment in the constrained Internet of
Things (IoT), which usually conflicts with the requirements introduced by
HTTPS.
Constrained IoT devices may be restricted in memory, power consumption,
link layer frame sizes, throughput, and latency. They may
only have a handful kilobytes of both RAM and ROM. They may sleep for long
durations of time, after which they need to refresh the named resources they
know about. Name resolution in such scenarios must take into account link
layer frame sizes of only a few hundred bytes, bit rates in the magnitude
of kilobits per second, and latencies of several seconds {{-constr-nodes}} {{-constr-nodes-bis}} [^remove-constr-nodes].

[^remove-constr-nodes]: RFC Ed.: Please remove the {{-constr-nodes}} reference and replace it with {{-constr-nodes-bis}} throughout the document in case {{-constr-nodes-bis}} becomes an RFC before publication.

In order not to be burdened by the resource requirements of TCP and HTTPS, constrained IoT devices could use DNS over DTLS {{-dodtls}}.
In contrast to DNS over DTLS, DoC
can take advantage of CoAP features to mitigate drawbacks of datagram-based
communication. These features include: block-wise transfer {{-coap-blockwise}}, which solves
the Path MTU problem of DNS over DTLS (see {{-dodtls, Section 5}}); CoAP
proxies, which provide an additional level of caching; re-use of data
structures for application traffic and DNS information, which saves memory
on constrained devices.

To avoid the resource requirements of DTLS or TLS on top of UDP (e.g., introduced by DNS over DTLS {{-dodtls}} or DNS over QUIC {{-doq}}), DoC allows for lightweight message protection based on OSCORE.

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


The most important components of DoC can be seen in {{fig-overview-arch}}: a DoC
client tries to resolve DNS information by sending DNS queries carried within
CoAP requests to a DoC server.
That DoC server is a DNS client (i.e., a stub or recursive resolver) that resolves DNS information by using other DNS transports such
as DNS over UDP {{-dns}}, DNS over HTTPS {{-doh}}, or DNS over QUIC {{-doq}} when communicating with the upstream
DNS infrastructure.
Using that information, the DoC server then replies to the queries of the DoC client with DNS
responses carried within CoAP responses.

Note that this specification is distinct from DoH, since the CoAP-specific FETCH method {{-coap-fetch}} is used.
This has the benefit of having the DNS query in the body as when using the POST method, but still with the same caching advantages of responses to requests that use the GET method.
Having the DNS query in the body means that we do not need extra base64 encoding, which would increase
code complexity and message sizes.
Also, this allows for the block-wise transfer of queries {{-coap-blockwise}}.


Terminology and Conventions
===========================

{::boilerplate bcp14-tagged}

A server that provides the service specified in this document is called a "DoC
server" to differentiate it from a classic "DNS server".
A DoC server acts either as a DNS stub resolver or a DNS recursive resolver {{-dns-terminology}}.
As such, the DoC server communicates with an "upstream DNS infrastructure" or a single "upstream DNS server".
A "DoC resource" is a CoAP resource {{-coap}} at the DoC server that DoC clients can target to send a DNS query in a CoAP request.

A client using the service specified in this document to retrieve the
DNS information is called a "DoC client".

The term "constrained nodes" is used as defined in {{-constr-nodes}}.
{{-core-link-format}} describes that "Constrained RESTful Environments (CoRE)" realize the Representational State Transfer (REST) architecture {{REST}} in a suitable form for such constrained nodes.


The terms "payload" and "body" are used as defined in {{-coap-blockwise, Section 2}}.
Note that, when block-wise transfer is not used, the terms "payload" and "body" are to be understood as equal.

For better readability, in the examples in this document the binary payload and resource records are shown in a hexadecimal representation as well as a human-readable format.
In the actual message sent and received, however, they are encoded in the binary message format defined in {{-dns}}.

Selection of a DoC Server   {#sec:doc-server-selection}
=========================

While there is no path specified for the DoC resource, it is RECOMMENDED to use the root path "/"
to keep the CoAP requests small.

The DoC client needs to know the DoC server and the DoC resource at the DoC server.
Possible options to assure this could be manual configuration of a Uniform Resource Identifier (URI) {{-uri}} or Constrained Resource Identifier (CRI) {{-cri}},
or automatic configuration, e.g., using a CoRE resource directory
{{-core-rd}}, DHCP or Router Advertisement options {{-dnr}}, or discovery of designated resolvers
{{-ddr}}.
Automatic configuration SHOULD only be done from a trusted source.

## Discovery by Resource Type
For discovery of the DoC resource through a link mechanism that allows describing a resource type
(e.g., the Resource Type attribute in {{-core-link-format}}), this document introduces the resource type "core.dns".
It can be used to identify a generic DNS resolver that is available to the client.

## Discovery using SVCB Resource Records or DNR
A DoC server can also be discovered using Service Binding (SVCB) Resource Records (RR) {{-svcb}} {{-svcb-dns}} or Discovery of Network-designated Resolvers (DNR)
Service Parameters {{-dnr}}.
{{-coap-tcp}} defines the Application-Layer Protocol Negotiation (ALPN) ID for CoAP over TLS servers and {{-coap-dtls-alpn}} defines the ALPN ID for CoAP over DTLS servers.
Because the ALPN extension is only defined for (D)TLS, these mechanisms cannot be used for DoC servers which use only OSCORE {{-oscore}} and Ephemeral Diffie-Hellman Over COSE (EDHOC) {{-edhoc}} (with COSE abbreviating "Concise Binary Object Notation (CBOR) Object Signing and Encryption" {{?RFC9052}}) for security.
Specifying an alternate discovery mechanism is out of scope of this specification.
{{-core-dnr}} provides  further exploration of the challenges here.

This document is not an SVCB mapping document for the CoAP schemes
as defined in {{Section 2.4.3 of -svcb}}.
A full SVCB mapping is being prepared in {{-transport-indication}},
generalizing mechanisms that are introduced in this document for discovery of DoC.

This document specifies "docpath" as
a single-valued SvcParamKey that is mandatory for DoC SVCB records.
If the "docpath" SvcParamKey is absent, the service should not be considered a valid DoC service.

The docpath is devided up into segments of the absolute path to the DoC resource (docpath-segment),
each a sequence of 1-255 octets.
In ABNF {{-abnf}}:

~~~
docpath-segment = 1*255OCTET
~~~

Note that this restricts the length of each docpath-segment to at most 255 octets.
Paths with longer segments cannot be advertised with the "docpath" SvcParam and are thus NOT
RECOMMENDED for the path to the DoC resource.

The presentation format value of "docpath" SHALL be a comma-separated list ({{Appendix A.1 of -svcb}})
of 0 or more docpath-segments.
The root path "/" is represented by 0 docpath-segments, i.e., an empty list.
The same considerations for the "," and "\" characters in docpath-segments for zone-file
implementations as for the alpn-ids in an "alpn" SvcParam MAY apply ({{Section 7.1.1 of -svcb}}).

The wire-format value for "docpath" consists of 0 or more sequences of octets prefixed by their
respective length as a single octet.
We call this single octet the length octet.
The length octet and the corresponding sequence form a length-value pair.
These length-value pairs are concatenated to form the SvcParamValue.
These pairs MUST exactly fill the SvcParamValue; otherwise, the SvcParamValue is malformed.
Each such length-value pair represents one segment of the absolute path to the DoC resource.
The root path "/" is represented by 0 length-value pairs, i.e., SvcParamValue length 0.

Note that this format uses the same encoding as the "alpn" SvcParam and can reuse the
decoders and encoders for that SvcParam with the adaption that a length of zero is allowed.
As long as each docpath-segment is of length 0 and 24 octets, it is easily transferred into the path
representation in CRIs {{-cri}} by masking each length octet with the CBOR text string major type 3
(`0x60` as an octet, see {{-cbor}}).
Furthermore, it is easily transferable into a sequence of CoAP Uri-Path options by
mapping each length octet into the Option Delta and Option Length of the corresponding CoAP Uri-Path
option, provided the docpath-segments are all of a length between 0 and 12 octets (see {{-coap,
Section 3.1}}).
Likewise, it can be transferred into a URI path-abempty form by replacing each length octet with the "/" character
None of the abovementioned prevent longer docpath-segments than the considered, they just make the
translation harder, as they require to make space for the longer delimiters, in turn requiring to move octets.

To use the service binding from an SVCB RR, the DoC client MUST send a DoC request constructed from the SvcParams including "docpath".
The construction algorithm for DoC requests is as follows, going through the provided records in order of their priority.

- If the "alpn" SvcParam value for the service is "coap", a CoAP request for CoAP over TLS MUST be constructed.
  If it is "co", a CoAP request for CoAP over DTLS MUST be constructed.
  Any other SvcParamKeys specifying a transport are out of the scope of this document.
- The destination address for the request SHOULD be taken from additional information about the target, e.g., from an AAAA record associated with the target name or from an "ipv6hint" SvcParam value.
  As a fallback, an address MAY be queried for the target name of the SVCB record.
- The destination port for the request MUST be taken from the "port" SvcParam value, if present.
  Otherwise, take the default port of the CoAP transport, e.g., with regards to this specification TCP port 5684 for "coap" or UDP port 5684 for "co".
  This document introduces no limitations on the ports that can be used.
  If a malicious SVCB record allows its originator to influence message payloads, {{Section 12 of -svcb}} recommends placing such restrictions in the SVCB mapping document.
  The records used in this document only infuence the Uri-Path option.
  That option is only sent in the plaintext of an encrytped (D)TLS channel,
  and thus does not warrant any limitations.
- The request URI's hostname component MUST be the Authentication Domain Name (ADN) when obtained through DNR
  and MUST be the target name of the SVCB record when obtained through a `_dns` query
  (if AliasMode is used, to the target name of the AliasMode record).
  This can be achieved efficiently by setting that name in TLS Server Name Indication (SNI) {{?RFC8446}},
  or by setting the Uri-Host option on each request.
- For each element in the CBOR sequence of the "docpath" SvcParam value, a Uri-Path option MUST be added to the request.
- If the request constructed this way receives a response, the same SVCB record MUST be used for construction of future DoC queries.
  If not, or if the endpoint becomes unreachable, the algorithm SHOULD be repeated with the SVCB record with the next highest priority.

A more generalized construction algorithm for any CoAP request can be found in {{-transport-indication}}.

### Examples

[^replace-hex]

[^replace-hex]: RFC Ed.: Since the number for "docpath" was not assigned at the time of writing, we
    used the hex `ff 0a` (in decimal 65290; from the private use range of SvcParamKeys) throughout
    this section. Before publication, please replace `ff 0a` with the hexadecimal representation of
    the final value assigned by IANA in this section.

A typical SVCB resource record response for a DoC server at the root path "/" of the server looks
like the following (the "docpath" SvcParam is the last 4 bytes `ff 0a 00 00` in the binary):

    Resource record (binary):
      04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
      67 00 00 40 00 01 00 00 06 28 00 1e 00 01 03 64
      6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
      01 00 03 02 63 6f ff 0a 00 00

    Resource record (human-readable):
      _dns.example.org.  1576  IN SVCB 1 dns.example.org (
          alpn=co docpath )

The root path is RECOMMENDED but not required. Here are two examples where the "docpath" represents
paths of varying depth. First, "/dns" is provided in the following example
(the last 8 bytes `ff 0a 00 04 03 64 6e 73`):

    Resource record (binary):
      04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
      67 00 00 40 00 01 00 00 00 55 00 22 00 01 03 64
      6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
      01 00 03 02 63 6f ff 0a 00 04 03 64 6e 73

    Resource record (human-readable):
      _dns.example.org.    85  IN SVCB 1 dns.example.org (
          alpn=co docpath=dns )

Second, an examples for the path "/n/s" (the last 8 bytes `ff 0a 00 04 01 6e 01 73`):

    Resource record (binary):
      04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
      67 00 00 40 00 01 00 00 06 6b 00 22 00 01 03 64
      6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
      01 00 03 02 63 6f ff 0a 00 04 01 6e 01 73

    Resource record (human-readable):
      _dns.example.org.   643  IN SVCB 1 dns.example.org (
          alpn=co docpath=n,s )


If the server also provides DNS over HTTPS, "dohpath" and "docpath" MAY co-exist:

    Resource record (binary):
      04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
      67 00 00 40 00 01 00 00 01 ad 00 2b 00 01 03 64
      6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
      01 00 06 02 68 33 02 63 6f 00 07 00 07 2f 7b 3f
      64 6e 73 7d ff 0a 00 00

    Resource record (human-readable):
      _dns.example.org.   429  IN SVCB 1 dns.example.org (
          alpn=h3,co dohpath=/{?dns} docpath )

Basic Message Exchange
======================

The "application/dns-message" Content-Format    {#sec:content-format}
--------------------------------------------
This document defines a CoAP Content-Format identifier for the Internet
media type "application/dns-message" to be the mnemonic 553 — based on the port assignment of DNS.
This media type is defined as in {{Section 6 of -doh}}, i.e., a single DNS message encoded in the DNS on-the-wire format {{-dns}}.
Both DoC client and DoC server MUST be able to parse contents in the "application/dns-message" Content-Format.
For the purposes of this document, only OPCODE 0 (Query) is supported for DNS messages.
Future work might provide specifications and considerations for other values of OPCODE.
Unless another error takes precedence, a DoC server uses RCODE = 4, NotImp {{-dns}}, in its response to a query with an OPCODE that it does not implement (see also {{sec:resp-examples}}).

DNS Queries in CoAP Requests    {#sec:queries}
----------------------------

A DoC client encodes a single DNS query in one or more CoAP request
messages that use the CoAP FETCH {{-coap-fetch}} request method.
Requests SHOULD include an Accept option to indicate the type of content that can be parsed in the response.

Since CoAP provides reliability at the message layer (e.g., through Confirmable messages) the retransmission mechanism of the
DNS protocol as defined in {{-dns}} is not needed.

### Request Format

When sending a CoAP request, a DoC client MUST include the DNS query in the body of the CoAP request.
As specified in {{Section 2.3.1 of -coap-fetch}}, the type of content of the body MUST be indicated using the Content-Format option.
This document specifies the usage of Content-Format "application/dns-message" (for details, see {{sec:content-format}}).
A DoC server MUST be able to parse requests of Content-Format "application/dns-message".

### Support of CoAP Caching {#sec:req-caching}

The DoC client SHOULD set the ID field of the DNS header to 0 to enable a CoAP cache (e.g., a CoAP proxy en-route) to respond to the same DNS queries with a cache entry.
This ensures that the CoAP Cache-Key (see {{-coap-fetch, Section 2}}) does not change when multiple DNS queries for the same DNS data, carried in CoAP requests, are issued.
Apart from losing these caching benefits, there is no harm it not setting it to 0, e.g., when the query was received from somewhere else.
In any instance, a DoC server MUST copy the ID from the query in its response to that query.

### Examples {#sec:req-examples}

The following example illustrates the usage of a CoAP message to
resolve "example.org. IN AAAA" based on the URI "coaps://\[2001:db8::1\]/". The
CoAP body is encoded in the "application/dns-message" Content-Format.

    FETCH coaps://[2001:db8::1]/
    Content-Format: 553 (application/dns-message)
    Accept: 553 (application/dns-message)
    Payload (binary):
      00 00 01 20 00 01 00 00 00 00 00 00 07 65 78 61
      6d 70 6c 65 03 6f 72 67 00 00 1c 00 01

    Payload (human-readable):
      ;; ->>Header<<- opcode: QUERY, status: NOERROR, id: 0
      ;; flags: rd ad; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

      ;; QUESTION SECTION:
      ;example.org.             IN      AAAA


DNS Responses in CoAP Responses
-------------------------------

Each DNS query-response pair is mapped to a CoAP request-response operation.
DNS responses are provided in the body of the CoAP response, i.e., it is also possible to transfer them using block-wise transfer {{-coap-blockwise}}.
A DoC server MUST be able to produce responses in the "application/dns-message"
Content-Format (for details, see {{sec:content-format}}) when requested.
A DoC client MUST be able to understand responses in the "application/dns-message" Content-Format
when it does not send an Accept option.
Any response Content-Format other than "application/dns-message" MUST be indicated with
the Content-Format option by the DoC server.

### Response Codes and Handling DNS and CoAP errors

A DNS response indicates either success or failure in the RCODE of the DNS header (see {{Section 4.1.1 of -dns}}).
It is RECOMMENDED that CoAP responses that carry a parseable DNS response use a 2.05 (Content) response code.

CoAP responses using non-successful response codes MUST NOT contain a DNS response
and MUST only be used for errors in the CoAP layer or when a request does not
fulfill the requirements of the DoC protocol.

Communication errors with an upstream DNS server (e.g., timeouts) MUST be indicated by including a DNS response with the appropriate RCODE in a successful CoAP response, i.e., using a 2.xx response code.
When an error occurs at the CoAP layer, e.g., if an unexpected request method or an unsupported Content-Format in the request are used, the DoC server SHOULD respond with an appropriate CoAP error.

A DoC client might try to repeat a non-successful exchange unless otherwise prohibited.
The DoC client might also decide to repeat a non-successful exchange with a different URI, for instance, when the response indicates an unsupported Content-Format.

### Support of CoAP Caching {#sec:resp-caching}

For reliability and energy saving measures, content decoupling (such as en-route caching on proxies) takes a far greater role than it does in HTTP.
Likewise, CoAP makes it possible to use cache validation to refresh stale cache entries to reduce the number of large response messages.
For cache validation, CoAP implementations regularly use hashing over the message content for ETag generation (see {{-coap, Section 5.10.6}}).
As such, the approach to guarantee the same cache key for DNS responses as proposed in DoH ({{-doh, Section 5.1}}) is not sufficient and needs to be updated so that the TTLs in the response are more often the same regardless of query time.

The DoC server MUST ensure that the sum of the Max-Age value of a CoAP response and any TTL in the
DNS response is less than or equal to the corresponding TTL received from an upstream DNS server.
This also includes the default Max-Age value of 60 seconds (see {{Section 5.10.5 of -coap}}) when no Max-Age option is provided.
The DoC client MUST then add the Max-Age value of the carrying CoAP response to all TTLs in a DNS response on reception and use these calculated TTLs for the associated records.

The RECOMMENDED algorithm for a DoC server to meet the requirement for DoC is as follows:
Set the Max-Age option of a response to the minimum TTL of a DNS response and subtract this value from all TTLs of that DNS response.
This prevents expired records unintentionally being served from an intermediate CoAP cache.
Additionally, if the ETag for cache validation is based on the content of the response, it allows that ETag not to change.
This then remains the case even if the TTL values are updated by an upstream DNS cache.
If only one record set per DNS response is assumed, a simplification of this algorithm is to just set all TTLs in the response to 0 and set the TTLs at the DoC client to the value of the Max-Age option.

If shorter caching periods are plausible, e.g., if the RCODE of the message indicates an error that should only be cached for a minimal duration, the value for the Max-Age option SHOULD be set accordingly.
This value might be 0, but if the DoC server knows that the error will persist, greater values are also conceivable, depending on the projected duration of the error.
The same applies for DNS responses that for any reason do not carry any records with a TTL.

### Examples {#sec:resp-examples}

The following example illustrates the response to the query "example.org. IN
AAAA record", with recursion turned on. Successful responses carry one answer
record including address 2001:db8:1:0:1:2:3:4 and TTL 79689.

A successful response:

    2.05 Content
    Content-Format: 553 (application/dns-message)
    Max-Age: 58719
    Payload (human-readable):
      ;; ->>Header<<- opcode: QUERY, status: NOERROR, id: 0
      ;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

      ;; QUESTION SECTION:
      ;example.org.                 IN      AAAA
      ;; ANSWER SECTION:
      ;example.org.         79689   IN      AAAA    2001:db8:1:0:1:2:3:4

When a DNS error &ndash; NxDomain (RCODE = 3) for "does.not.exist" in this case &ndash; is noted in the DNS response, the CoAP response still indicates success.

    2.05 Content
    Content-Format: 553 (application/dns-message)
    Payload (human-readable):
      ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 0
      ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

      ;; QUESTION SECTION:
      ;does.not.exist.              IN      AAAA

As described in {{sec:content-format}}, a DoC server uses NotImp (RCODE = 4) if it does not support an OPCODE—a DNS Update (OPCODE = 5) for "example.org" in this case.

    2.05 Content
    Content-Format: 553 (application/dns-message)
    Payload (human-readable):
      ;; ->>Header<<- opcode: UPDATE, status: NOTIMP, id: 0
      ;; flags: qr ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

      ;; QUERY SECTION:
      ;example.org.                 IN      AAAA

When an error occurs at the CoAP layer, the DoC server responds with
an appropriate CoAP error, for instance 4.15 (Unsupported Content-Format)
if the Content-Format option in the request was not set to
"application/dns-message" and the Content-Format is not otherwise supported by
the server.

    4.15 Unsupported Content-Format
    [no payload]

Interaction with other CoAP and CoRE Features
=============================================

DNS Push Notifications and CoAP Observe
---------------------------------------
DNS Push Notifications {{-dns-push}} provides the capability to asynchronously notify clients about resource record changes.
However, it results in additional overhead, which conflicts with constrained resources.
This is the reason why it is RECOMMENDED to use CoAP Observe {{-coap-observe}} instead of DNS Push
in the DoC domain.
The DoC server SHOULD provide Observe capabilities on its DoC resource and do as follows.

If the CoAP request indicates that the DoC client wants to observe a resource record, a DoC server
MAY use a DNS Subscribe message instead of a classic DNS query to fetch the
information on behalf of a DoC client.
If this is not supported by the DoC server, it MUST act as if the DoC resource were not observable.

Whenever the DoC server receives a DNS Push message from the DNS
infrastructure for an observed resource record, the DoC server sends an appropriate Observe notification response
to the DoC client.

If no more DoC clients observe a resource record for which the DoC server has an open subscription,
the DoC server MUST use a DNS Unsubscribe message to close its subscription to the
resource record as well.

A DoC server can still provide Observe capabilities to its DoC resource without providing this
proxying to DNS Push, e.g., if it receives new information on a record through other means.

OSCORE
------
It is RECOMMENDED to carry DNS messages protected using OSCORE {{-oscore}} between the DoC client
and the DoC server. The establishment and maintenance of the OSCORE Security Context is out of the
scope of this document.


{{-cachable-oscore}} describes a method to allow cache retrieval of OSCORE responses and discusses
the corresponding implications on message sizes and security properties.

Mapping DoC to DoH
------------------
This document provides no specification on how to map between DoC and DoH, e.g., at a CoAP-to-HTTP-proxy.
In fact, such a direct mapping is NOT RECOMMENDED:
rewriting the FETCH method ({{sec:queries}}) and the TTL rewriting ({{sec:resp-caching}}) as
specified in this draft would be non-trivial.
It is RECOMMENDED to use a DNS forwarder to map between DoC and DoH, as would be the case for
mapping between any other pair of DNS transports.

Considerations for Unprotected Use {#sec:unprotected-coap}
==================================
The use of DoC without confidentiality and integrity protection is NOT RECOMMENDED.
Without secure communication, many possible attacks need to be evaluated in the context of
the application's threat model.
This includes known threats for unprotected DNS {{-dns-threats}} {{-dns-privacy}} and CoAP {{Section 11 of -coap}}.
While DoC does not use the random ID of the DNS header (see {{sec:req-caching}}), equivalent protection against off-path poisoning attacks is achieved by using random large token values for unprotected CoAP requests.
If a DoC message is unprotected it MUST use a random token of at least 2 bytes length to mitigate this kind of poisoning attacks.

Implementation Status
=====================

{::boilerplate rfc7942}

[^remove-impl-status]

[^remove-impl-status]: RFC Ed.: Please remove this section before publication. When deleting this
    section, please also remove RFC7942 from the references of this document.

DoC Client
----------
The authors of this document provide a [DoC client implementation available
in the IoT operating system RIOT][gcoap_dns].

Level of maturity:
: production

Version compatibility:
: draft-ietf-core-dns-over-coap-13

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
: draft-ietf-core-dns-over-coap-13

License:
: MIT

Contact information:
: `Martine S. Lenders <martine.lenders@tu-dresden.de>`

Last update of this information:
: September 2024

Security Considerations
=======================

General CoAP security considerations ({{RFC7252, Section 11}}) apply to DoC.
DoC also inherits the security considerations of the protocols used for secure communication, e.g., OSCORE ({{-oscore, Section 12}}) or DTLS ({{-dtls12, Section 5}} and {{-dtls13, Section 11}}).
Additionally, DoC uses request patterns that require the maintenance of long-lived security
contexts.
{{Section 2.6 of -core-corrclar}} provides insights on what can be done when those are resumed from a new endpoint.

Though DTLS v1.2 {{-dtls12}} was obsoleteted by DTLS v1.3 {{-dtls13}} there are still many CoAP
implementations that still use v1.2 at the time of writing.
As such, this document also accounts for the usage of DTLS v1.2 even though newer versions are
RECOMMENDED when using DTLS to secure CoAP.

When using unprotected CoAP (see {{sec:unprotected-coap}}), setting the ID of a DNS message to 0 as
specified in {{sec:req-caching}} opens the DNS cache of a DoC client to cache poisoning attacks
via response spoofing.
This document requires an unpredictable CoAP token in each DoC query from the client when CoAP is
not secured to mitigate such an attack over DoC (see {{sec:unprotected-coap}}).

For secure communication via DTLS or OSCORE, the impact of a fixed ID on security is limited, as both
harden against injecting spoofed responses.
Consequently, the ID of the DNS message can be set to 0 without any concern in order to leverage the advantages of CoAP caching.

A DoC client must be aware that the DoC server
may communicate unprotected with the upstream DNS infrastructure, e.g., using DNS over UDP.
DoC can only guarantee confidentiality and integrity of communication between parties for which the
security context is exchanged.
The DoC server may use another security context to communicate upstream with both confidentiality and integrity
(e.g., DNS over QUIC {{-doq}}), but, while recommended, this is opaque to the DoC client on the protocol level.
Record integrity can also be ensured upstream using, e.g., DNSSEC {{-dnssec}}.

A DoC client may not be able to perform DNSSEC validation,
e.g., due to code size constraints, or due to the size of the responses.
It may trust its DoC server to perform DNSSEC validation;
how that trust is expressed is out of the scope of this document.
For instance, a DoC client may be, configured to use a particular credential by which it recognizes an eligible DoC server.
That information can also imply trust in the DNSSEC validation by that server.

IANA Considerations
===================

[^replace-xxxx]

[^replace-xxxx]: RFC Ed.: throughout this section, please replace
    RFC-XXXX with the RFC number of this specification and remove this
    note.

This document has the following actions for IANA.

CoAP Content-Formats Registry
-----------------------------

IANA is requested to assign a CoAP Content-Format ID for the "application/dns-message" media
type in the "CoAP Content-Formats" registry, within the "Constrained RESTful Environments (CoRE) Parameters"
registry group {{-coap}}, corresponding to the "application/dns-message" media
type from the "Media Types" registry (see {{-doh}}).

Content Type: application/dns-message

Content Coding: -

ID: 553 (suggested)

Reference: {{-doh}} and \[RFC-XXXX\], {{sec:content-format}}

DNS Service Bindings (SVCB) Registry
------------------------------------

IANA is requested to add the following entry to the "Service Parameter Keys (SvcParamKeys)" registry within the "DNS Service Bindings (SVCB)" registry group.
The definition of this parameter can be found in {{sec:doc-server-selection}}.

| Number  | Name           | Meaning                            | Change Controller | Reference       |
| ------- | -------------- | ---------------------------------- | ----------------- | --------------- |
| 10 (suggested)     | docpath        | DNS over CoAP resource path | IETF | \[RFC-XXXX\], {{sec:doc-server-selection}} |
{: #tab-svc-param-keys title="Values for SvcParamKeys"}

Resource Type (rt=) Link Target Attribute Values Registry
---------------------------------------------------------

 IANA is requested to add a new Resource Type (rt=) Link Target Attribute "core.dns" to the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group.

Value: core.dns

Description: DNS over CoAP resource.

Reference: \[RFC-XXXX\], {{sec:doc-server-selection}}


--- back

Evaluation {#sec:evaluation}
==========
The authors of this document presented the design, implementation, and analysis of DoC in their
paper "Securing Name Resolution in the IoT: DNS over CoAP" {{DoC-paper}}.

Change Log
==========

[^remove-changelog]

[^remove-changelog]: RFC Ed.: Please remove this section before publication.

Since [draft-ietf-core-dns-over-coap-16]
----------------------------------------
- Mention TLS as possible protection mechanism in abstract and intro
- Fix representation format in the docpath examples
- Make docpath wire-format paragraph easier to parse

Since [draft-ietf-core-dns-over-coap-15]
----------------------------------------
- Address Genart and Artart review:
    - Add editor's note about removing RFC7228 reference in case rfc7228bis comes out before
      publication
    - Address minor nits
    - Resolve less well-known abbreviations
    - Name default ports for "coap" and "co"
    - Add reasoning why we also consider DTLS v1.2 (RFC 6347)
    - Add illustrative reference for ETag generation
- Address DNS SVCB SvcParamKeys IANA expert review:
    - docpath: clarifications and examples
    - Rework representation format and wire-format of "docpath"
    - State that we don't do the full SVCB mapping
    - Explicitly do not limit what port= can do.
    - port limitations: We're not the SVCB mapping document
- Address Tsvart Review
    - Prefer ADN for Uri-Host; don't prescribe *how* it is set

Since [draft-ietf-core-dns-over-coap-14]
----------------------------------------
- Remove superfluous and confusing step in SVCB to request algorithm
- Address AD review:
  - Remove RFC8949 as CBOR diagnostic notation reference
  - CoRE-specific FETCH method -> CoAP-specific FETCH method
  - Remove double reference to BCP 219
  - Fix wording and references around SVCB records and ALPN
  - Move format description for examples to Terminology section
  - Retitle section 5 to "Interaction with other CoAP and CoRE Features"
  - Make prevention of poisoning attacks normative for unprotected CoAP
  - Provide specs on if the SHOULD on ID=0 does not apply
  - Make construction algorithm normative
  - Add definition for CoRE
  - General grammar, wording, and spelling cleanups

Since [draft-ietf-core-dns-over-coap-13]
----------------------------------------
- Address shepherd review

Since [draft-ietf-core-dns-over-coap-12]
----------------------------------------
- Address Esko's review
- Address Marcos's review
- Address Mikolai's review

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
- Add reference to DNS Update [\[RFC2136\]](https://datatracker.ietf.org/doc/html/rfc2136), clarify that it is currently not considered
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

The authors of this document want to thank Mike Bishop, Carsten Bormann, Elwyn B. Davies, Esko Dijk, Thomas Fossati, Mikolai Gütschow, Todd Herr, Tommy Pauly, Ben Schwartz, Marco Tiloca, and Tim Wicinski for their feedback and comments.

[draft-ietf-core-dns-over-coap-16]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-16
[draft-ietf-core-dns-over-coap-15]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-15
[draft-ietf-core-dns-over-coap-14]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-14
[draft-ietf-core-dns-over-coap-13]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-13
[draft-ietf-core-dns-over-coap-12]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-12
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
