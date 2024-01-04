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

venue:
  group: Congestion Control Working Group (ccwg)
  mail: ccwg@ietf.org
  github: ietf-wg-ccwg/rfc5033bis


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
suggested congestion control algorithms that differ
from the general congestion control principles outlined in {{!RFC2914}}.
The guidance is intended to be useful to authors proposing alternate
congestion control and for the IETF community when evaluating whether
a proposal is appropriate for publication in the RFC series and for
deployment in the Internet.

This document updates the similarly titled {{!RFC5033}} that was
published in 2007 as a Best Current Practice to evaluate new
congestion control algorithms as Experimental or Proposed Standard RFCs.

In 2007, TCP was the dominant consumer of this work, and proposals were
typically discussed in research groups, for example the
Internet Congestion Control Research Group (ICCRG).

Since RFC 5033 was published, many conditions have changed.
The set of protocols using these algorithms has spread beyond
TCP and SCTP to include DCCP, QUIC, and beyond.
Some congestion control algorithm proponents now have the opportunity
to test and deploy at scale without IETF review.
There is more interest in specialized use cases such as data centers, and in
support for a variety of upper layer protocols/applications, e.g.,
real-time protocols.
Finally, the community has gained much more experience with indications
of congestion beyond packet loss.

Section 4 of the UDP Usage Guidelines {{RFC8085}} provide
additional guidelines for multicast and broadcast usage of UDP.
The evaluation considerations described in the present document could be
applicable to these congestion control algorithms, but this is not 
the intended scope of this document.

Multiple congestion control algorithms
have been developed outside of the IETF, including at least two that saw
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

This document is meant to reduce the barriers to entry for new congestion
control work. As such, proponents should not interpret these criteria as a
checklist of requirements before approaching the IETF. Instead, proponents
are encouraged to think about these issues beforehand, and have the willingness
to do the work implied by the rest of this document.

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
promising, are not yet deemed safe enough for widespread deployment as
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
There can be environments where
the protocol is deemed *safe* for use, but it is still is not *recommended*
for use because it does not perform well for the user.

As examples of such statements, {{?RFC3649}} specifying HighSpeed TCP
includes a statement in the abstract stating that the proposal is
Experimental, but may be deployed in the current Internet.  In
contrast, the Quick-Start document {{?RFC4782}} includes a paragraph in
the abstract stating the mechanism is only being proposed for
controlled environments.  The abstract specifies environments where
the Quick-Start request could give false positives (and therefore
would be unsafe for incremental deployment where some routers
forward, but do not process the option).  The abstract also specifies environments
where packets containing the Quick-Start request could be dropped in
the network; in such an environment, Quick-Start would not be unsafe
to deploy, but deployment would not be recommended because it
could lead to unnecessary delays for the connections attempting to use
Quick-Start. The Quick-Start method is discussed as an example in {{?RFC9049}}.

For authors of alternate congestion control schemes who are not ready
to bring their congestion control mechanisms to the IETF for
standardization (either as Experimental or as Proposed Standard), one
possibility would be to submit an internet-draft that documents the
alternate congestion control mechanism for the benefit of the IETF
and IRTF communities.  This is particularly encouraged in order to
ensure algorithm specifications are widely disseminated to facilitate
further research.  Such an internet-draft could also be
considered for publication as an Informational RFC, as a first step in the process
towards standardization.  Such a document would be expected to
carry an explicit warning against using the scheme in the global
Internet.

Note: we are not changing the RFC publication process for non-IETF
produced documents (e.g., those from the IRTF or Independent
Submissions via the RFC-Editor).  However, we would hope the
guidelines in this document inform the IESG as they consider whether
to add a note to such documents.

# Evaluation Criteria {#evaluation-criteria}

As noted above, authors are expected to do a well-rounded evaluation
of the pros and cons of proposals brought to the IETF.  The following
are guidelines to help authors and the IETF community.  Concerns that
fall outside the scope of these guidelines are certainly possible;
these guidelines should not be considered as an all-encompassing
check-list.

When considering a new congestion control proposal, the community MUST
consider the following criteria. These criteria will be evaluated in various
domains (see {{general-use}} and {{special-cases}}).

## Single Algorithm Behavior

The following criteria evaluate the proposed algorithm when one or more
flows using that algorithm share a bottleneck link, with no other algorithms
operating.

### Protection Against Congestion Collapse

The alternate congestion control mechanism should either stop
sending when the packet drop rate exceeds some threshold
{{?RFC3714}}, or should include some notion of "full backoff".  For
"full backoff", at some point the algorithm would reduce the
sending rate to one packet per round-trip time and then
exponentially backoff the time between single packet
transmissions if congestion persists.  Exactly when either "full
backoff" or a pause in sending comes into play will be
algorithm-specific.  However, as discussed in {{!RFC2914}} and {{!RFC8961}}, this
requirement is crucial to protect the network in times of extreme
congestion.

If the result of full backoff is used, this test does not require that the
full backoff mechanism must be identical to that of TCP
{{?RFC2988}} {{!RFC8961}}.  As an example, this does not preclude full
backoff mechanisms that would give flows with different round-
trip times comparable capacity during backoff.

### Protection Against Bufferbloat

The alternate congestion control mechanism should reduce its sending
rate if the round trip time (RTT) significantly increases. Exactly how
the algorithm reduces its sending rate is algorithm specific, but see
{{!RFC8961}} and {{!RFC8085}} for requirements.

Bufferbloat {{Bufferbloat}} refers to the building of long queues in
the network. Many network routers are configured with very large buffers.
If congestion starts happening, classic TCP congestion control algorithms
{{!RFC5681}} will continue sending at a high rate until the buffer fills
up completely and packet losses occur. Every connection going through
that bottleneck will experience high latency.  This adds unwanted latency that
impacts highly interactive applications like games, but it also affects routine
web browsing and video playing.

This problem became apparent in the last decade and was not discussed in
the Congestion Control Principles published in September 2002 {{!RFC2914}}.
The classic congestion control algorithm {{!RFC5681}} and the widely deployed
Cubic algorithm {{?RFC9438}} do not address it, but newly designed congestion
control algorithms have the opportunity to improve the state of the art.

### Fairness within the Alternate Congestion Control Algorithm.

When multiple competing flows all use the same
alternate congestion control algorithm, the proposal should
explore how the capacity is shared among the competing flows.
Capacity fairness can be important when a small number of similar
flows compete to fill a bottleneck. It can however also not be useful:
for example when comparing flows seek to send at different rates or
when some of the flows do not last sufficiently long to approach
asymptotic behavior.

### Avoiding Starvation of other Flows.

In contexts where differing congestion control
algorithms are used, it is important to understand whether
an alternate congestion control algorithm can induce more
harm to sharing flows than existing
defined methods. The measure of harm is not restricted to the equality
of capacity, but ought also to consider metrics such as the
latency introduced, or an increase in packet loss. This evaluation must
assess the potential to cause starvation, including assurance that
a loss of all feedback (e.g., detected by expiry of a retransmission time out)
results in backoff.

### Short Flows

A great deal of congestion control analysis concerns the steady-state behavior
of long flows. However, many internet flows are relatively short-lived. If they
never experience a packet loss, they remain in the "slow start" mode of
operation {{?RFC5681}} that features exponential congestion window growth.

Proposals will consider how new and short-lived flows affect long-lived flows,
and vice versa.

## Mixed Algorithm Behavior

These criteria evaluate the interaction of the proposal with commonly deployed
congestion controls.

### Existing General-Purpose Transports

Evaluate the impact on TCP {{!RFC9293}}, SCTP {{!RFC9260}}, DCCP {{!RFC4340}},
and QUIC {{!RFC9000}}.

Proposed congestion control mechanisms should be evaluated when
competing with standard IETF congestion control {{!RFC5681}},
{{!RFC9260}}, {{!RFC4340}}, {{!RFC9002}}, {{!RFC9438}}.  Alternate
congestion controllers that have a significantly negative impact on
traffic using standard congestion control may be suspect and this aspect should
be part of the community's decision making with regards to the suitability of
the alternate congestion control mechanism. The community should also consider
other non-standard congestion controls known to be widely deployed,

We note that this bullet is not a requirement for strict Reno- or Cubic-
friendliness as a prerequisite for an alternate congestion
control mechanism to advance to Experimental.  As an example,
HighSpeed TCP is a congestion control mechanism that is
Experimental, but that is not TCP-friendly in all environments.
When a new algorithm is deployed, the existing major deployments need to be
considered to avoid severe performance degradation.
We also note that this guideline does not constrain the interaction with
non-best-effort traffic.

As an example from an Experimental RFC, fairness with standard TCP is discussed
in Sections 4 and 6 of {{?RFC3649}} (HighSpeed TCP) and using spare capacity is
discussed in Sections 6, 11.1, and 12 of {{?RFC3649}}.

### Real-Time Protocols

General-purpose protocols coexist in the Internet with real-time congestion
control protocols, which in general have finite throughput requirements (i.e.
they do not seek to utilize all available capacity) and more strict latency
bounds.

{{?RFC8868}} provides suggestions for real-time congestion control design and
{{?RFC8867}} suggests test cases. {{?RFC9392}} describes some considerations
for the RTP Control Protocol (RTCP). In particular, feedback for real-time
flows can be less frequent than the acknowledgements provided by reliable
transports. This document does not change the informational status of those
RFCs.

New proposals SHOULD consider coexistence with widely deployed real-time
congestion controls. Regrettably, at the time of writing, many algorithms with
detailed public specifications are not widely deployed, while many widely
deployed real-time congestion controls have incomplete public specifications.

To the extent that behavior of widely deployed algorithms is understood, proposals
can analyze and simulate the their interaction with those algorithms. To the
extent they are not, experiments can be conducted where possible.

Note that in many deployments, real-time traffic is directed into distinct
queues via Differentiated Services Code Points (DSCP) or other mechanisms,
which substantially reduces the interplay with other traffic. However, proposals
cannot assume that this is always the case.

### Short and Long Flows

(TODO: Discuss short and long flows)

## Other Criteria

### Differences with Congestion Control Principles

Proposed congestion control mechanisms SHOULD include a clear
explanation of the deviations from {{!RFC2914}} and {{!RFC7141}}.

### Incremental Deployment.

The proposal should discuss whether the alternate congestion
control mechanism allows for incremental deployment in the
targeted environment.  For a mechanism targeted for deployment in
the current Internet, it would be helpful for the proposal to
discuss what is known (if anything) about the correct operation
of the mechanism with some of the equipment installed in the
current Internet, e.g., routers, transparent proxies, WAN
optimizers, intrusion detection systems, home routers, and the
like.

As a similar concern, if the alternate congestion control
mechanism is intended only for specific environments (and not the
global Internet), the proposal should consider how this intention
is to be carried out.  The community will have to address the
question of whether the scope can be enforced by simply stating
the restrictions or whether additional protocol mechanisms are
required to enforce the scoping.  The answer will necessarily
depend on the change being proposed.

As an example from an Experimental RFC, deployment issues are
discussed in Sections 10.3 and 10.4 of {{?RFC4782}} (Quick-Start).

# General Use {#general-use}

The criteria in {{evaluation-criteria}} will be evaluated in the
following scenarios. Unless a proposal is explicitly forbidden on the
public internet, the community MUST find that it meets the criteria
in these scenarios for the proposal to progress.

The evaluation in each scenario should occur over a representative range of
bandwidths, delays, and queue depths. Of course, the set of parameters
representative of the public internet will change over time.

These criteria are meant to capture a statistically dominant set of internet
conditions. In the case that the algorithm has been tested at internet scale,
the results from that deployment are often useful for answering these questions.

## Wired Networks

(TODO: Describe properties of wired networks.)

Proposals should be investigated for robust performance with different
queueing mechanisms in the routers,
especially Random Early Detection (RED) {{FJ03}} and Drop-Tail.
This evaluation is often not included in the internet-draft
itself, but in related papers cited in the draft.

## Wireless networks

While the early Internet was dominated by wired links, the properties
of wireless links have become extremely important to Internet performance.
In particular, congestion controllers should be evaluated in situations
where some packet losses are due to radio effects, rather than router
queue drops; the link capacity varies over time due to changing link conditions;
and media access delays and link-layer retransmission lead to increased jitter
in round-trip times. See {{?RFC3819}} and Section 16 of {{Tools}} for further
discussion of wireless properties.

## Tail-drop queues

Congestion control performance is affected by the queue discipline applied at
the bottleneck link. The default queue discipline that MUST be evaluated is
drop-tail, First In First Out (FIFO). See {{aqm}} for evaluation of other queue
disciplines.

# Special Cases {#special-cases}

The criteria in {{evaluation-criteria}} will be evaluated in the
following scenarios, unless the proposal specifically excludes its use in a
scenario. The community MAY allow a proposal to progress even if the criteria
indicate an unsatisfactory result for these scenarios.

In general, measurements from internet-scale deployments will not expose the
properties of operation in these scenarios, as they are statistically small.

## Active Queue Management (AQM) {#aqm}

Proposals SHOULD be evaluated under a variety of bottleneck queue disciplines.
At a minimum, a proposal should reason about an algorithm's response to various
AQMs. Simulation or empirical results are, of course, valuable.

Note that evaluation of AQM techniques -- as opposed to their impact on specific
congestion control proposals -- is out of scope of this document. {{?RFC7567}}
describes design considerations for AQMs.

Among the AQM techniques that might have an impact on a congestion control
algorithm are FQ-CoDel {{?RFC8290}}; Proportional Integral Controller Enhanced
(PIE) {{?RFC8033}}; and Low Latency, Low Loss, and Scalable Throughput (L4S)
{{?RFC9332}}.

Congestion control algorithms that set one of the two Explicit Congestion Transport (ECT)
codepoints in the IP header can gain the benefits of receiving Explicit Congestion Notifictaion (ECN)
Congestion Experienced (CE) signals from an on-path AQM {{?RFC8087}}.
Use of ECN {{?RFC3168},{{?RFC9332}} results in requirements for
the congestion control algorithm to react when it receives a packet with an ECN-CE marking.
This reaction needs to be evaluated to confirm that the algorithm conforms with the
requirements of the ECT codepoint that was used.

## Internet of Things

The "Internet of Things" (IoT) is a broad concept, but for congestion control
purposes it is often associated with unique characteristics.

IoT nodes might be more constrained in power, CPU, or other parameters than
conventional Internet hosts. This might place limits on the complexity of any
given algorithm. These power and radio constraints might make the volume of
control packets in a given algorithm a key evaluation metric.

Furthermore, many IoT applications do not a have a human in the loop, and
therefore have weaker latency constraints because they do not relate to a user
experience.

Extremely low-power links can lead to very low throughput and a low bandwidth-
delay product, well below the standard operating range of most Internet flows.

## Satellite

Satellite links often have delays longer than typical for wired paths
{{?RFC2488}} and high delay bandwidth products {{?RFC3649}}.

## Misbehaving Nodes

The proposal should explore how the alternate congestion control
mechanism performs with non-compliant senders, receivers, or
routers.  In addition, the proposal should explore how the
alternate congestion control mechanism performs with outside
attackers.  This can be particularly important for congestion
control mechanisms that involve explicit feedback from routers
along the path.

As an example from an Experimental RFC, performance with
misbehaving nodes and outside attackers is discussed in Sections
9.4, 9.5, and 9.6 of {{?RFC4782}} (Quick-Start).  This includes
discussion of misbehaving senders and receivers; collusion
between misbehaving routers; misbehaving middleboxes; and the
potential use of Quick-Start to attack routers or to tie up
available Quick-Start bandwidth.

## Tunnel Behavior

When the proposal relies on explicit signals from the path, the effect of
traffic passing through the tunnel -- where routers may not be aware of the
underlying flow -- MUST be considered.

## Extreme Packet Reordering

{{?RFC4653}} discusses the effect of extreme packet reordering.

## Transient Events

The proposal should consider how the alternate congestion control
mechanism would perform in the presence of transient events such
as sudden congestion, a routing change, or a mobility event.
Routing changes, link disconnections, intermittent link
connectivity, and mobility are discussed in more detail in
Section 17 of {{Tools}}.

As an example from an Experimental RFC, response to transient
events is discussed in Section&nbsp;9.2 of {{?RFC4782}} (Quick-Start).

### Sudden changes in Path

An IETF transport is not tied to a specific Internet path or type of path.
The set of routers that form a path can and do change with time,
this will cause the properties of the path to change with respect to time.
New CCs MUST evaluate the impact of changes in the path, and be robust
to changes in path characteristics on the interval of common Internet re-routing intervals.

Even when the set of routers constituting a path does not change, the properties of
that path can vary with time (e.g., due to a change of link capacity, relative priority, or a change
in the rate of other traffic sharing a bottleneck), with similar impacts on congestion control.

## Multipath

Multipath transport protocols permit more than one path to be differentiated and used by
a single connection at the sender.
A multipath sender can schedule which packets travel on which of its active paths.
This enables a tradeoff in timeliness and reliability. There are various ways that
multipath techniques can be used,

One example use is to provide fail-over from one path to
another when the original path is no longer viable or to switch the traffic from
one path to another when this is expected to improve performance
(latency, throughput, reliability, cost).
Designs need to independently track the congestion state of each path,
and need to demonstrate independent congestion control for each path being used.
New multipath CCs that implement path fail-over MUST evaluate the harm resulting
from a change in the path, and show that this does not result in flow starvation.
Synchronisation of failover (e.g., where multiple flows change their path on similar
timeframes) can also contribute to harm and/or reduce fairness,
these effects also ought to be evaluated.

Another example use is concurrent multipath, where the transport protocol simultaneously
schedules flows to aggregate the capacity across multiple paths.
The Internet provides no guarantee that different paths
(e.g., using different endpoint addresses) are disjoint.
This has additional implications:
New CCs MUST evaluate the potential
harm to other flows when the multiple paths share a common
congested bottleneck
(or share resources that are coupled between different paths,
such as an overall capacity limit), and SHOULD consider
the potential for harm to other flows. Synchronisation of CC mechanisms
(e.g., where multiple flows change their behaviour on similar
timeframes) can also contribute to harm and/or reduce fairness,
these effects also ought to be evaluated.
At the time of writing, there are no IETF standards for concurrent
multipath congestion control in the general Internet.

# Security Considerations

This document does not represent a change to any aspect of the TCP/IP
protocol suite and therefore does not directly impact Internet
security.  The implementation of various facets of the Internet's
current congestion control algorithms do have security implications
(e.g., as outlined in {{!RFC5681}}).  Alternate congestion control
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

## Since draft-ietf-ccwg-rfc5033bis-02
{:numbered="false"}

- Added discussion of real-time protocols
- Added discussion of short flows
- Added IoT section
- Added discussion of AQM response
- Editorial changes

## Since draft-ietf-ccwg-rfc5033bis-01
{:numbered="false"}

- Added discussion of multipath transports
- Totally reorganized central sections of the draft

## Since draft-ietf-ccwg-rfc5033bis-00
{:numbered="false"}

- Added QUIC, other congestion control standards
- Added wireless environments
- Aligned motivation for this work with the CCWG charter
- Refined discussion of QuickStart

## Since draft-scheffenegger-congress-rfc5033bis-00
{:numbered="false"}

- Renamed file to reflect WG adpotion
- Updated authorship and acknowledgements.
- Include updated text suggested by Dave Taht
- Added criterion for bufferbloat
- Mentioned Cubic and BBR as motivation
- Include section to track updates between revisions
- Update references

## Since RFC5033
{:numbered="false"}

- converted to Markdown and xml2rfc v3
- various formatting changes

