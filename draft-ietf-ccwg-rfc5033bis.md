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

Introducing new or modified congestion controller algorithms in the global Internet have
possible ramifications to both the traffic using the new method
and to traffic using a standardized congestion control algorithm. Therefore, the IETF
must proceed with caution when evaluating proposals for alternate congestion control.
The goal of this document is to provide guidance for considering standardization
of an alternate congestion control algorithm at the IETF. It replaces RFC5033 to reflect
changes in the congestion control landscape.

--- middle

# Introduction

This document provides guidelines for the IETF to use when evaluating
a proposed congestion control algorithm that differ
from the general congestion control principles outlined in {{!RFC2914}}.
The guidance is intended to be useful to authors proposing alternate
congestion control algorithms and for the IETF community when evaluating whether
a proposal is appropriate for publication in the RFC series and for
deployment in the Internet.

This document obsoletes the similarly titled {{?RFC5033}} that was
published in 2007 as a Best Current Practice to evaluate alternate
congestion control algorithms as Experimental or Proposed Standard RFCs.

The IETF's standard congestion control algorithms have been shown to have
performance challenges in various environments
(e.g., high-speed networks, cellular and WiFi wireless technologies,
long distance satellite links)
and also for specific traffic workloads (VoIP, gaming, and videoconferencing).

In 2007, TCP was the dominant consumer of this work, and proposals were
typically discussed in research groups, for example the
Internet Congestion Control Research Group (ICCRG).

Since RFC 5033 was published, many conditions have changed.
The set of protocols using these algorithms has spread beyond
TCP and SCTP to include DCCP, QUIC, and beyond.
Some proponents of alternative congestion control algorithms now have the opportunity
to test and deploy at scale without IETF review.
There is more interest in specialized use cases, such as data centers, and in
support for a variety of upper layer protocols/applications, e.g.,
real-time protocols.
Finally, the community has gained much more experience with indications
of congestion beyond packet loss.

Multicast congestion control is a considerably less mature field of study
and are not in scope for this document.
However, Section 4 of the UDP Usage Guidelines {{RFC8085}} provide
additional guidelines for multicast and broadcast usage of UDP.

Congestion control algorithms
have been developed outside of the IETF, including at least two that saw
large scale deployment: Cubic {{HRX08}}
and Bottleneck Bandwidth and Round-trip propagation time (BBR) {{BBR-draft}}.

Cubic was documented in a research publication in 2007 {{HRX08}},
and was then adopted as the default congestion control algorithm for
the TCP implementation in Linux. It was already used in a significant
fraction of TCP connections over the Internet before being documented
in an Informational Internet Draft in 2015, published as an
Informational RFC in 2017 {{?RFC8312}} and then as a proposed
standard in 2023 {{?RFC9438}}.

BBR is developed as an internal research project by Google,
with the first implementation contributed to Linux kernel 4.19 in 2016.
It was described in an IRTF draft in 2018, and that draft is
regularly updated to document the evolving versions of the algorithm
{{BBR-draft}}. BBR is widely used for Google services using either
TCP or QUIC {{?RFC9000}}, and is also widely deployed outside of
Google.

We cannot say now whether the original authors of {{?RFC5033}}
expected that developers would be somehow waiting for IETF review
before widely deploying a new congestion control algorithm over the
Internet, but the examples of Cubic and BBR teach us that
deployment of new algorithms is not in fact gated by publication
of the algorithm as an RFC. Nevertheless, guidelines are
important, if only to remind potential inventors and developers of
the multiple facets of the congestion control problem.

The evaluation guidelines in this document are intended to be consistent with
the congestion control principles from {{!RFC2914}} of preventing
congestion collapse, considering fairness, and optimizing the flow's
own performance in terms of throughput, delay, and loss.
{{!RFC2914}}
also discusses the goal of avoiding a congestion control "arms race"
among competing transport protocols.

This document does not give hard-and-fast requirements for an
appropriate congestion control algorithm.
Rather, the document provides
a set of criteria that should be considered and weighed by the
developers of alternative algorithms and by the IETF
in the context of each proposal.

The high-order criteria for any proposal
is a serious scientific study of the pros and cons occurs when a proposal is
considered for publication by the IETF or before it is deployed at
large scale.

After initial studies, we encourage authors to write a specification
of their proposals for publication in the RFC series to allow others
to concretely understand and investigate the wealth of proposals in
this space.

This document is meant to reduce the barriers to entry for new congestion
control work to the IETF. As such, proponents ought not to interpret these criteria as a
checklist of requirements before approaching the IETF. Instead, proponents
are encouraged to think about these issues beforehand, and have the willingness
to do the work implied by the remainder of this document.

# Document Status

This document applies to proposals that seek Experimental or
Standards Track status. Evaluation of both cases involves the same questions,
but with different expectations for both the answers and the degree of
certainty it the answers.

Congestion control algorithms without experience of Internet-scale deployment SHOULD
seek Experimental status until real-world data is able to answer the questions
in {{general-use}}. Congestion control algorithms with a record of measured Internet-
scale deployment MAY directly seek the Standards Track if the community believes
it is safe, and the design is stable, guided by the considerations in
{{general-use}}. The exince of this data does not waive the other
considerations in this document.

Algorithms that are designed for special environments (e.g., data centers) and
forbidden from use in the Internet would, of course, instead seek real-world
data for those environments.

Experimental specifications SHOULD NOT be deployed as a default. They SHOULD
only be deployed in situations where they are being actively measured, and where
it is possible to deactivate if there are signs of pathological behavior.

Each published alternate congestion control algorithm is REQUIRED to include a
statement in the abstract indicating whether or not there is IETF consensus that
the proposal is considered safe for use on the Internet. Each published algorithm
is also required to include a statement in the
abstract describing environments where the protocol is not recommended
for deployment. There can be environments where the controller is deemed *safe*
for use, but it is still is not *recommended* for use because it does not
perform well for the user.

As examples of such statements, {{?RFC3649}} specifying HighSpeed TCP
includes a statement in the abstract stating that the proposal is
Experimental, but may be deployed in the current Internet.  In
contrast, the Quick-Start document {{?RFC4782}} includes a paragraph in
the abstract stating the mechanism is only being proposed for
controlled environments.  The abstract specifies environments where
the Quick-Start request could give false positives (and therefore
would be unsafe for incremental deployment where some routers
forward, but do not process the option). The abstract also specifies environments
where packets containing the Quick-Start request could be dropped in
the network; in such an environment, Quick-Start would not be unsafe
to deploy, but deployment would not be recommended because it
could lead to unnecessary delays for the connections attempting to use
Quick-Start. The Quick-Start method is discussed as an example in {{?RFC9049}}.

Though out of scope of this document, a proponent of a new
algorithm could alternatively
seek publication as an Informational or Experimental RFC via the Internet
Research Task Force (IRTF). In general, these proposals are expected to be less
mature than ones that follow the procedures in this document. Documentation of
deployed congestion control algorithms that cannot be changed by IETF or IRTF review
are invited to publish as an Informational RFC via the Independent Stream Editor
(ISE).

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

The following criteria evaluate the proposal when one or more flows
using that algorithm share a bottleneck link (i.e. with no flows
using a differing congestion controi algorithm).

### Protection Against Congestion Collapse

The alternate congestion control algorithm should either stop
sending when the packet drop rate exceeds some threshold
{{?RFC3714}}, or should include some notion of "full backoff".  For
"full backoff", at some point the algorithm would reduce the
sending rate to one packet per round-trip time and then
exponentially backoff the time between single packet
transmissions if the congestion persists.  Exactly when either "full
backoff" or a pause in sending comes into play will be
algorithm-specific.  However, as discussed in {{!RFC2914}} and {{!RFC8961}}, this
requirement is crucial to protect the network in times of extreme (persistent)
congestion.

If the result of full backoff is used, this test does not require that the
full backoff mechanism must be identical to that of TCP
{{?RFC2988}} {{!RFC8961}}.  As an example, this does not preclude full
backoff mechanisms that would give flows with different round-
trip times comparable capacity during backoff.

### Protection Against Bufferbloat

The alternate congestion control algorithm should reduce its sending
rate if the round trip time (RTT) significantly increases. Exactly how
the algorithm reduces its sending rate is algorithm-specific, but see
{{!RFC8961}} and {{!RFC8085}} for requirements.

Bufferbloat {{Bufferbloat}} refers to the building of long queues in
the network. Many network routers are configured with very large buffers.
If congestion is detected, classic TCP congestion control algorithms
{{!RFC5681}} will continue sending at a high rate until a First-In First-Out
(FIFO) buffer completely fills and packet losses then occur. Every connection pasing through
that bottleneck will then experience increased latency.  This adds unwanted latency that
impacts highly interactive applications like games, but it also affects routine
web browsing and video playing.

This problem became apparent in the last decade and was not discussed in
the Congestion Control Principles published in September 2002 {{!RFC2914}}.
The classic congestion control algorithm {{!RFC5681}} and the widely deployed
Cubic algorithm {{?RFC9438}} do not address it, but a newly designed
algorithm has the opportunity to improve the state of the art.

### Fairness within the Alternate Congestion Control Algorithm.

When multiple competing flows all use the same
alternate congestion control algorithm, the proposal should
explore how the capacity is shared among the competing flows.
Capacity fairness can be important when a small number of similar
flows compete to fill a bottleneck. It can however also not be useful,
for example, when comparing flows that seek to send at different rates or
when some of the flows do not last sufficiently long to approach
asymptotic behavior.

### Avoiding Starvation of Flows sharing a bottleneck.

In contexts where differing congestion control
algorithms are used, it is important to understand whether
a proposal can induce more
harm to sharing flows than for the existing
defined methods. The measure of harm is not restricted to the equality
of capacity, but ought also to consider metrics such as the
latency introduced, or an increase in packet loss. An evaluation must
assess the potential to cause starvation, including assurance that
a loss of all feedback (e.g., detected by expiry of a retransmission time out)
results in backoff.

### Short Flows {#short-flows}

A great deal of congestion control analysis concerns the steady-state behavior
of long flows. However, many Internet flows are relatively short-lived. If they
never experience a packet loss, a short-lived flow remains in the "slow start" mode of
operation {{?RFC5681}}, e.g., that features exponential congestion window growth.

A proposals for a new congestion control algorithm MUST consider how new and short-lived flows affect long-lived flows,
and vice versa.

## Mixed Algorithm Behavior

These criteria evaluate the interaction of the proposal with commonly deployed
congestion control algorithms.

### Existing General-Purpose Transports

Evaluate the impact on TCP {{!RFC9293}}, SCTP {{!RFC9260}}, DCCP {{!RFC4340}},
and QUIC {{!RFC9000}}.

A proposed congestion control algorithm SHOULD be evaluated when
competing with standard IETF congestion control {{!RFC5681}},
{{!RFC9260}}, {{!RFC4340}}, {{!RFC9002}}, {{!RFC9438}}.  A proposal
that has a significantly negative impact on
traffic using standard congestion control might be suspect and this aspect should
be part of the community's decision making with regards to the suitability of
the proposed congestion control algorithm. The community should also consider
other non-standard congestion control algorithms that are known to be widely deployed,

We note that this guideline is not a requirement for strict Reno- or Cubic-
friendliness as a prerequisite for an alternate congestion
control mechanism to advance to Experimental or Standards Track status.  As an example,
HighSpeed TCP is a congestion control mechanism specified as
Experimental, that is not TCP-friendly in all environments.
When a new algorithm is deployed, the existing major deployments need to be
considered to avoid severe performance degradation.
We also note that this guideline does not constrain the interaction with
non-best-effort traffic.

As an example from an Experimental RFC, fairness with standard TCP is discussed
in Sections 4 and 6 of {{?RFC3649}} (HighSpeed TCP) and using spare capacity is
discussed in Sections 6, 11.1, and 12 of {{?RFC3649}}.

### Real-Time Protocols

General-purpose protocols need to coexist in the Internet with real-time congestion
control algorithms, which, in general, have finite throughput requirements (i.e.
do not seek to utilize all available capacity) and more strict latency
bounds.

{{?RFC8868}} provides suggestions for real-time congestion control design and
{{?RFC8867}} suggests test cases. {{?RFC9392}} describes some considerations
for the RTP Control Protocol (RTCP). In particular, feedback for real-time
flows can be less frequent than the acknowledgements provided by reliable
transports. This document does not change the informational status of those
RFCs.

New proposals SHOULD consider coexistence with widely deployed real-time
congestion control algorithms. Regrettably, at the time of writing, many algorithms with
detailed public specifications are not widely deployed, while many widely
deployed real-time congestion control algorithms have incomplete public specifications.
It is hoped this situation will change.

To the extent that behavior of widely deployed algorithms is understood, proposals
can analyze and simulate their interaction with those algorithms. To the
extent they are not, experiments can be conducted where possible.

Note that in many deployments, real-time traffic is directed into distinct
queues via Differentiated Services Code Points (DSCP) or other mechanisms,
which substantially reduces the interplay with other traffic. However, a proposal
targeting Internet use
MUST NOT assume that all paths support specific mechanisms.

### Short and Long Flows

The effect on short-lived and long-lived flows using other common congestion
control algorithms MUST be evaluated, as in {{short-flows}}.

## Other Criteria

### Differences with Congestion Control Principles

Proposed congestion control algorithms SHOULD include a clear
explanation of any deviations from {{!RFC2914}} and {{!RFC7141}}.

### Incremental Deployment

The proposal ought to discuss whether the proposal
allows for incremental deployment in the
targeted environment.  For a mechanism targeted for deployment in
the current Internet, it would be helpful for a proposal to
discuss what is known (if anything) about the correct operation
of the mechanisms with some of the equipment that can be installed in the
current Internet, e.g., routers, transparent proxies, WAN
optimizers, intrusion detection systems, home routers, and the
like.

As a similar concern, if the proposal
is intended only for specific environments (and not the
global Internet), the proposal should consider how this intention
is to be realised.  The community will have to address the
question of whether the scope can be enforced by stating
the restrictions or whether additional protocol mechanisms are
required to enforce this scoping.  The answer will necessarily
depend on the change that is being proposed.

As an example from an Experimental RFC, deployment issues are
discussed in Sections 10.3 and 10.4 of {{?RFC4782}} (Quick-Start).

# General Use {#general-use}

The criteria in {{evaluation-criteria}} will be evaluated in the
following scenarios.
Unless a proposed congestion control algorithm
explicitly forbids use on the
public Internet, the community MUST find that it meets the criteria
in these scenarios for the proposal to progress.

The evaluation in each scenario should occur over a representative range of
bandwidths, delays, and queue depths. Of course, the set of parameters
representative of the public Internet will change over time.

These criteria are intended to capture a statistically dominant set of Internet
conditions. In the case that a proposed algorithm has been ted at Internet scale,
the results from that deployment are often useful for answering these questions.

## Tunnel Behavior

When a proposal relies on explicit signals from the path, proposals
MUST consider the effect of
traffic passing through a tunnel, where routers may not be aware of the
flow.

## Paths with Tail-drop Queues

The performance of a congestion control algorithm is affected by the queue discipline applied at
the bottleneck link. The drop-tail queue discipline (using a FIFO buffer) MUST be evaluated.
See {{aqm}} for evaluation of other queue
disciplines.

## Wired Paths

Wired networks are usually characterized by extremely low rates of packet loss except
for those due to queue drops. They tend to have stable aggregate bandwidth,
usually higher than other types of links, and low non-queueing delay. Because
the properties are relatively simple, wired links are typically used as a
"baseline" case even if they are not always the bottleneck link in the modern
Internet.

## Wireless Paths

While the early Internet was dominated by wired links, the properties
of wireless links have become extremely important to Internet performance.
In particular, a proposal should be evaluated in situations
where some packet losses are due to radio effects, rather than router
queue drops; the link capacity varies over time due to changing link conditions;
and media access delays and link-layer retransmission lead to increased jitter
in round-trip times. See {{?RFC3819}} and Section 16 of {{Tools}} for further
discussion of wireless properties.

# Special Cases {#special-cases}

The criteria in {{evaluation-criteria}} will be evaluated in the
following scenarios, unless the proposal specifically excludes its use in a
scenario. The community MAY allow a proposal to progress even if the criteria
indicate an unsatisfactory result for these scenarios.

In general, measurements from Internet-scale deployments will not expose the
properties of operation in these scenarios, as they are statistically small.

## Active Queue Management (AQM) {#aqm}

Proposals SHOULD be evaluated under a variety of bottleneck queue disciplines.
The effect of an AQM discipline can be hard to detect by Internet evaluation.
At a minimum, a proposal should reason about an algorithm's response to various
AQM disciplines. Simulation or empirical results are, of course, valuable.

Among the AQM techniques that might have an impact on a proposed congestion control
algorithm are FQ-CoDel {{?RFC8290}}; Proportional Integral Controller Enhanced
(PIE) {{?RFC8033}}; and Low Latency, Low Loss, and Scalable Throughput (L4S)
{{?RFC9332}}.

Congestion control proposals that set one of the two Explicit Congestion Transport (ECT)
codepoints in the IP header can gain the benefits of receiving Explicit Congestion Notifictaion (ECN)
Congestion Experienced (CE) signals from an on-path AQM {{?RFC8087}}.
Use of ECN {{?RFC3168}},{{?RFC9332}} results in requirements for
the congestion control algorithm to react when it receives a packet with an ECN-CE marking.
This reaction needs to be evaluated to confirm that the algorithm conforms with the
requirements of the ECT codepoint that was used.

Note that evaluation of AQM techniques -- as opposed to their impact on specific
congestion control proposals -- is out of scope of this document. {{?RFC7567}}
describes design considerations for AQMs.

## Paths with Varying Delay {#delay}

An Internet Path can include simple links, where the minimum delay is the propagation delay,
and any additional delay can be attributed to link buffering. This cannot be assumed.
An Internet Path can also include complex subnetworks where the minimum delay changes over
various time scales, resulting in a non-stationary minimum delay.

This occurs when a subnet changes the forwarding path to optimise capacity, resilience, etc.
It could also arise when a subnet uses a capacity management method where the available
resource is periodically distributed among the active nodes and where a node might then
have to buffer data until an assigned transmission opportunity or when the physical path
changes (e.g., when the length of a wireless path changes, or the physical layer changes
its mode of operation).
Variation also arises when a higher priority diffserv traffic classic prompts the
transmission by a lower class. In these cases, the delay varies as a function of
external factors and attempting to infer congestion from an increase in the delay
results in reduced throughput. The jitter from variation over short timescales
might not be distinguishable similar from other effects.

Congestion control proposals SHOULD be evaluated to ensure their operation is robust
when there is a significant change in the minimum delay.

## Internet of Things

The "Internet of Things" (IoT) is a broad concept, but when evaluating
a proposal,
it is often associated with unique characteristics.

IoT nodes might be more constrained in power, CPU, or other parameters than
conventional Internet hosts. This might place limits on the complexity of any
given algorithm. These power and radio constraints might make the volume of
control packets in a given algorithm a key evaluation metric.

Furthermore, many IoT applications do not a have a human in the loop, and
therefore can have weaker latency constraints because they do not relate to a user
experience, but still need to share the path with other flows with different
constraints.

Extremely low-power links can lead to very low throughput and a low bandwidth-
delay product, well below the standard operating range of most Internet flows.

## Paths with High Delay

A proposal ought not to presume that all general Internet paths have a low
delay.
Some paths include links that contibute much more delay than for a typical Internet path.
Satellite links often have delays longer than typical for wired paths
{{?RFC2488}} and high delay bandwidth products {{?RFC3649}}.
Also, many syms
use dynamic capacity assignment that can result in variation of the delay
and the capacity over timescales of the order of the path RTT.
Robustness to delay and delay variation may be a key evaluation metric.

## Misbehaving Nodes

The proposal should explore how the congestion control
proposal performs with non-compliant senders, receivers, or
routers.  In addition, the proposal should explore how an
alternate congestion control algorithm performs with outside
attackers.  This can be particularly important for proposals
that involve explicit feedback from routers
along the path.

As an example from an Experimental RFC, performance with
misbehaving nodes and outside attackers is discussed in Sections
9.4, 9.5, and 9.6 of {{?RFC4782}} (Quick-Start).  This includes
discussion of misbehaving senders and receivers; collusion
between misbehaving routers; misbehaving middleboxes; and the
potential use of Quick-Start to attack routers or to tie up
available Quick-Start bandwidth.

## Extreme Packet Reordering

A proposal ought not to presume that all general Internet paths reliably deliver
packets in order. {{?RFC4653}} discusses the effect of extreme packet reordering.

## Transient Events

The proposal SHOULD consider how the proposed congestion control
algorithm would perform in the presence of transient events such
as sudden onset of congestion, a routing change, or a mobility event.
Routing changes, link disconnections, intermittent link
connectivity, and mobility are discussed in more detail in
Section 17 of {{Tools}}.

As an example from an Experimental RFC, response to transient
events is discussed in Section&nbsp;9.2 of {{?RFC4782}} (Quick-Start).

### Sudden changes in the Path

An IETF transport is not tied to a specific Internet path or type of path.
The set of routers that form a path can and do change with time,
this will cause the properties of the path to change with respect to time.
New CCs MUST evaluate the impact of changes in the path, and be robust
to changes in path characteristics on the interval of common Internet re-routing intervals.

Even when the set of routers constituting a path does not change, the properties of
that path can vary with time (e.g., due to a change of link capacity, relative priority, or a change
in the rate of other traffic sharing a bottleneck), with a potential impact on the
operation of a congestion control algorithm.

## Multipath Transport

Multipath transport protocols permit more than one path to be differentiated and used by
a single connection at the sender.
A multipath sender can schedule which packets travel on which of its active paths.
This enables a tradeoff in timeliness and reliability. There are various ways that
multipath techniques can be used.

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
(e.g., as outlined in {{!RFC5681}}).

The IETF process that results in publication needs to
ensure that these security implications are considered.
Proposals therefore ought to be mindful of pitfalls, and should
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

The editors would like to thanks to Reese Enghardt and Dave Taht for suggesting improvements to this document.

# Evolution of RFC5033bis
{:numbered="false"}

## Since draft-ietf-ccwg-rfc5033bis-02
{:numbered="false"}

- Added discussion of real-time protocols
- Added discussion of short flows
- Listed properties of wired networks
- Added IoT section
- Added discussion of AQM response
- Rewrote the "Document Status" section
- Adding improved first sentence of abstract and intro.
- Added section on Multicast, noting this is out of scope
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

