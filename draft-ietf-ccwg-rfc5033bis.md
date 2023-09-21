---
title: "Specifying New Congestion Control Algorithms"
abbrev: "New CC Algorithms"
docname: draft-ietf-ccwg-rfc5033bis-latest
category: bcp
stream: IETF
obsoletes: 5033

ipr: trust200902
area: General
workgroup: CCWG
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: M. Duke
    name: Martin Duke
    organization: Google LLC
    email: martin.h.duke@gmail.com
    role: editor
 -
    ins: G. Fairhurst
    name: Godred Fairhurst
    organization: University of Aberdeen
    email: gorry@erg.abdn.ac.uk
    role: editor
contributor:
 -
    ins: C. Huitema
    name: Christian Huitema
    organization: Private Octopus, Inc.
    email: huitema@huitema.net

normative:

informative:

  BBR-draft: I-D.cardwell-iccrg-bbr-congestion-control

  HRX08:
    title: "CUBIC: a new TCP-friendly high-speed TCP variant"
    seriesinfo: ACM SIGOPS Operating Systems Review, vol. 42, no. 5, pp. 64-74
    date: 2008-07
    target: https://doi.org/10.1145/1400097.1400105
    author:
      -
        ins: S. Ha
        name: Sangtae Ha
      -
        ins: I. Rhee
        name: Injong Rhee
      -
        ins: L. Xu
        name: Lisong Xu

  FJ03:
    title: Random Early Detection Gateways for Congestion Avoidance
    seriesinfo: IEEE/ACM Transactions on Networking, V.1 N.4
    date: 1993-8
    author:
    - ins: S. Floyd
    - ins: V. Jacobson

  Tools:
    title: Tools for the Evaluation of Simulation and Testbed Scenarios
    target: "https://datatracker.ietf.org/doc/draft-irtf-tmrg-tools"
    seriesinfo: Work in Progress
    date: 2007-7
    author:
    - ins: S. Floyd
    - ins: E. Kohler

  Bufferbloat:
    title: The Blind Men and the Elephant
    target: https://www.ietf.org/blog/blind-men-and-elephant/
    author:
      ins: J. Gettys
      name: Jim Gettys
    date: 2018-2-10
    seriesinfo: IETF Blog

--- abstract

The IETF's standard congestion control schemes have been widely shown
to be inadequate for various environments (e.g., high-speed networks,
wireless technologies such as 3GPP and WiFi, long distance satellite
links) and also in conflict with the needed, more isochronous,
behaviors of VoIP, gaming, and videoconferencing traffic.
Recent research has yielded many alternate congestion
control schemes that significantly differ from the IETF's congestion
control principles.
Using these new congestion control schemes in
the global Internet has possible ramifications to both the traffic
using the new congestion control and to traffic using the currently
standardized congestion control.
Therefore, the IETF must proceed
with caution when dealing with alternate congestion control
proposals.
The goal of this document is to provide guidance for
considering alternate congestion control algorithms within the IETF.

--- middle

# Introduction

This document provides guidelines for the IETF to use when evaluating
suggested congestion control algorithms that significantly differ
from the general congestion control principles outlined in {{!RFC2914}}.
The guidance is intended to be useful to authors proposing alternate
congestion control and for the IETF community when evaluating whether
a proposal is appropriate for publication in the RFC series and for
deployment in the Internet.

This document updates the similarly titled {{!RFC5033}} that was
published in 2007. Since then, multiple congestion control algorithms
were developed outside of the IETF, including at least two that saw
large scale deployment: Cubic {{HRX08}} and BBR {{BBR-draft}}.

Cubic was documented in a research publication in 2007 {{HRX08}},
and then adopted as the default congestion control algorithm for
the TCP implementation in Linux. It was already used in a significant
fraction of TCP connections over the Internet before being documented
in an informational Internet Draft in 2015, being published as an
informational RFC in 2017 {{?RFC8312}} and then as a proposed
standard in 2023 {{?RFC9438}}.

BBR is developed as an internal research project by Google,
with the first implementation contributed to Linux kernel 4.19 in 2016.
It was described in an IRTF draft in 2018, and that draft is
regularly updated to document the evolving versions of the algorithm
{{BBR-draft}}. BBR is widely used for Google services using either
TCP or QUIC {{?RFC9000}}, and is also largely deployed outside of
Google.

We cannot say now whether the original authors of {{?RFC5033}}
expected that developers would be somehow waiting for IETF review
before widely deploying a congestion control algorithm over the
Internet, but the examples of Cubic and BBR teaches us that
deployment of new algorithms is not in fact gated by publication
of the algorithm as an RFC. Nevertheless, guidelines are
important, if only to remind potential inventors and developers of
the multiple facets of the congestion control problem.

The guidelines in this document are intended to be consistent with
the congestion control principles from {{!RFC2914}} of preventing
congestion collapse, considering fairness, and optimizing the flow's
own performance in terms of throughput, delay, and loss.
{{!RFC2914}}
also discusses the goal of avoiding a congestion control "arms race"
among competing transport protocols.

This document does not give hard-and-fast requirements for an
appropriate congestion control scheme.
Rather, the document provides
a set of criteria that should be considered and weighed by the
developers of congestion control algorithms and by the IETF
in the context of each proposal.
The high-order criteria for any new
proposal is that a serious scientific study of the pros and cons of
the proposal needs to have been done before a proposal is
considered for publication by the IETF or before it is deployed at
large scale.

After initial studies, we encourage authors to write a specification
of their proposals for publication in the RFC series to allow others
to concretely understand and investigate the wealth of proposals in
this space.

# Document Status

Following the lead of HighSpeed TCP {{?RFC3649}}, alternate congestion
control algorithms are expected to be published as "Experimental"
RFCs until such time that the community better understands the
solution space.
Traditionally, the meaning of "Experimental" status
has varied in its use and interpretation.
As part of this document
we define two classes of congestion control proposals that can be
published with the "Experimental" status.
The first class includes
algorithms that are judged to be safe to deploy for best-effort
traffic in the global Internet and further investigated in that
environment.
The second class includes algorithms that, while
promising, are not deemed safe enough for widespread deployment as
best-effort traffic on the Internet, but are being specified to
facilitate investigations in simulation, testbeds, or controlled
environments.
The second class can also include algorithms where the
IETF does not yet have sufficient understanding to decide if the
algorithm is or is not safe for deployment on the Internet.

Each alternate congestion control algorithm published is required to
include a statement in the abstract indicating whether or not the
proposal is considered safe for use on the Internet.
Each alternate
congestion control algorithm published is also required to include a
statement in the abstract describing environments where the protocol
is not recommended for deployment.
There may be environments where
the protocol is deemed *safe* for use, but still is not *recommended*
for use because it does not perform well for the user.

As examples of such statements, {{?RFC3649}} specifying HighSpeed TCP
includes a statement in the abstract stating that the proposal is
Experimental, but may be deployed in the current Internet.  In
contrast, the Quick-Start document {{?RFC4782}} includes a paragraph in
the abstract stating the mechanism is only being proposed for
controlled environments.  The abstract specifies environments where
the Quick-Start request could give false positives (and therefore
would be unsafe to deploy).  The abstract also specifies environments
where packets containing the Quick-Start request could be dropped in
the network; in such an environment, Quick-Start would not be unsafe
to deploy, but deployment would still not be recommended because it
could cause unnecessary delays for the connections attempting to use
Quick-Start.

For authors of alternate congestion control schemes who are not ready
to bring their congestion control mechanisms to the IETF for
standardization (either as Experimental or as Proposed Standard), one
possibility would be to submit an internet-draft that documents the
alternate congestion control mechanism for the benefit of the IETF
and IRTF communities.  This is particularly encouraged in order to
get algorithm specifications widely disseminated to facilitate
further research.  Such an internet-draft could be submitted to be
considered as an Informational RFC, as a first step in the process
towards standardization.  Such a document would also be expected to
carry an explicit warning against using the scheme in the global
Internet.

Note: we are not changing the RFC publication process for non-IETF
produced documents (e.g., those from the IRTF or Independent
Submissions via the RFC-Editor).  However, we would hope the
guidelines in this document inform the IESG as they consider whether
to add a note to such documents.

# Guidelines

As noted above, authors are expected to do a well-rounded evaluation
of the pros and cons of proposals brought to the IETF.  The following
are guidelines to help authors and the IETF community.  Concerns that
fall outside the scope of these guidelines are certainly possible;
these guidelines should not be considered as an all-encompassing
check-list.

(0)
: Differences with Congestion Control Principles {{!RFC2914}}

: Proposed congestion control mechanisms should include a clear
explanation of the deviations from {{!RFC2914}}.

(1)
: Impact on Standard TCP, SCTP {{!RFC2960}}, and DCCP {{!RFC4340}}.

: Evaluation of proposed congestion control mechanisms should include test cases
competing with standard IETF congestion control {{!RFC2581}},
{{!RFC2960}}, {{!RFC4340}}.  Alternate congestion controllers that have a
significantly negative impact on traffic using standard
congestion control may be suspect and this aspect should be part
of the community's decision making with regards to the
suitability of the alternate congestion control mechanism.

: We note that this bullet is not a requirement for strict TCP-
friendliness as a prerequisite for an alternate congestion
control mechanism to advance to Experimental.  As an example,
HighSpeed TCP is a congestion control mechanism that is
Experimental, but that is not TCP-friendly in all environments.
We also note that this guideline does not constrain the fairness
offered for non-best-effort traffic.

: As an example from an Experimental RFC, fairness with standard
TCP is discussed in Sections 4 and 6 of {{?RFC3649}} (HighSpeed TCP)
and using spare capacity is discussed in Sections 6, 11.1, and 12
of {{?RFC3649}}.

(2)
: Wireless links

: While the early Internet was dominated by wired links, the properties
of wireless links have become extremely important to Internet performance.
In particular, congestion controllers should be evaluated in situations
where some packet losses are due to radio effects, rather than router
queue drops; the link capacity varies over time due to changing link conditions;
and media access delays and link-layer retransmission lead to increased jitter
in round-trip times. See {{?RFC3819}} and Section 16 of {{Tools}} for further
discussion of wireless properties.

(3)
: Difficult Environments.

: The proposed algorithms should be assessed in difficult
environments.  We note that there is still much to be desired in
terms of the performance of TCP in some of these difficult
environments.  For congestion control mechanisms with explicit
feedback from routers, difficult environments can include paths
with non-IP queues at layer-two, IP tunnels, and the like.  A
minimum goal for experimental mechanisms proposed for widespread
deployment in the Internet should be that they do not perform
significantly worse than TCP in these environments.

: While it is impossible to enumerate all the possible "difficult
environments", we note that the IETF has previously grappled with
paths with long delays {{?RFC2488}}, high delay bandwidth products
{{?RFC3649}}, high packet corruption rates {{?RFC3155}}, packet
reordering {{?RFC4653}}, and significantly slow links {{?RFC3150}}.
Aspects of alternate congestion control that impact networks with
these characteristics should be detailed.

: As an example from an Experimental RFC, performance in difficult
environments is discussed in Sections 6, 9.2, and 10.2 of
{{?RFC4782}} (Quick-Start).

(4)
: Investigating a Range of Environments.

: Similar to the last criteria, proposed alternate congestion
controllers should be assessed in a range of environments.  For
instance, proposals should be investigated across a range of
capacities, round-trip times, levels of traffic on the reverse
path, and levels of statistical multiplexing at the congested
link.  Similarly, proposals should be investigated for robust
performance with different queueing mechanisms in the routers,
especially Random Early Detection (RED) {{FJ03}} and Drop-Tail.
This evaluation is often not included in the internet-draft
itself, but in related papers cited in the draft.

: A particularly important aspect of evaluating a proposal for
standardization is in understanding where the algorithm breaks
down.  Therefore, particular attention should be paid to
characterizing the areas where the proposed mechanism does not
perform well.

: As an example from an Experimental RFC, performance in a range of
environments is discussed in Section&nbsp;12 of {{?RFC3649}} (HighSpeed
TCP) and Section&nbsp;9.7 of {{?RFC4782}} (Quick-Start).

(5)
: Protection Against Congestion Collapse

: The alternate congestion control mechanism should either stop
sending when the packet drop rate exceeds some threshold
{{?RFC3714}}, or should include some notion of "full backoff".  For
"full backoff", at some point the algorithm would reduce the
sending rate to one packet per round-trip time and then
exponentially backoff the time between single packet
transmissions if congestion persists.  Exactly when either "full
backoff" or a pause in sending comes into play will be
algorithm-specific.  However, as discussed in {{!RFC2914}}, this
requirement is crucial to protect the network in times of extreme
congestion.

: If "full backoff" is used, this bullet does not require that the
full backoff mechanism must be identical to that of TCP
{{?RFC2988}}.  As an example, this bullet does not preclude full
backoff mechanisms that would give flows with different round-
trip times comparable caapcity during backoff.

(6)
: Protection Against Bufferbloat

: The alternate congestion control mechanism should reduce its sending
rate if the round trip time (RTT) significantly increases. Exactly how
the algorithm reduces its sending rate is algorithm specific.

: Bufferbloat {{Bufferbloat}} refers to the building of long queues in
the network. Many network routers are configured with very large buffers.
If congestion starts happening, classic TCP congestion control algorithms
{{!RFC5681}} will continue sending at a high rate until the buffer fills
up completely and packet losses occur. Every connection going through
that bottleneck will experience high latency.
This is particularly bad for highly interactive applications like games,
but it also affects routine web browsing and video playing.

: This problem became apparent in the last decade and was not discussed in
the Congestion Control Principles published in September 2002 {{!RFC2914}}.
The classic congestion control algorithm {{!RFC5681}} and the widely deployed
Cubic algorithm {{?RFC9438}} do not address it, but newly designed congestion
control algorithms have the opportunity to improve the state of the art.

(7)
: Fairness within the Alternate Congestion Control Algorithm.

: In environments with multiple competing flows all using the same
alternate congestion control algorithm, the proposal should
explore how the capacity is shared among the competing flows.

(8)
: Performance with Misbehaving Nodes and Outside Attackers.

: The proposal should explore how the alternate congestion control
mechanism performs with misbehaving senders, receivers, or
routers.  In addition, the proposal should explore how the
alternate congestion control mechanism performs with outside
attackers.  This can be particularly important for congestion
control mechanisms that involve explicit feedback from routers
along the path.

: As an example from an Experimental RFC, performance with
misbehaving nodes and outside attackers is discussed in Sections
9.4, 9.5, and 9.6 of {{?RFC4782}} (Quick-Start).  This includes
discussion of misbehaving senders and receivers; collusion
between misbehaving routers; misbehaving middleboxes; and the
potential use of Quick-Start to attack routers or to tie up
available Quick-Start bandwidth.

(9)
: Responses to Sudden or Transient Events.

: The proposal should consider how the alternate congestion control
mechanism would perform in the presence of transient events such
as sudden congestion, a routing change, or a mobility event.
Routing changes, link disconnections, intermittent link
connectivity, and mobility are discussed in more detail in
Section 17 of {{Tools}}.

: As an example from an Experimental RFC, response to transient
events is discussed in Section&nbsp;9.2 of {{?RFC4782}} (Quick-Start).

(10)
: Incremental Deployment.

: The proposal should discuss whether the alternate congestion
control mechanism allows for incremental deployment in the
targeted environment.  For a mechanism targeted for deployment in
the current Internet, it would be helpful for the proposal to
discuss what is known (if anything) about the correct operation
of the mechanism with some of the equipment installed in the
current Internet, e.g., routers, transparent proxies, WAN
optimizers, intrusion detection systems, home routers, and the
like.

: As a similar concern, if the alternate congestion control
mechanism is intended only for specific environments (and not the
global Internet), the proposal should consider how this intention
is to be carried out.  The community will have to address the
question of whether the scope can be enforced by simply stating
the restrictions or whether additional protocol mechanisms are
required to enforce the scoping.  The answer will necessarily
depend on the change being proposed.

: As an example from an Experimental RFC, deployment issues are
discussed in Sections 10.3 and 10.4 of {{?RFC4782}} (Quick-Start).

# Minimum Requirements

This section suggests minimum requirements for a document to be
approved as Experimental with approval for widespread deployment in
the global Internet.

The minimum requirements for approval for widespread deployment in
the global Internet include the following guidelines on: (1)
assessing the impact on standard congestion control, (2) performance in
wireless environments, (4)
investigation of the proposed mechanism in a range of environments,
(5) protection against congestion collapse, and (10) discussing
whether the mechanism allows for incremental deployment.

For other guidelines, the author must
perform the suggested evaluations and provide recommended analysis.
Evidence that the proposed mechanism has significantly more problems
than those of TCP should be a cause for concern in approval for
widespread deployment in the global Internet.

# Security Considerations

This document does not represent a change to any aspect of the TCP/IP
protocol suite and therefore does not directly impact Internet
security.  The implementation of various facets of the Internet's
current congestion control algorithms do have security implications
(e.g., as outlined in {{!RFC2581}}).  Alternate congestion control
schemes should be mindful of such pitfalls, as well, and should
examine any potential security issues that may arise.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

Sally Floyd and Mark Allman were the authors of this document's predecessor,
RFC5033, which served the community well for over a decade.

Thanks to Richard Scheffenegger for helping to get this revision process started.

Discussions with Lars Eggert and Aaron Falk seeded the original RFC5033.
Bob Briscoe, Gorry Fairhurst, Doug Leith, Jitendra Padhye,
Colin Perkins, Pekka Savola, members of TSVWG, and participants at
the TCP Workshop at Microsoft Research all provided feedback and
contributions to that document.  It also drew from {{?RFC5166}}.

These individuals suggested improvements to this document:

<ul spacing="compact">
<li><t><contact fullname="Dave Taht"/></t></li>

</ul>

# Evolution of RFC5033bis
{:numbered="false"}

## Since draft-scheffenegger-congress-rfc5033bis-00
{:numbered="false"}

- Renamed file to reflect WG adpotion
- Updated authorship and acknowledgements.
- Include updated text suggested by Dave Taht
- Added criterion for bufferbloat
- Added wireless environments
- Mentioned Cubic and BBR as motivation
- Include section to track updates between revisions
- Update references

## Since RFC5033
{:numbered="false"}

- converted to Markdown and xml2rfc v3
- various formatting changes

