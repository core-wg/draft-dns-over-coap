---
title: "DNS over the Constrained Application Protocol (DoC)"
abbrev: DoC
docname: draft-ietf-core-dns-over-coap-20
number: 9953
ipr: trust200902
lang: en
consensus: true
obsoletes:
updates:
pi: [toc, symrefs, sortrefs]
category: std
submissiontype: IETF
v: 3

date: 2026-03
area: WIT
workgroup: CoRE
keyword:
  - CoRE
  - CoAP
  - DNS

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
  STD13: dns
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
  RFC8768: coap-hop-limit
  RFC8949: cbor
  RFC9147: dtls13
  RFC9460: svcb
  RFC9461: svcb-dns
  RFC9462: ddr
  RFC9463: dnr
  PRE-RFC9952:
    title: >
      The Application-Layer Protocol Negotiation (ALPN) ID Specification for the Constrained Application Protocol (CoAP) over DTLS
    target: https://www.rfc-editor.org/info/rfc9952
    seriesinfo:
      RFC: PRE-9952
      DOI: 10.17487/PRE-RFC9952
    date: March 2026
    author:
      -
        ins: M. S. Lenders
        surname: Lenders
        fullname: Martine Sophie Lenders
      -
        ins: C. Amsüss
        surname: Amsüss
        fullname: Christian Amsüss
      -
        ins: T. C. Schmidt
        surname: Schmidt
        fullname: Thomas C. Schmidt
      -
        ins: M. Wählisch
        surname: Wählisch
        fullname: Matthias Wählisch

informative:
  BCP219: dns-terminology
  BCP237: dnssec
  RFC3833: dns-threats
  RFC6690: core-link-format
  RFC7228: constr-nodes
  RFC7858: dot
  RFC8094: dodtls
  RFC9076: dns-privacy
  RFC9176: core-rd
  RFC9250: doq
  RFC9528: edhoc
  I-D.ietf-core-href:
    display: CRI
  I-D.ietf-core-transport-indication:
    display: TRANSPORT-IND
  I-D.ietf-iotops-7228bis:
    display: RFC7228bis
  I-D.amsuess-core-cachable-oscore:
    display: CACHABLE-OSCORE
  DoC-paper:
    seriesinfo:
      DOI: 10.1145/3609423
    title: 'Securing Name Resolution in the IoT: DNS over CoAP'
    author:
    - name: Martine S. Lenders
      ins: M. Lenders
      org: TU Dresden, Germany
    - name: Christian Amsüss
      ins: C. Amsüss
      org: Unaffiliated, Vienna, Austria
    - name: Cenk Gündogan
      ins: C. Gündogan
      org: Huawei Technologies, Munich, Germany
    - name: Marcin Nawrocki
      ins: M. Nawrocki
      org: TU Dresden, Germany
    - name: Thomas C. Schmidt
      ins: T. Schmidt
      org: HAW Hamburg, Hamburg, Germany
    - name: Matthias Wählisch
      ins: M. Wählisch
      org: TU Dresden &amp; Barkhausen Institut, Dresden, Germany
    date: '2023-09-28'
    refcontent: Proceedings of the ACM on Networking, vol. 1, no. CoNEXT2, pp. 1-25
  I-D.ietf-core-corr-clar:
    display: CoAP-CORR-CLAR
  REST:
    target: https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
    title: Architectural Styles and the Design of Network-based Software Architectures
    author:
      ins: R. Fielding
      name: Roy Thomas Fielding
      org: University of California, Irvine
    date: 2000
    refcontent:
      "Ph.D. Dissertation, University of California, Irvine"
    format:
      HTML: https://ics.uci.edu/~fielding/pubs/dissertation/top.htm
      PDF: https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
--- abstract

<!--[rfced] FYI: We updated [I-D.ietf-core-coap-dtls-alpn] to [PRE-RFC9952]
for now. We will make the final updates in RFCXML (i.e., remove "PRE-").
-->

<!--[rfced] Please note that the title of the document has been
updated as follows. The abbreviation has been expanded per Section 3.6
of RFC 7322 ("RFC Style Guide"). We also added "the". Please review.

Original:
   DNS over CoAP (DoC)

Current:
   DNS over the Constrained Application Protocol (DoC)
-->

<!--[rfced] May we remove "(CoAPS)" in the Abstract as this
term/abbreviation is not used elsewhere in the document?  Please
review.

Original:
   These CoAP messages can be protected by (D)TLS-Secured CoAP (CoAPS)
   or Object Security for Constrained RESTful Environments (OSCORE) to
   provide encrypted DNS message exchange for constrained devices in
   the Internet of Things (IoT).

Perhaps:
   These CoAP messages can be protected by (D)TLS-Secured CoAP or
   Object Security for Constrained RESTful Environments (OSCORE) to
   provide encrypted DNS message exchange for constrained devices in
   the Internet of Things (IoT).
-->

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
CoAP message exchange. Each CoAP message can be secured by DTLS 1.2 or newer {{-dtls12}} {{-dtls13}}
as well as Object Security for Constrained RESTful Environments (OSCORE) {{-oscore}}
and TLS 1.3 or newer {{-coap-tcp}} {{?RFC8446}}
to ensure message integrity and confidentiality.

The application use case of DoC is inspired by DNS over HTTPS (DoH) {{-doh}}.
However, DoC aims for deployment in the constrained Internet of
Things (IoT), which usually conflicts with the requirements introduced by
HTTPS.
Constrained IoT devices may be restricted in memory, power consumption,
link-layer frame sizes, throughput, and latency. They may
only have a handful kilobytes of both RAM and ROM. They may sleep for long
durations of time, after which they need to refresh the named resources they
know about. Name resolution in such scenarios must take into account link-layer
frame sizes of only a few hundred bytes, bit rates in the magnitude
of kilobits per second, and latencies of several seconds {{-constr-nodes}} {{I-D.ietf-iotops-7228bis}}.

<!--[rfced] FYI: draft-ietf-iotops-7228bis has not been published yet
(currently, its IESG state is "I-D Exists"). Thus, we have left
references to RFC 7228 and draft-ietf-iotops-7228bis as is.

Author note:
   Please remove the {{-constr-nodes}} reference and replace
   it with {{I-D.ietf-iotops-7228bis}} throughout the document in case
   {{I-D.ietf-iotops-7228bis}} becomes an RFC before publication.
-->

In order not to be burdened by the resource requirements of TCP and HTTPS, constrained IoT devices could use DNS over DTLS {{-dodtls}}.
In contrast to DNS over DTLS, DoC
can take advantage of CoAP features to mitigate drawbacks of datagram-based
communication. These features include (1) block-wise transfer {{-coap-blockwise}}, which solves
the Path MTU problem of DNS over DTLS (see {{-dodtls, Section 5}}), (2) CoAP
proxies, which provide an additional level of caching, and (3) reuse of data
structures for application traffic and DNS information, which saves memory
on constrained devices.

To avoid the resource requirements of DTLS or TLS on top of UDP (e.g., introduced by DNS over DTLS {{-dodtls}} or DNS over QUIC {{-doq}}), DoC allows for lightweight message protection based on OSCORE.

~~~ aasvg
              . FETCH coaps://[2001:db8::1]/
             /
            /
           CoAP request
+------+   [DNS query]   +------+------+   DNS query     .---------------.
| DoC  |---------------->| DoC  | DNS  |--- --- --- --->|      DNS        |
|Client|<----------------|Server|Client|<--- --- --- ---| Infrastructure  |
+------+  CoAP response  +------+------+  DNS response   '---------------'
          [DNS response]
   \                           / \                                     /
    '------DNS over CoAP------'   '----DNS over UDP/HTTPS/QUIC/...----'

~~~
{: #fig-overview-arch title="Basic DoC Architecture"}

<!--[rfced] FYI - We updated "authoritive name server" to "authoritative name
server" to match other usage in this document and in other RFCs.

Original:
   That DoC server can be the authoritive name server for the queried
   record or a DNS client (i.e., a stub or recursive resolver) that
   resolves DNS information by using other DNS transports such as DNS
   over UDP [STD13], DNS over HTTPS [RFC8484], or DNS over QUIC
   [RFC9250] when communicating with the upstream DNS infrastructure.

Updated:
   That DoC server can be the authoritative name server for the queried
   record or a DNS client (i.e., a stub or recursive resolver) that
   resolves DNS information by using other DNS transports such as DNS
   over UDP [STD13], DNS over HTTPS [RFC8484], or DNS over QUIC
   [RFC9250] when communicating with the upstream DNS infrastructure.
-->

The most important components of DoC can be seen in {{fig-overview-arch}}: a DoC
client tries to resolve DNS information by sending DNS queries carried within
CoAP requests to a DoC server.
That DoC server can be the authoritative name server for the queried record or a DNS client (i.e., a stub or recursive resolver) that resolves DNS information by using other DNS transports such
as DNS over UDP {{-dns}}, DNS over HTTPS {{-doh}}, or DNS over QUIC {{-doq}} when communicating with the upstream
DNS infrastructure.
Using that information, the DoC server then replies to the queries of the DoC client with DNS
responses carried within CoAP responses.
A DoC server MAY also serve as a DNSSEC validator to provide DNSSEC validation to the more
constrained DoC clients.

Note that this specification is distinct from DoH because the CoAP-specific FETCH method {{-coap-fetch}} is used.
A benefit of using this method is having the DNS query in the body such as when using the POST method, but with the same caching advantages of responses to requests that use the GET method.
Having the DNS query in the body means that there is no need for extra base64 encoding, which would increase
code complexity and message sizes.
Also, this allows for the block-wise transfer of queries {{-coap-blockwise}}.


Terminology and Conventions
===========================

{::boilerplate bcp14-tagged}

A server that provides the service specified in this document is called a "DoC
server" to differentiate it from a classic "DNS server".
A DoC server acts as either a DNS stub resolver or a DNS recursive resolver {{-dns-terminology}}.
As such, the DoC server communicates with an "upstream DNS infrastructure" or a single "upstream DNS server".
A "DoC resource" is a CoAP resource {{-coap}} at the DoC server that DoC clients can target in order to send a DNS query in a CoAP request.

A client using the service specified in this document to retrieve
the DNS information is called a "DoC client".

The term "constrained nodes" is used as defined in {{-constr-nodes}}.
{{-core-link-format}} describes that Constrained RESTful Environments (CoRE) realize the Representational State Transfer (REST) architecture {{REST}} in a suitable form for such constrained nodes.

A DoC server can provide Observe capabilities as defined in {{-coap-observe, Section 1.2}}.
As part of that, it administers a "list of observers". DoC clients using these capabilities are "observers" as defined in {{-coap-observe, Section 1.2}}.
A "notification" is a CoAP response message with an Observe option; see {{-coap-observe, Section 4.2}}.

The terms "payload" and "body" are used as defined in {{-coap-blockwise, Section 2}}.
Note that, when block-wise transfer is not used, the terms "payload" and "body" are to be understood as equal.

In the examples in this document, the binary payload and resource records are shown in a hexadecimal representation as well as a human-readable format for better readability. However, in the actual message sent and received, they are encoded in the binary message format defined in {{-dns}}.

Selection of a DoC Server   {#sec:doc-server-selection}
=========================

While there is no path specified for the DoC resource, it is RECOMMENDED to use the root path "/"
to keep the CoAP requests small.

The DoC client needs to know the DoC server and the DoC resource at the DoC server.
Possible options to assure this could be (1) manual configuration of a Uniform Resource Identifier (URI) {{-uri}} or Constrained Resource Identifier (CRI) {{I-D.ietf-core-href}}
or (2) automatic configuration, e.g., using a CoRE resource directory
{{-core-rd}}, DHCP or Router Advertisement options {{-dnr}}, or discovery of designated resolvers
{{-ddr}}.
Automatic configuration MUST only be done from a trusted source.

## Discovery by Resource Type
For discovery of the DoC resource through a link mechanism that allows describing a resource type
(e.g., the Resource Type attribute in {{-core-link-format}}), this document introduces the resource type "core.dns".
It can be used to identify a generic DNS resolver that is available to the client.

## Discovery Using SVCB Resource Records or DNR
A DoC server can also be discovered using Service Binding (SVCB) Resource Records (RRs) {{-svcb}} {{-svcb-dns}} resolved via another DNS service (e.g., provided by an unencrypted local resolver) or Discovery of Network-designated Resolvers (DNR)
Service Parameters {{-dnr}} via DHCP or Router Advertisements.
{{-coap-tcp}} defines the Application-Layer Protocol Negotiation (ALPN) ID for CoAP over TLS servers and {{PRE-RFC9952}} defines the ALPN ID for CoAP over DTLS servers.
DoC servers that use only OSCORE {{-oscore}} and Ephemeral Diffie-Hellman Over COSE (EDHOC) {{-edhoc}} (COSE stands for "Concise Binary Object Notation (CBOR) Object Signing and Encryption" {{?RFC9052}}) to support security cannot be discovered using these SVCB RR or DNR mechanisms.
Specifying an alternate discovery mechanism is out of the scope of this document.

This document is not an SVCB mapping document for the CoAP schemes
as defined in {{Section 2.4.3 of -svcb}}.
A full SVCB mapping is specified in {{I-D.ietf-core-transport-indication}}.
It generalizes mechanisms for all CoAP services.
This document introduces only the discovery of DoC services.

This document specifies "docpath" as
a single-valued Service Parameter Key (SvcParamKey) that is mandatory for DoC SVCB records.
If the "docpath" SvcParamKey is absent, the service should not be considered a valid DoC service.

The docpath is divided up into segments of the absolute path to the DoC resource (docpath-segment),
each a sequence of 1-255 octets.
In ABNF {{-abnf}}:

~~~ abnf
docpath-segment = 1*255OCTET
~~~

Note that this restricts the length of each docpath-segment to at most 255 octets.
Paths with longer segments cannot be advertised with the "docpath" SvcParam and are thus NOT
RECOMMENDED for the path to the DoC resource.

The presentation format value of "docpath" SHALL be a comma-separated list ({{Appendix A.1 of -svcb}})
of 0 or more docpath-segments.
The root path "/" is represented by 0 docpath-segments, i.e., an empty list.

<!-- [note to rfced] there must be two '\' here for the markdown to be rendered properly -->

The same considerations apply for the "," and "\\" characters in docpath-segments for zone-file
implementations and the alpn-ids in an "alpn" SvcParam ({{Section 7.1.1 of -svcb}}).

The wire-format value for "docpath" consists of 0 or more sequences of octets prefixed by their
respective length as a single octet.
We call this single octet the length octet.
The length octet and the corresponding sequence form a length-value pair.
These length-value pairs are concatenated to form the SvcParamValue.
These pairs MUST exactly fill the SvcParamValue; otherwise, the SvcParamValue is malformed.
Each such length-value pair represents one segment of the absolute path to the DoC resource.
The root path "/" is represented by 0 length-value pairs, i.e., SvcParamValue length 0.

<!-- [rfced] Please clarify "is of length 0 and 24 octets" in this sentence.

Original:
   As long as each docpath-
   segment is of length 0 and 24 octets, it is easily transferred into
   the path representation in CRIs [I-D.ietf-core-href] by masking each
   length octet with the CBOR text string major type 3 (0x60 as an
   octet, see [RFC8949]).

Perhaps:
   As long as each docpath-
   segment has a length between 0 and 24 octets, it is easily transferred into
   the path representation in CRIs [CRI] by masking each length octet
   with the CBOR text string major type 3 (0x60 as an octet; see
   [RFC8949]).
-->

<!--[rfced] We are having trouble parsing this sentence. Please let us
know if it can be revised as shown below for clarity.

Original:
   Likewise, it can be transferred into a URI path-abempty form by
   replacing each length octet with the "/" character None of the
   abovementioned prevent longer docpath-segments than the considered,
   they just make the translation harder, as they require to make space
   for the longer delimiters, in turn requiring to move octets.

Perhaps:
   Likewise, it can be transferred into a URI path-abempty form by
   replacing each length octet with the "/" character. None of the
   abovementioned prevent longer docpath-segments than the considered
   ones; they just make the translation harder as space is required
   for the longer delimiters, which in turn require octets to be
   moved.
-->

<!-- [rfced] May we update "going through" to "with" here to improve clarity?

Original:
   The construction algorithm for DoC
   requests is as follows, going through the provided records in order
   of their priority.

Perhaps:
   The construction algorithm for DoC
   requests is as follows, with the provided records in order
   of their priority.
-->

<!-- [note to rfced] there was a technical error in the submitted version -20 of the draft:

Original:
    As long as each docpath-segment is of length 0 and 24 octets, ...

Correction:
    As long as each docpath-segment is of a length between 0 and 23 octets, ...

-->

Note that this format uses the same encoding as the "alpn" SvcParam, and it can reuse the
decoders and encoders for that SvcParam with the adaption that a length of zero is allowed.
As long as each docpath-segment is of a length between 0 and 23 octets, it is easily transferred into the path
representation in CRIs {{I-D.ietf-core-href}} by masking each length octet with the CBOR text string major type 3
(`0x60` as an octet; see {{-cbor}}).
Furthermore, it is easily transferable into a sequence of CoAP Uri-Path options by
mapping each length octet into the Option Delta and Option Length of the corresponding CoAP Uri-Path
option, provided the docpath-segments are all of a length between 0 and 12 octets (see {{-coap,
Section 3.1}}). Likewise, it can be transferred into a URI path-abempty form by replacing each length octet with the "/" character
None of the abovementioned prevent longer docpath-segments than the considered, they just make the
translation harder, as they require to make space for the longer delimiters, in turn requiring to move octets.

To use the service binding from an SVCB RR or DNR Encrypted DNS option, the DoC client MUST send a DoC request constructed from the SvcParams including "docpath".
The construction algorithm for DoC requests is as follows, going through the provided records in order of their priority.
For the purposes of this algorithm, the DoC client is assumed to be SVCB-optional (see {{Section 3 of -svcb}}).

<!-- [rfced] How may we update the third item in the series for parallel
structure? Would either removing "from" or adding "information" be correct?

Original:
   This may include (1) A
   or AAAA RRs associated with the target name and delivered with the
   SVCB RR (see [RFC9462]), (2) "ipv4hint" or "ipv6hint" SvcParams
   from the SVCB RR (see [RFC9461]), or (3) from IPv4 or IPv6
   addresses provided if DNR [RFC9463] is used.

Perhaps A (cut "from"):
   This may include (1) A
   or AAAA RRs associated with the target name and delivered with the
   SVCB RR (see [RFC9462]), (2) "ipv4hint" or "ipv6hint" SvcParams
   from the SVCB RR (see [RFC9461]), or (3) IPv4 or IPv6
   addresses provided if DNR [RFC9463] is used.

or
Perhaps B (add "information"):
   This may include (1) A
   or AAAA RRs associated with the target name and delivered with the
   SVCB RR (see [RFC9462]), (2) "ipv4hint" or "ipv6hint" SvcParams
   from the SVCB RR (see [RFC9461]), or (3) information from IPv4 or IPv6
   addresses provided if DNR [RFC9463] is used.
-->

- If the "alpn" SvcParam value for the service is "coap", a CoAP request for CoAP over TLS MUST be constructed {{-coap-tcp}}.
  If it is "co", a CoAP request for CoAP over DTLS MUST be constructed {{PRE-RFC9952}}.
  Any other SvcParamKeys specifying a transport are out of the scope of this document.
- The destination address for the request SHOULD be taken from additional information about the target.
  This may include (1) A or AAAA RRs associated with the target name and delivered with the SVCB RR (see {{-ddr}}), (2) "ipv4hint" or "ipv6hint" SvcParams from the SVCB RR (see {{-svcb-dns}}), or (3) from IPv4 or IPv6 addresses provided if DNR {{-dnr}} is used.
  As a fallback, an address MAY be queried for the target name of the SVCB record from another DNS service.
- The destination port for the request MUST be taken from the "port" SvcParam value, if present.
  Otherwise, take the default port of the CoAP transport, e.g., with regards to this specification, the default is TCP port 5684 for "coap" or UDP port 5684 for "co".
  This document introduces no limitations on the ports that can be used.
  If a malicious SVCB record allows its originator to influence message payloads, {{Section 12 of -svcb}} recommends placing such restrictions in the SVCB mapping document.
  The records used in this document only influence the Uri-Path option.
  That option is only sent in the plaintext of an encrypted (D)TLS channel
  and thus does not warrant any limitations.
- The request URI's hostname component MUST be the Authentication Domain Name (ADN) when obtained through DNR
  and MUST be the target name of the SVCB record when obtained through a `_dns` query
  (if AliasMode is used to the target name of the AliasMode record).
  This can be achieved efficiently by setting that name in TLS Server Name Indication (SNI) {{?RFC8446}}
  or by setting the Uri-Host option on each request.
- For each element in the CBOR sequence of the "docpath" SvcParam value, a Uri-Path option MUST be added to the request.
- If the request constructed this way receives a response, the same SVCB record MUST be used for construction of future DoC queries.
  If not, or if the endpoint becomes unreachable, the algorithm repeats with the SVCB RR or DNR Encrypted DNS option with the next highest Service Priority as a fallback (see {{Sections 2.4.1 and 3 of -svcb}}).

A more generalized construction algorithm for any CoAP request can be found in {{I-D.ietf-core-transport-indication}}.

### Examples

<!--[rfced] Per the following note, we have replaced "ff 0a" with "00 0a" in
the examples in Section 3.2.1 (per IANA's assignment of "10" for
"docpath"). Please confirm that this is correct and let us know if any further
updates are needed.

Author note:
   Since the number for "docpath" was not assigned at the time of
   writing, we used the hex `ff 0a` (in decimal 65290; from the
   private use range of SvcParamKeys) throughout this section. Before
   publication, please replace `ff 0a` with the hexadecimal
   representation of the final value assigned by IANA in this
   section. Please remove this paragraph after that.
-->

A typical SVCB resource record response for a DoC server at the root path "/" of the server looks
like the following (the "docpath" SvcParam is the last 4 bytes `00 0a 00 00` in the binary):

~~~
Resource record (binary):
  04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
  67 00 00 40 00 01 00 00 06 28 00 1e 00 01 03 64
  6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
  01 00 03 02 63 6f 00 0a 00 00

Resource record (human-readable):
  _dns.example.org.  1576  IN SVCB 1 dns.example.org (
      alpn=co docpath )
~~~
{: gi="sourcecode"}

The root path is RECOMMENDED but not required. Here are two examples where the "docpath" represents
paths of varying depth. First, "/dns" is provided in the following example
(the last 8 bytes `00 0a 00 04 03 64 6e 73`):

~~~
Resource record (binary):
  04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
  67 00 00 40 00 01 00 00 00 55 00 22 00 01 03 64
  6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
  01 00 03 02 63 6f 00 0a 00 04 03 64 6e 73

Resource record (human-readable):
  _dns.example.org.    85  IN SVCB 1 dns.example.org (
      alpn=co docpath=dns )
~~~
{: gi="sourcecode"}

Second, see an example for the path "/n/s" (the last 8 bytes `00 0a 00 04 01 6e 01 73`):

~~~
Resource record (binary):
  04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
  67 00 00 40 00 01 00 00 06 6b 00 22 00 01 03 64
  6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
  01 00 03 02 63 6f 00 0a 00 04 01 6e 01 73

Resource record (human-readable):
  _dns.example.org.   643  IN SVCB 1 dns.example.org (
      alpn=co docpath=n,s )
~~~
{: gi="sourcecode"}

If the server also provides DNS over HTTPS, "dohpath" and "docpath" MAY coexist:

~~~
Resource record (binary):
  04 5f 64 6e 73 07 65 78 61 6d 70 6c 65 03 6f 72
  67 00 00 40 00 01 00 00 01 ad 00 2b 00 01 03 64
  6e 73 07 65 78 61 6d 70 6c 65 03 6f 72 67 00 00
  01 00 06 02 68 33 02 63 6f 00 07 00 07 2f 7b 3f
  64 6e 73 7d 00 0a 00 00

Resource record (human-readable):
  _dns.example.org.   429  IN SVCB 1 dns.example.org (
      alpn=h3,co dohpath=/{?dns} docpath )
~~~
{: gi="sourcecode"}

Basic Message Exchange
======================

The "application/dns-message" Content-Format    {#sec:content-format}
--------------------------------------------
This document defines a CoAP Content-Format ID for the Internet
media type "application/dns-message" to be the mnemonic 553, based on the port assignment of DNS.
This media type is defined as in {{Section 6 of -doh}}, i.e., a single DNS message encoded in the DNS on-the-wire format {{-dns}}.
Both DoC client and DoC server MUST be able to parse contents in the "application/dns-message" Content-Format.
This document only specifies OPCODE 0 (Query) for DNS over CoAP messages.
Future documents can provide considerations for additional OPCODEs or extend its specification (e.g., by describing whether other CoAP codes need to be used for which OPCODE).
Unless another error takes precedence, a DoC server uses RCODE = 4, NotImp {{-dns}}, in its response to a query with an OPCODE that it does not implement (see also {{sec:resp-examples}}).

DNS Queries in CoAP Requests    {#sec:queries}
----------------------------

A DoC client encodes a single DNS query in one or more CoAP request
messages that use the CoAP FETCH {{-coap-fetch}} request method.
Requests SHOULD include an Accept option to indicate the type of content that can be parsed in the response.

Since CoAP provides reliability at the message layer (e.g., through Confirmable messages), the retransmission mechanism of the
DNS protocol as defined in {{-dns}} is not needed.

### Request Format

When sending a CoAP request, a DoC client MUST include the DNS query in the body of the CoAP request.
As specified in {{Section 2.3.1 of -coap-fetch}}, the type of content of the body MUST be indicated using the Content-Format option.
This document specifies the usage of Content-Format "application/dns-message" (for details, see {{sec:content-format}}).

### Support of CoAP Caching {#sec:req-caching}

<!--[rfced] We note that "Cache-Key" appears as "cache key" in RFC
8132. Would you like to match use in RFC 8132?

Original:
   This ensures that the CoAP Cache-Key (see [RFC8132], Section 2)
   does not change when multiple DNS queries for the same DNS data,
   carried in CoAP requests, are issued.

Perhaps:
   This ensures that the CoAP cache key (see [RFC8132], Section 2)
   does not change when multiple DNS queries for the same DNS data,
   carried in CoAP requests, are issued.
-->
The DoC client SHOULD set the ID field of the DNS header to 0 to enable a CoAP cache (e.g., a CoAP proxy en route) to respond to the same DNS queries with a cache entry.
This ensures that the CoAP Cache-Key (see {{-coap-fetch, Section 2}}) does not change when multiple DNS queries for the same DNS data, carried in CoAP requests, are issued.
Apart from losing these caching benefits, there is no harm in not setting it to 0, e.g., when the query was received from somewhere else.
In any instance, a DoC server MUST copy the ID from the query in its response to that query.

### Example {#sec:req-examples}

The following example illustrates the usage of a CoAP message to
resolve "example.org. IN AAAA" based on the URI "coaps://\[2001:db8::1\]/". The
CoAP body is encoded in the "application/dns-message" Content-Format.

~~~
FETCH coaps://[2001:db8::1]/
Content-Format: 553 (application/dns-message)
Accept: 553 (application/dns-message)
Payload (binary):
  00 00 01 00 00 01 00 00 00 00 00 00 07 65 78 61
  6d 70 6c 65 03 6f 72 67 00 00 1c 00 01

Payload (human-readable):
  ;; ->>Header<<- opcode: QUERY, status: NOERROR, id: 0
  ;; flags: rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

  ;; QUESTION SECTION:
  ;example.org.             IN      AAAA
~~~
{: gi="sourcecode"}

DNS Responses in CoAP Responses
-------------------------------

Each DNS query-response pair is mapped to a CoAP request-response operation.
DNS responses are provided in the body of the CoAP response, i.e., it is also possible to transfer them using block-wise transfer {{-coap-blockwise}}.
A DoC server MUST be able to produce responses in the "application/dns-message"
Content-Format (for details, see {{sec:content-format}}) when requested.
The use of the Accept option in the request is optional.
However, all DoC clients MUST be able to parse an "application/dns-message" response (see also {{sec:content-format}}).
Any response Content-Format other than "application/dns-message" MUST be indicated with
the Content-Format option by the DoC server.

### Response Codes and Handling DNS and CoAP Errors {#sec:resp-codes}

A DNS response indicates either success or failure in the RCODE of the DNS header (see {{-dns}}).
It is RECOMMENDED that CoAP responses that carry a parsable DNS response use a 2.05 (Content) response code.

CoAP responses using non-successful response codes MUST NOT contain a DNS response
and MUST only be used for errors in the CoAP layer or when a request does not
fulfill the requirements of the DoC protocol.

Communication errors with an upstream DNS server (e.g., timeouts) MUST be indicated by including a DNS response with the appropriate RCODE in a successful CoAP response, i.e., using a 2.xx response code.
When an error occurs at the CoAP layer, e.g., if an unexpected request method or an unsupported Content-Format in the request are used, the DoC server SHOULD respond with an appropriate CoAP error.

A DoC client might try to repeat a non-successful exchange unless otherwise prohibited.
The DoC client might also decide to repeat a non-successful exchange with a different URI, for instance, when the response indicates an unsupported Content-Format.

### Support of CoAP Caching {#sec:resp-caching}

For reliability and energy-saving measures, content decoupling (such as en-route caching on proxies) takes a far greater role than it does in HTTP.
Likewise, CoAP makes it possible to use cache validation to refresh stale cache entries to reduce the number of large response messages.
For cache validation, CoAP implementations regularly use hashing over the message content for ETag generation (see {{-coap, Section 5.10.6}}).
As such, the approach to guarantee the same cache key for DNS responses as proposed in DoH ({{-doh, Section 5.1}}) is not sufficient and needs to be updated so that the TTLs in the response are more often the same regardless of query time.

The DoC server MUST ensure that the sum of the Max-Age value of a CoAP response and any TTL in the
DNS response is less than or equal to the corresponding TTL received from an upstream DNS server.
This also includes the default Max-Age value of 60 seconds (see {{Section 5.10.5 of -coap}}) when no Max-Age option is provided.
The DoC client MUST then add the Max-Age value of the carrying CoAP response to all TTLs in a DNS response on reception and use these calculated TTLs for the associated records.

To meet the requirement for DoC, the RECOMMENDED algorithm for a DoC server is as follows:
Set the Max-Age option of a response to the minimum TTL of a DNS response and subtract this value from all TTLs of that DNS response.
This prevents expired records from unintentionally being served from an intermediate CoAP cache.
Additionally, if the ETag for cache validation is based on the content of the response, it allows that ETag not to change.
This then remains the case even if the TTL values are updated by an upstream DNS cache.
If only one record set per DNS response is assumed, a simplification of this algorithm is to just set all TTLs in the response to 0 and set the TTLs at the DoC client to the value of the Max-Age option.

If shorter caching periods are plausible, e.g., if the RCODE of the message indicates an error that should only be cached for a minimal duration, the value for the Max-Age option SHOULD be set accordingly.
This value might be 0, but if the DoC server knows that the error will persist, greater values are also conceivable, depending on the projected duration of the error.
The same applies for DNS responses that, for any reason, do not carry any records with a TTL.

### Examples {#sec:resp-examples}

The following example illustrates the response to the query "example.org. IN
AAAA record", with recursion turned on. Successful responses carry one answer
record including the address 2001:db8:1:0:1:2:3:4 and TTL 79689.

A successful response:

~~~
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
~~~
{: gi="sourcecode"}

When a DNS error -- NxDomain (RCODE = 3) for "does.not.exist" in this case -- is noted in the DNS response, the CoAP response still indicates success.

~~~
2.05 Content
Content-Format: 553 (application/dns-message)
Payload (human-readable):
  ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 0
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

  ;; QUESTION SECTION:
  ;does.not.exist.              IN      AAAA
~~~
{: gi="sourcecode"}

<!-- [rfced] Please review the text starting with "OPCODE—a DNS
Update ...". Should this be updated as follows or in some other way?

Original:
   As described in Section 4.1, a DoC server uses NotImp (RCODE = 4) if
   it does not support an OPCODE—a DNS Update (OPCODE = 5) for
   "example.org" in this case.

Perhaps:
   As described in Section 4.1, a DoC server uses NotImp (RCODE = 4) if
   it does not support an OPCODE - in this case, a DNS Update (OPCODE = 5) for
   "example.org" is used.
-->

As described in {{sec:content-format}}, a DoC server uses NotImp (RCODE = 4) if it does not support an OPCODE-a DNS Update (OPCODE = 5) for "example.org" in this case.

~~~
2.05 Content
Content-Format: 553 (application/dns-message)
Payload (human-readable):
  ;; ->>Header<<- opcode: UPDATE, status: NOTIMP, id: 0
  ;; flags: qr ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

  ;; QUERY SECTION:
  ;example.org.                 IN      AAAA
~~~
{: gi="sourcecode"}

When an error occurs at the CoAP layer, the DoC server responds with
an appropriate CoAP error, for instance, 4.15 (Unsupported Content-Format)
if the Content-Format option in the request was not set to
"application/dns-message" and the Content-Format is not otherwise supported by
the server.

~~~
4.15 Unsupported Content-Format
[no payload]
~~~
{: gi="sourcecode"}

Interaction with Other CoAP and CoRE Features
=============================================

DNS Push Notifications and CoAP Observe
---------------------------------------
DNS Push Notifications {{-dns-push}} provide the capability to asynchronously notify clients about resource record changes.
However, it results in additional overhead, which conflicts with constrained resources.
This is the reason why it is RECOMMENDED to use CoAP Observe {{-coap-observe}} instead of DNS Push
in the DoC domain.
This is particularly useful if a short-lived record is requested frequently.
The DoC server SHOULD provide Observe capabilities on its DoC resource and do as follows.

If a DoC client wants to observe a resource record, a DoC server can respond with a notification
and add the client to its list of observers for that resource in accordance with {{-coap-observe}}.
The DoC server MAY subscribe to DNS push notifications for that record.
This involves sending a DNS Subscribe message (see {{Section 6.2 of -dns-push}}),
instead of a classic DNS query to fetch the
information on behalf of the DoC client.

<!--[rfced] Please clarify what "a failure to do so" refers to in the
following sentence.

Original:
   As there is no CoAP observer anymore from the perspective of the
   DoC server, a failure to do so cannot be communicated back to any
   DoC observer.
-->

After the list of observers for a particular DNS query has become empty
(by explicit or implicit cancellation of the observation as per {{Section 3.6 of -coap-observe}}),
and no other reason to subscribe to that request is present,
the DoC server SHOULD cancel the corresponding subscription.
This can involve sending a DNS Unsubscribe message or closing the session (see {{Section 6.4 of -dns-push}}).
As there is no CoAP observer anymore from the perspective of the DoC server, a failure to do so cannot be communicated back to any DoC observer.
As such, error handling (if any) needs to be resolved between the DoC server and the upstream DNS infrastructure.

Whenever the DoC server receives a DNS Push message from the DNS
infrastructure for an observed resource record, the DoC server sends an appropriate Observe notification response
to the DoC client.

A server that responds with notifications (i.e., sends the Observe option) needs to have the means of obtaining current resource records.
This may happen through DNS Push or also by upstream polling or implicit circumstances (e.g., if the DoC server is the authoritative name server for the record and wants to notify about changes).

OSCORE
------
It is RECOMMENDED to carry DNS messages protected using OSCORE {{-oscore}} between the DoC client
and the DoC server. The establishment and maintenance of the OSCORE security context is out of the
scope of this document.


{{I-D.amsuess-core-cachable-oscore}} describes a method to allow cache retrieval of OSCORE responses and discusses
the corresponding implications on message sizes and security properties.

Mapping DoC to DoH
------------------
This document provides no specification on how to map between DoC and DoH, e.g., at a CoAP-to-HTTP proxy. Such a direct mapping is NOT RECOMMENDED:
Rewriting the FETCH method ({{sec:queries}}) and TTL ({{sec:resp-caching}}) as
specified in this document would be non-trivial.
It is RECOMMENDED to use a DNS forwarder to map between DoC and DoH, as would be the case for
mapping between any other pair of DNS transports.

Considerations for Unprotected Use {#sec:unprotected-coap}
==================================
The use of DoC without confidentiality and integrity protection is NOT RECOMMENDED.
Without secure communication, many possible attacks need to be evaluated in the context of
the application's threat model.
This includes known threats for unprotected DNS {{-dns-threats}} {{-dns-privacy}} and CoAP ({{Section 11 of -coap}}).
While DoC does not use the random ID of the DNS header (see {{sec:req-caching}}), equivalent protection against off-path poisoning attacks is achieved by using random large token values for unprotected CoAP requests.
If a DoC message is unprotected, it MUST use a random token with a length of at least 2 bytes to mitigate this kind of poisoning attack.

Security Considerations
=======================

General CoAP security considerations ({{RFC7252, Section 11}}) apply to DoC.
DoC also inherits the security considerations of the protocols used for secure communication, e.g., OSCORE ({{-oscore, Section 12}}) as well as DTLS 1.2 or newer ({{-dtls12, Section 5}} and {{-dtls13, Section 11}}).
Additionally, DoC uses request patterns that require the maintenance of long-lived security
contexts.
{{Section 2.6 of I-D.ietf-core-corr-clar}} provides insights on what can be done when those are resumed from a new endpoint.

Though DTLS v1.2 {{-dtls12}} was obsoleted by DTLS v1.3 {{-dtls13}}, there are many CoAP
implementations that still use v1.2 at the time of writing.
As such, this document also accounts for the usage of DTLS v1.2 even though newer versions are
RECOMMENDED when using DTLS to secure CoAP.

When using unprotected CoAP (see {{sec:unprotected-coap}}), setting the ID of a DNS message to 0 as
specified in {{sec:req-caching}} opens the DNS cache of a DoC client to cache poisoning attacks
via response spoofing.
This document requires an unpredictable CoAP token in each DoC query from the client when CoAP is
not secured to mitigate such an attack over DoC (see {{sec:unprotected-coap}}).

<!--[rfced] FYI: We added "to protect" to this sentence for
clarity. Please let us know if it changes the intended meaning.

Original:
   For secure communication via (D)TLS or OSCORE, an unpredictable ID
   against spoofing is not necessary.

Updated:
   For secure communication via (D)TLS or OSCORE, an unpredictable ID
   to protect against spoofing is not necessary.
-->

For secure communication via (D)TLS or OSCORE, an unpredictable ID to protect against spoofing is not necessary.
Both (D)TLS and OSCORE offer mechanisms to harden against injecting spoofed responses in their protocol design.
Consequently, the ID of the DNS message can be set to 0 without any concern in order to leverage the advantages of CoAP caching.

A DoC client must be aware that the DoC server
may communicate unprotected with the upstream DNS infrastructure, e.g., using DNS over UDP.
DoC can only guarantee confidentiality and integrity of communication between parties for which the
security context is exchanged.
The DoC server may use another security context to communicate upstream with both confidentiality and integrity
(e.g., DNS over QUIC {{-doq}}); however, while recommended, this is opaque to the DoC client on the protocol level.
Record integrity can also be ensured upstream using DNSSEC {{-dnssec}}.

A DoC client may not be able to perform DNSSEC validation,
e.g., due to code size constraints or the size of the responses.
It may trust its DoC server to perform DNSSEC validation;
how that trust is expressed is out of the scope of this document.
For instance, a DoC client may be configured to use a particular credential by which it recognizes an eligible DoC server.
That information can also imply trust in the DNSSEC validation by that DoC server.

IANA Considerations
===================

CoAP Content-Formats Registry
-----------------------------

IANA has assigned a CoAP Content-Format ID for the "application/dns-message" media
type in the "CoAP Content-Formats" registry in the "Constrained RESTful Environments (CoRE) Parameters"
registry group {{-coap}}; this corresponds to the "application/dns-message" media
type from the "Media Types" registry (see {{-doh}}).

| Content Type | ID       | Reference                          |
| ------------ | -------- | ---------------------------------- |
| application/dns-message | 553       | {{-doh}} and RFC 9953, {{sec:content-format}} |
{: #tab-coap-content-format title="CoAP Content-Format ID"}

DNS SVBC Service Parameter Keys (SvcParamKeys) Registry
------------------------------------

IANA has added the following entry to the "DNS SVCB Service Parameter Keys (SvcParamKeys)" registry in the "DNS Service Bindings (SVCB)" registry group.
The definition of this parameter can be found in {{sec:doc-server-selection}}.

| Number  | Name           | Meaning                            | Change Controller | Reference       |
| ------- | -------------- | ---------------------------------- | ----------------- | --------------- |
| 10     | docpath        | DNS over CoAP resource path | IETF | RFC 9953, {{sec:doc-server-selection}} |
{: #tab-svc-param-keys title="Value for SvcParamKeys"}

Resource Type (rt=) Link Target Attribute Values Registry
---------------------------------------------------------

IANA has added "core.dns" to the "Resource Type (rt=) Link Target Attribute Values" registry in the "Constrained RESTful Environments (CoRE) Parameters" registry group.

| Value        | Description  | Reference       |
| ------------ | ------------ | --------------- |
| core.dns     | DNS over CoAP resource         | RFC 9953, {{sec:doc-server-selection}} |
{: #tab-resource-type title="Resource Type (rt=) Link Target Attribute"}

Operational Considerations
==========================

## Coexistence of Different DNS and CoAP Transports

Many DNS transports may coexist on the DoC server, such as DNS over UDP {{-dns}}, DNS over (D)TLS {{-dot}} {{-dodtls}}, DNS over HTTPS {{-doh}}, or DNS over QUIC {{-doq}}.
In principle, transports employing channel or object security should be preferred.
In constrained scenarios, DNS over CoAP is preferable to DNS over DTLS.
The final decision regarding the preference, however, heavily depends on the use case and is therefore left to the implementers or users and is not defined in this document.

CoAP supports Confirmable and Non-Confirmable messages {{-coap}} to deploy different levels of reliability.
However, this document does not enforce any of these message types, as the decision on which one is appropriate depends on the characteristics of the network where DoC is deployed.

## Redirects

Application-layer redirects (e.g., HTTP) redirect a client to a new server.
In the case of DoC, this leads to a new DNS server.
This new DNS server may provide different answers to the same DNS query than the previous DNS server.
At the time of writing, CoAP does not support redirection.
Future specifications of CoAP redirect may need to consider the impact of different results between previous and new DNS servers.

## Proxy Hop Limit

Mistakes might lead to CoAP proxies forming infinite loops.
Using the CoAP Hop-Limit option {{-coap-hop-limit}} mitigates such loops.

## Error Handling

{{sec:resp-codes}} specifies that DNS operational errors should be reported back to a DoC client
using the appropriate DNS RCODE.
If a DoC client did not receive any successful DNS messages from a DoC server for a while, it might
indicate that the DoC server lost connectivity to the upstream DNS infrastructure.
The DoC client should handle this error case like a recursive resolver that lost connectivity to the upstream DNS infrastructure.
In case of CoAP errors, the usual mechanisms for CoAP response codes apply.

## DNS Extensions

DNS extensions that are specific to the choice of transport, such as described in {{?RFC7828}}, are not applicable to DoC.

--- back

Evaluation {#sec:evaluation}
==========
The authors of this document presented the design, implementation, and analysis of DoC in their
paper "Securing Name Resolution in the IoT: DNS over CoAP" {{DoC-paper}}.

<!-- [rfced] FYI: We removed the change log, which included a
reference to RFC 2136. If RFC 2136 should be mentioned elsewhere in
the running text, please let us know.
-->

<!--[rfced] We note that "draft-amsuess-core-cachable-oscore" is
expired and has been replaced by "draft-ietf-core-cacheable-oscore".
May we replace the current entry below with the entry for
"draft-ietf-core-cacheable-oscore"?

Current:
 [I-D.amsuess-core-cachable-oscore]
   Amsüss, C. and M. Tiloca, "Cacheable OSCORE", Work in Progress,
   Internet-Draft, draft-amsuess-core-cachable-oscore-11, 6 July 2025,
   <https://datatracker.ietf.org/doc/html/draft-amsuess-core-cachable-
   oscore-11>.

Perhaps:
 [CACHABLE-OSCORE]
    Amsüss, C. and M. Tiloca, "Cacheable OSCORE", Work in
    Progress, Internet-Draft, draft-ietf-core-cacheable-
    oscore-00, 22 September 2025,
    <https://datatracker.ietf.org/doc/html/draft-ietf-core-
    cacheable-oscore-00>.
-->

<!--[rfced] Sourcecode and artwork

a) Some lines in Figure 1 are too long for the TXT output. This figure is
marked as artwork, so it needs to have a width of 72 characters or less. How
may we revise this figure to fit these parameters? We tested removing some
space in the figure; please check out the following test files and let us know
if this would work (see TXT file for ascii art and HTML for SVG). If not, please
provide an updated figure.

Test files:
https://www.rfc-editor.org/authors/rfc9953test.md
https://www.rfc-editor.org/authors/rfc9953test.txt
https://www.rfc-editor.org/authors/rfc9953test.html


b) We have updated the blocks in Sections 3.2, 3.2.1, 4.2.3, and 4.3.3 to be
marked as sourcecode. We set the type for the block in Section 3.2 as "abnf"
(i.e., "~~~ abnf"). Please let us know if the type should be set for the other
sourcecode blocks. For example, should the ones in Section 3.2.1 be marked as
type "dns-rr"? If the current list of preferred values (see link below) does
not contain an applicable type, feel free to let us know. Also, it is
acceptable to leave the type not set.

List of sourcecode types:
https://www.rfc-editor.org/rpc/wiki/doku.php?id=sourcecode-types


c) The blocks in Section 4.3.3 are too long for the TXT output. We marked
these as sourcecode, so they should have a width of 69 characters or less. The
long lines are currently 70 characters. Would moving all the lines with
semicolons over to the left one space (in just this section or in all the
sourcecode in the document) be a good solution? We tried this in the test
files listed above so you can see what the output will look like. Feel free to
offer other suggestions as well.
-->

<!--[rfced] Please review the "Inclusive Language" portion of the online
Style Guide <https://www.rfc-editor.org/styleguide/part2/#inclusive_language>
and let us know if any changes are needed.  Updates of this nature typically
result in more precise language, which is helpful for readers.

Note that our script did not flag any words in particular, but this should
still be reviewed as a best practice.
-->

# Acknowledgments
{:unnumbered}

The authors of this document want to thank {{{Mike Bishop}}}, {{{Carsten Bormann}}}, {{{Mohamed Boucadair}}}, {{{Deb Cooley}}}, {{{Vladimír Čunát}}}, {{{Roman Danyliw}}}, {{{Elwyn B. Davies}}}, {{{Esko Dijk}}}, {{{Gorry Fairhurst}}}, {{{Thomas Fossati}}}, {{{Mikolai Gütschow}}}, {{{Todd Herr}}}, {{{Tommy Pauly}}}, {{{Jan Romann}}}, {{{Ben Schwartz}}}, {{{Orie Steele}}}, {{{Marco Tiloca}}}, {{{Éric Vyncke}}}, {{{Tim Wicinski}}}, and {{{Paul Wouters}}} for their feedback and comments.

[draft-ietf-core-dns-over-coap-18]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-18
[draft-ietf-core-dns-over-coap-17]: https://datatracker.ietf.org/doc/html/draft-ietf-core-dns-over-coap-17
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
