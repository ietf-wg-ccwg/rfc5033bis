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

  BBRv1-draft: I-D.cardwell-iccrg-bbr-congestion-control-00

  ECN-Encaps: I-D.ietf-tsvwg-ecn-encap-guidelines

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

  Tools:
    title: Tools for the Evaluation of Simulation and Testbed Scenarios
    target: "https://datatracker.ietf.org/doc/draft-irtf-tmrg-tools"
    seriesinfo: Work in Progress
    date: 2007-7
    author:
    - ins: S. Floyd
    - ins: E. Kohler

  Bufferbloat:
    title: "Bufferbloat: Dark Buffers in the Internet"
    target: https://queue.acm.org/detail.cfm?id=2071893
    author:
      ins: Jim Gettys
    author:
      ins: Kathleen Nichols
    date: 2011
    seriesinfo: ACM Queue Volume 9, Issue 11

  BBRv1-Evaluation:
    title: "Experimental evaluation of BBR congestion control"
    target: https://ieeexplore.ieee.org/document/8117540
    author:
      ins: M. Hock
      ins: R. Bless
      ins: M. Zitterbart
    date: 2017
    seriesinfo: 2017 IEEE 25th International Conference on Network Protocols (ICNP)

--- abstract

This document replaces RFC 5033, which discusses the principles and
guidelines for standardzing new congestion control algorithms. It seeks to
ensure that proposed congestion control algorithms operate without harm and
efficiently alongside other algorithms in the global Internet. It emphasizes the
need for comprehensive testing and validation to prevent adverse interactions
with existing flows. This document provides a framework for the development and
assessment of congestion control mechanisms, promoting stability across diverse
network environments. It obsoletes RFC5033 to reflect changes in the congestion
control landscape.

--- middle

# Introduction

This document provides guidelines for the IETF to use when evaluating a proposed
congestion control algorithm that differs from the general congestion control
principles outlined in {{!RFC2914}}. The guidance is intended to be useful to
authors proposing congestion control algorithms and for the IETF community when
evaluating whether a proposal is appropriate for publication in the RFC series
and for deployment in the Internet.

This document obsoletes {{?RFC5033}}, which was published in 2007 as a Best
Current Practice for evaluating proposed congestion control algorithms as
Experimental or Proposed Standard RFCs.

The IETF specifies standard Internet congestion control algorithms in the RFC-series.
These congestion control algorithms can suffer performance challenges when used in
differing environments (e.g., high-speed networks, cellular and WiFi wireless
technologies, and long distance satellite links), and also when flows carry
specific workloads (Voice over IP (VoIP), gaming, and videoconferencing).

When [RFC5033] was published in 2007, TCP [RFC9293] was the primary focus of
IETF congestion control efforts, with proposals typically discussed within the
Internet Congestion Control Research Group (ICCRG). Concurrently, the Datagram
Congestion Control Protocol (DCCP) [RFC4340] was developed to define new
congestion control algorithms for datagram traffic, while the Stream Control
Transmission Protocol (SCTP) [RFC9260] reused TCP congestion control algorithms.

Since then, several changes have occurred. The range of protocols utilizing
congestion control algorithms has expanded to include QUIC [RFC9000] and RTP
Media Congestion Avoidance Techniques (RMCAT) (e.g., [RFC8836]. Additionally,
some alternative congestion control algorithms have been tested and deployed
at scale without full IETF review. There is increased interest in specialized
use cases, such as data centers (e.g., [RFC8257], and in supporting a variety of
upper layer protocols and applications, such as real-time protocols. Moreover,
the community has gained significant experience with congestion indications
beyond packet loss.

Multicast congestion control is a considerably less mature field of study and
is not in the scope of this document. However, {{Section 4 of ?RFC8085}} provides
additional guidelines for multicast and broadcast usage of UDP.

Congestion control algorithms have been developed outside of the IETF, including
at least two that saw large scale deployment: CUBIC {{HRX08}} and Bottleneck
Bandwidth and Round-trip propagation time (BBR) {{BBR-draft}}.

CUBIC was documented in a research publication in 2007 {{HRX08}}, and was then
adopted as the default congestion control algorithm for the TCP implementation
in Linux. It was already used in a significant fraction of TCP connections over
the Internet before being documented in an Informational Internet-Draft in
2015, published as an Informational RFC in 2017 as {{?RFC8312}} and then as a
Proposed Standard in 2023 {{?RFC9438}}.

At the time of writing, BBR is being developed as an internal research project
by Google, with the first implementation contributed to Linux kernel 4.19 in
2016. It was described in an IRTF Internet-Draft in 2018, and that Internet-
Draft is regularly updated to document the evolving versions of the algorithm
{{BBR-draft}}. BBR is currently widely used for Google services using either
TCP or QUIC, and is also widely deployed outside of Google.

We cannot say now whether the original authors of {{?RFC5033}} expected that
developers would be waiting for IETF review before widely deploying a
new congestion control algorithm over the Internet, but the examples of CUBIC
and BBR teach us that deployment of new algorithms is not, in fact, gated by the
publication of the algorithm as an RFC.

Nevertheless, a specification for a congestion control algorithm provides a number of
advantages:

- It can help implementers, operators, and other interested parties
  develop a shared understanding of how the algorithm works and how it is
  expected to behave in various scenarios and configurations.
- It can help potential contributors understand the algorithm,
  which can make it easier for them to suggest improvements and/or identify
  limitations. Furthermore, the specification can help multiple contributors
  align on a consensus change to the algorithm.
- A specification that is accessible to anyone can circumvent the issue that
  some implementers may be unable to read open source reference implementations
  due to the constraints of some open source licenses.

Beyond helping develop specific algorithm proposals, guidelines can also serve
as a reminder to potential inventors and developers of the multiple facets of
the congestion control problem.

The evaluation guidelines in this document are intended to be consistent with
the congestion control principles from {{!RFC2914}} of preventing congestion
collapse, considering fairness, and optimizing a flow's own performance in terms
of throughput, delay, and loss. {{!RFC2914}} also discusses the goal of avoiding
a congestion control "arms race" among competing transport protocols.

This document does not give hard-and-fast requirements for an appropriate
congestion control algorithm. Rather, the document provides a set of criteria
that should be considered and weighed by the developers of alternative
algorithms and by the IETF in the context of each proposal.

The high-order criterion for advancing any proposal within the IETF is a serious
scientific study of the pros and cons that occurs when the proposal is
considered for publication by the IETF, or before it is deployed at large scale.

After initial studies, authors are encouraged to write a specification of their
proposal for publication in the RFC series. This allows others to understand and
investigate the wealth of proposals in this space.

This document is intended to reduce the barriers to entry for new congestion
control work to the IETF. As such, proponents of new congestion control
algorithms ought not to interpret these criteria as a checklist of requirements
before approaching the IETF. Instead, proponents are encouraged to think about
these issues beforehand, and have the willingness to do the work implied by the
remainder of this document.

# Specification of Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Guidelines for Authors

## Guidelines for Authors about Evaluation

This document does not specify specific evaluation methods, short of internet-scale deployment and measurement, to test the criteria described below. There are multiple possible approaches to evaluation. Each has a role, and the most appropriate approach depends on the criteria being evaluated and the maturity of the specification.

For many algorithms, an initial evaluation will consider individual protocol mechanisms in a simulator to analyse their stability and safety across a wide range of conditions, including overload.  For example, {{?RFC8869}} describes evaluation test cases for interactive real-time media over wireless networks. Such results could also be published or discussed in IRTF research groups, such as ICCRG and MAPRG.

Before a proposed congestion control algorithm is published as an Experimental
or Standards Track RFC, the community SHOULD gain practical experience with
implementation and experience using the algorithm. Where there is implementation
by independent teams, this can help provide assurance that a specification has
avoided assumptions or ambiguity. An independent evaluation by multiple teams
helps provide assurance that the design meets the evaluation criteria, and can
assess typical interactions with other traffic. This evaluation could use an
emulated laboratory environment or a controlled experiment (within a limited
domain or at Internet-scale). Evidence of results is normally considered by the
working group in deciding if a specification is ready for publication and ought
to be documented in any request for the working group to publish the
specification.

Publication might occur without multiple implementations if a single
implementation is widely used, open source, and shown to have positive impact on
the Internet, particularly if the target status is Experimental.

## Guidelines for Authors about Document Status

This document applies to proposals for congestion control algorithms that
seek Experimental or Standards Track status. Evaluation of both cases involves
the same questions, but with different expectations for both the answers and the
degree of certainty of those answers.

Congestion control algorithms without empirical evidence of Internet-scale
deployment MUST seek Experimental status, unless they are not targeted at
general use.

Specifications published as Experimental ought to explain the reason for
the status and what further information would be required to progress to
standards track. For example, section 12 of {{?RFC6928}} provides
“Usage and Deployment Recommendations” that describe the experiments
expected by the TCPM working group. Section 4 of {{?RFC4614}} provides other
examples of extensions that were considered experimental
when the specification was published.

Experimental specifications SHOULD NOT be deployed as a default. They SHOULD
only be deployed in situations where they are being actively measured, and where
it is possible to deactivate them if there are signs of pathological behavior.

Congestion control algorithms with a
record of measured Internet-scale deployment MAY directly seek the Standards
Track if there is solid data that reflects that it is safe, and the design is
stable, guided by the considerations in {{general-use}}. However, the existence
of this data does not waive the other considerations in this document.

Each published congestion control algorithm is REQUIRED to include a statement
in the abstract indicating whether or not there is IETF consensus that the
proposed congestion control algorithm is considered safe for use on the
Internet. Each published algorithm is also REQUIRED to include a statement in
the abstract describing environments where the protocol is not recommended for
deployment. There can be environments where the congestion control algorithm is
deemed safe for use, but it is still is not recommended for use because it
does not perform well for the user.

As examples of such statements, {{?RFC3649}} specifies HighSpeed TCP and
includes a statement in the abstract stating that the proposed congestion
control algorithm is Experimental, but may be deployed in the Internet. In
contrast, the Quick-Start document {{?RFC4782}} includes a paragraph in the
abstract stating that the mechanism is only being proposed for use in
controlled environments. The abstract specifies environments where the
Quick-Start request could give false positives (and therefore would be unsafe for
incremental deployment where some routers forward, but do not process the
option). The abstract also specifies environments where packets containing the
Quick-Start request could be dropped in the network; in such an environment,
Quick-Start would not be unsafe to deploy, but deployment is not recommended
because it could lead to unnecessary delays for the connections attempting to
use Quick-Start. The Quick-Start method is discussed as an example in
{{?RFC9049}}.

Strictly speaking, Informational RFCs in the IETF stream need not meet all of
the criteria in this document, as they do not carry a formal recommendation
from the community. Instead, the community judges the publication of
Informational RFCs based on the value of their addition to the RFC series.

Although out of the scope of this document, proponents of a new algorithm could
alternatively seek publication as an Informational or Experimental RFC via the
Internet Research Task Force (IRTF). In general, these algorithms are expected
to be less mature than ones that follow the procedures in this document. Authors
documenting deployed congestion control algorithms that cannot be changed by
IETF or IRTF review are invited to seek publication as an Informational RFC via
the Independent Stream Editor (ISE).

# Specifying Algorithms for Use in Controlled Environments {#controlled-environments}

Algorithms can be designed for general Internet deployment or for use in
controlled environments {{?RFC8799}}. Within a controlled environment,
an operator can ensure that flows
are isolated from other Internet flows, or they might
allow these flows to share resources with other Internet flows.
A data center is an example of a controlled environment, which often deploys
fabrics with rich signalling from switches to endpoints.

Algorithms that
rely on specific functions or configurations in the network need to provide a
reference or specification for these functions (an RFC or another stable
specification). For publication to proceed, the IETF will need to assess whether
a working group exists that can properly assess the network-layer aspects and
their interaction with the congestion control.

In evaluating a new proposal for use in a controlled environment, the IETF needs
to understand the usage, e.g., how the usage is scoped to the controlled
environment, whether the algorithm will share resources with Internet traffic,
and consider what could happen if used in a protocol that is bridged across an
Internet path. Algorithms that are designed to be confined to a controlled
environment and are not intended for use in the general Internet, might instead
seek real-world data for those environments. In such cases, the evaluation
criteria in the remainder of this document might not apply.

# Evaluation Criteria {#evaluation-criteria}

As previously noted, authors are expected to conduct a comprehensive evaluation
of the advantages and disadvantages of congestion control algorithms presented
to the IETF. The following guidelines are intended to assist authors and the
IETF community in this endeavor. While these guidelines provide a helpful
framework, they should not be regarded as an exhaustive checklist, as concerns
beyond the scope of these guidelines may also arise.

When considering a proposed congestion control algorithm, the community MUST
consider the following criteria. These criteria will be evaluated in various
domains (see {{general-use}} and {{special-cases}}).

Some of the sections below will list criteria that SHOULD be met. It could
happen that these criteria are not in fact met by the proposal. In such cases,
the community MUST document whether not meeting the criteria is acceptable, for
example because there are practical limitations on carrying out an evaluation of
the criteria.

The requirement that the community consider a criterion does not imply that the
result needs to be described in a resulting RFC. There is no formal requirement
to document the results, although normal IETF policies for archiving proceedings
will provide a record.

This document, except where otherwise noted, does not provide normative guidance
on the acceptable thresholds for any of these criteria. Instead, the community
will use these evaluations as an input when considering whether to progress the
proposed algorithm.

## Single Algorithm Behavior

The criteria in this section evaluate the congestion control algorithm when one
or more flows using that algorithm share a bottleneck link (i.e., with no flows
using a differing congestion control algorithm).

### Protection Against Congestion Collapse

A congestion control algorithm should either stop sending when the packet drop
rate exceeds some threshold {{?RFC3714}}, or should include some notion of "full
backoff". For "full backoff", at some point the algorithm would reduce the
sending rate to one packet per round-trip time and then exponentially backoff
the time between single packet transmissions if the congestion persists. Exactly
when either "full backoff" or a pause in sending comes into play will be
algorithm-specific.  However, as discussed in {{!RFC2914}} and {{!RFC8961}},
this requirement is crucial to protect the network in times of extreme
(persistent) congestion.

If full backoff is used, this test does not require that the mechanism must be
identical to that of TCP ({{?RFC6298}}, {{!RFC8961}}). For example, this does
not preclude full backoff mechanisms that would give flows with different round-
trip times comparable capacity during backoff.

### Protection Against Bufferbloat

A congestion control algorithm should try to avoid maintaining excessive queues
in the network. Exactly how the algorithm achieves this is algorithm-specific,
but see {{!RFC8961}} and {{!RFC8085}} for requirements.

Bufferbloat {{Bufferbloat}} refers to the building of excessive queues in the
network. Many network routers are configured with very large buffers. The
standards-track Reno {{!RFC5681}} and CUBIC {{?RFC9438}} congestion control
algorithms send at progressively higher rates until a First-In First-Out (FIFO)
buffer completely fills, and packet losses then occur. Every connection passing
through that bottleneck experiences increased latency due to the high buffer
occupancy. This adds unwanted latency that negatively impacts highly interactive
applications such as videoconferencing or games, but it also affects routine web
browsing and video playing.

This problem has been widely discussed since 2011 {{Bufferbloat}}, but was not
discussed in the Congestion Control Principles published in September 2002
{{!RFC2914}}. The Reno and CUBIC congestion control algorithms do not address
this problem, but a new congestion control algorithm has the opportunity to improve the
state of the art.

### Protection Against High Packet Loss

A congestion control algorithm should try to avoid causing excessively high
rates of packet loss. To accomplish this, it should avoid excessive increases in
sending rate, and reduce its sending rate if experiencing high packet loss.

The first version of the BBR algorithm {{BBRv1-draft}} failed this requirement.
Experimental evaluation {{BBRv1-Evaluation}} showed that it caused a sustained
rate of packet loss when multiple BBRv1 flows shared a bottleneck and the buffer
size was less than roughly one and a half times the Bandwidth Delay Product
(BDP). This was unsatisfactory, and indeed further versions provided a fix for this
aspect of BBR {{BBR-draft}}.

This requirement does not imply that the algorithm should react to packet losses
in exactly the same way as current standards-track congestion control algorithms
(e.g., {{!RFC5681}}).

### Fairness within the Proposed Congestion Control Algorithm

When multiple competing flows all use the same proposed congestion control
algorithm, the proposal should explore how the capacity is shared among the
competing flows. Capacity fairness can be important when a small number of
similar flows compete to fill a bottleneck. However, it can also not be useful,
for example, when comparing flows that seek to send at different rates, or if
some of the flows do not last sufficiently long to approach asymptotic behavior.

### Short Flows {#short-flows}

A great deal of congestion control analysis concerns the steady-state behavior
of long flows. However, many Internet flows are relatively short-lived.
Many short-lived flows today remain in the "slow
start" mode of operation {{?RFC5681}} that commonly features exponential
congestion window growth because the flow
never experiences congestion (e.g., packet loss).

A proposed congestion control algorithm MUST consider how new and short-lived
flows affect long-lived flows, and vice versa.

## Mixed Algorithm Behavior

Mixed algorithm behavior criteria evaluate the interaction of the proposed
congestion control algorithm with commonly deployed congestion control
algorithms.

In contexts where differing congestion control algorithms are used, it is
important to understand whether the proposed congestion control algorithm could
result in more harm than previous standards-track algorithms (e.g.,
{{!RFC5681}}, {{!RFC9002}}, {{!RFC9438}}) to flows sharing a common bottleneck.
The measure of harm is not restricted to unequal capacity, but ought also to
consider metrics such as the introduced latency, or an increase in packet loss.
An evaluation MUST assess the potential to cause starvation, including
assurance that a loss of all feedback (e.g., detected by expiry of a
retransmission time out) results in backoff.

### Existing General-Purpose Congestion Control

A proposed congestion control algorithm MUST be evaluated when competing against
standard IETF congestion controls, e.g. {{!RFC5681}}, {{!RFC9002}},
{{!RFC9438}}. A proposed congestion control algorithm that has a significantly
negative impact on flows using standard congestion control might be suspect, and
this aspect should be part of the community's decision making with regards to
the suitability of the proposed congestion control algorithm. The community
should also consider other non-standard congestion control algorithms that are
known to be widely deployed.

Note that this guideline is not a requirement for strict Reno- or CUBIC-
friendliness as a prerequisite for a proposed congestion control mechanism to
advance to Experimental or Standards Track status. As an example, HighSpeed TCP
is a congestion control mechanism specified as Experimental, that is not TCP-
friendly in all environments. When a new congestion control algorithm is
deployed, the existing major algorithm deployments need to be considered to
avoid severe performance degradation. Note that this guideline does not
constrain the interaction with non-best-effort flows.

As an example from an Experimental RFC, fairness with standard TCP is discussed
in Sections 4 and 6 of {{?RFC3649}} (HighSpeed TCP) and using spare capacity is
discussed in Sections 6, 11.1, and 12 of {{?RFC3649}}.

### Real-Time Congestion Control

General-purpose algorithms need to coexist in the Internet with real-time
congestion control algorithms, which, in general, have finite throughput
requirements (i.e., do not seek to utilize all available capacity) and more
strict latency bounds. See {{?RFC8836}} for a description of the characteristics
of this use case and the resulting requirements.

{{?RFC8868}} provides suggestions for real-time congestion control design and
{{?RFC8867}} suggests test cases. {{?RFC9392}} describes some considerations
for the RTP Control Protocol (RTCP). In particular, real-time flows
can use less frequent feedback (acknowledgement) than that provided by reliable transports.
This document does not change the informational status of those RFCs.

A proposed congestion control algorithm SHOULD consider coexistence with widely
deployed real-time congestion control algorithms. Regrettably, at the time of
writing (2024), many algorithms with detailed public specifications are not
widely deployed, while many widely deployed real-time congestion control
algorithms have incomplete public specifications. It is hoped that this
situation will change.

To the extent that behavior of widely deployed algorithms is understood,
proponents of a proposed congestion control algorithm can analyze and simulate
a proposal's interaction with those algorithms. To the extent they are not, experiments
can be conducted where possible.

Real-time flows can be directed into distinct queues via Differentiated Services
Code Points (DSCP) or other mechanisms, which can substantially reduce the
interplay with other traffic. However, a proposal targeting general Internet use
can not assume this is always the case.

{{circuit-breakers}} describes the impact of network transport circuit breaker
algorithms. {{!RFC8083}} also defines a minimal set of RTP circuit breakers that
operate end-to-end across a path. This identifies conditions under which a sender needs to
stop transmitting media data to protect the network from excessive congestion.
It is expected that, in the absence of long-lived excessive congestion, RTP
applications running on best-effort IP networks will be able to operate without
triggering these circuit breakers.

### Short and Long Flows

The effect on short-lived and long-lived flows using other common congestion
control algorithms MUST be evaluated, as in {{short-flows}}.

## Other Criteria

### Differences with Congestion Control Principles

A proposed congestion control algorithm MUST clearly explain any deviations from
{{!RFC2914}} and {{!RFC7141}}.

### Incremental Deployment

A congestion control algorithm proposal MUST discuss whether it allows for
incremental deployment in the targeted environment. For a mechanism targeted for
deployment in the current Internet, the proposal SHOULD discuss what is known
(if anything) about the correct operation of the mechanisms with some of the
equipment in the current Internet, e.g., routers, transparent proxies, WAN
optimizers, intrusion detection systems, home routers, and the like.

Similarly, if the proposed congestion control algorithm is intended only for
specific environments (and not the global Internet), the proposal SHOULD
consider how this intention is to be realised.  The community will have to
address the question of whether the scope can be enforced by stating the
restrictions, or whether additional protocol mechanisms are required to enforce
this scoping.  The answer will necessarily depend on the proposed change.

As an example from an Experimental RFC, deployment issues are discussed in
Sections 10.3 and 10.4 of {{?RFC4782}} (Quick-Start).

# General Use {#general-use}

The criteria in {{evaluation-criteria}} will be evaluated in the following
scenarios. Unless a proposed congestion control specification explicitly forbids
use on the public Internet, there MUST be IETF consensus that it meets the
criteria in these scenarios for the proposed congestion control algorithm to
 progress.

The evaluation in each scenario SHOULD occur over a representative range of
bandwidths, delays, and queue depths. Of course, the set of parameters
representative of the public Internet will change over time.

These criteria are intended to capture a statistically dominant set of Internet
conditions. In the case that a proposed congestion control algorithm has been
tested at Internet scale, the results from that deployment are often useful for
answering these questions.

## Paths with Tail-drop Queues

The performance of a congestion control algorithm is affected by the queue
discipline applied at the bottleneck link. The drop-tail queue discipline (using
a FIFO buffer) MUST be evaluated. See {{aqm}} for evaluation of other queue
disciplines.

## Tunnel Behavior

When a proposed congestion control algorithm relies on explicit signals from the
path, the proposal MUST consider the effect of traffic passing through a tunnel,
where routers may not be aware of the flow.

The design of tunnels and similar encapsulations might need to consider nested
congestion control interactions. For example, when ECN is used by both an
IP and lower layer technology {{ECN-Encaps}}.

## Wired Paths

Wired networks are usually characterized by extremely low rates of packet loss
except for those due to queue drops. They tend to have stable aggregate
capacity, usually higher than other types of links, and low non-queueing delay.
Because the properties are relatively simple, wired links are typically used as a
"baseline" case even if they are not always the bottleneck link in the modern
Internet.

## Wireless Paths

While the early Internet was dominated by wired links, the properties of
wireless links have become important to Internet performance. In
particular, a proposed congestion control algorithm should be evaluated in
situations where some packet losses are due to radio effects, rather than router
queue drops; the link capacity varies over time due to changing link conditions;
and media access delays and link-layer retransmission lead to increased jitter
in round-trip times. See {{?RFC3819}} and Section 16 of {{Tools}} for further
discussion of wireless properties.

# Special Cases {#special-cases}

The criteria in {{evaluation-criteria}} will be evaluated in the following
scenarios, unless the proposed congestion control algorithm specifically
excludes its use in a scenario. For these specific use-cases, the community MAY
allow a proposal to progress even if the criteria indicate an unsatisfactory
result for these scenarios.

In general, measurements from Internet-scale deployments might not expose the
properties of operation in each of these scenarios, because they are not as
ubiquitous as the General Use scenarios.

## Active Queue Management (AQM) {#aqm}

The proposed congestion control algorithm SHOULD be evaluated under a variety of
bottleneck queue disciplines. The effect of an AQM discipline can be hard to
detect by Internet evaluation. At a minimum, a proposal should reason about an
algorithm's response to various AQM disciplines. Simulation or empirical results
are, of course, valuable.

Among the AQM techniques that might have an impact on a proposed congestion
control algorithm are Flow Queue CoDel (FQ-CoDel) {{?RFC8290}}; Proportional Integral Controller
Enhanced (PIE) {{?RFC8033}}; and Low Latency, Low Loss, and Scalable Throughput
(L4S) {{?RFC9332}}.

A proposed congestion control algorithm that sets one of the two Explicit
Congestion Transport (ECT) codepoints in the IP header can gain the benefits of
receiving Explicit Congestion Notification (ECN) Congestion Experienced (CE)
signals from an on-path AQM {{?RFC8087}}. Use of ECN {{?RFC3168}}, {{?RFC9332}}
requires the congestion control algorithm to react when it receives a packet
with an ECN-CE marking. This reaction needs to be evaluated to confirm that the
algorithm conforms with the requirements of the ECT codepoint that was used.

Note that evaluation of AQM techniques -- as opposed to their impact on a
specific proposed congestion control algorithm -- is out of scope of this
document. {{?RFC7567}} describes design considerations for AQMs.

## Operation with the Envelope set by Network Circuit Breakers {#circuit-breakers}

Some equipment in the network uses an automatic mechanism to continuously
monitor the use of resources by a flow or aggregate set of flows {{!RFC8084}}.
Such a network transport circuit breaker can automatically detect excessive
congestion, and when detected, it can terminate (or significantly reduce the
rate of) the flow(s). A well-designed congestion control algorithm ought to
react before the flow uses excessive resources, and therefore will operate
within the envelope set by network transport circuit breaker algorithms.

## Paths with Varying Delay {#delay}

An Internet Path can include simple links, where the minimum delay is the
propagation delay, and any additional delay can be attributed to link buffering.
This cannot be assumed. An Internet Path can also include complex subnetworks
where the minimum delay changes over various time scales, resulting in a non-
stationary minimum delay.

Varying delay occurs when a subnet changes the forwarding path to optimise capacity,
resilience, etc. It could also arise when a subnet uses a capacity management
method where the available resource is periodically distributed among the active
nodes. A node might then have to buffer data until an assigned transmission
opportunity or until the physical path changes (e.g., when the length of a
wireless path changes, or the physical layer changes its mode of operation).
Variation also arises when traffic with a higher priority DSCP pre-empts
transmission of traffic with a lower class. In these cases, the delay varies as
a function of external factors, and attempting to infer congestion from an
increase in the delay results in reduced throughput. This variation in the
delay over short timescales (jitter) might not be distinguishable from jitter
that results from other effects.

A proposed congestion control algorithm SHOULD be evaluated to ensure its
operation is robust when there is a significant change in the minimum delay.

## Internet of Things and Constrained Nodes

The "Internet of Things" (IoT) is a broad concept, but when evaluating a
proposed congestion control algorithm, it is often associated with unique
characteristics:
IoT nodes might be more constrained in power, CPU, or other parameters than
conventional Internet hosts. This might place limits on the complexity of any
given algorithm. These power and radio constraints might make the volume of
control packets in a given algorithm a key evaluation metric.

Extremely low-power links can lead to very low throughput and a low bandwidth-
delay product, well below the standard operating range of most Internet flows.

Furthermore, many IoT applications do not a have a human in the loop, and
therefore might have weaker latency constraints because they do not relate to a
user experience. Congestion control algorithm can still need to share the
path with other flows with different constraints.

## Paths with High Delay

A proposed congestion control algorithm ought not to presume that all general
Internet paths have a low delay. Some paths include links that contribute much
more delay than for a typical Internet path. Satellite links often have delays
longer than typical for wired paths {{?RFC2488}} and high delay bandwidth
products {{?RFC3649}}.

Paths can also present a variable delay as described in {{delay}}.

## Misbehaving Nodes

A proposed congestion control algorithm SHOULD explore how the algorithm
performs with non-compliant senders, receivers, or routers.  In addition, the
proposal should explore how a proposed congestion control algorithm performs
with outside attackers.  This can be particularly important for proposed
congestion control algorithms that involve explicit feedback from routers along
the path.

As an example from an Experimental RFC, performance with misbehaving nodes and
outside attackers is discussed in Sections 9.4, 9.5, and 9.6 of {{?RFC4782}}.
This includes discussion of misbehaving senders and receivers; collusion between
misbehaving routers; misbehaving middleboxes; and the potential use of Quick-
Start to attack routers or to tie up available Quick-Start bandwidth.

## Extreme Packet Reordering

A proposed congestion control algorithm ought not to presume that all general
Internet paths reliably deliver packets in order. {{?RFC4653}} discusses the
effect of extreme packet reordering.

## Transient Events

A proposed congestion control algorithm SHOULD consider how the proposed
congestion control algorithm would perform in the presence of transient events
such as sudden onset of congestion, a routing change, or a mobility event.
Routing changes, link disconnections, intermittent link connectivity, and
mobility are discussed in more detail in Section 16 of {{Tools}}.

As an example from an Experimental RFC, response to transient events is
discussed in {{Section 9.2 of ?RFC4782}}.

## Sudden changes in the Path

An IETF transport is not tied to a specific Internet path or type of path. The
set of routers that form a path can and do change with time. This will cause the
properties of the path to change with respect to time. A proposed congestion
control algorithm MUST evaluate the impact of changes in the path, and be robust
to changes in path characteristics on the interval of common Internet re-routing
intervals.

## Multipath Transport

Multipath transport protocols permit more than one path to be differentiated and
used by a single connection at the sender. A multipath sender can schedule which
packets travel on which of its active paths. This enables a tradeoff in
timeliness and reliability. There are various ways that multipath techniques can
be used.

One example use is to provide fail-over from one path to another when the
original path is no longer viable, or provides inferior performance.  Designs
need to independently track the congestion state of each path, and demonstrate
independent congestion control for each path being used. Authors of a proposed
multipath congestion control algorithm that implements path fail-over MUST
evaluate the harm to performance resulting from a change in the path, and show that this does
not result in flow starvation. Synchronisation of failover (e.g., where multiple
flows change their path on similar timeframes) can also contribute to harm
and/or reduce fairness. These effects also ought to be evaluated.

Another example use is concurrent multipath, where the transport protocol
simultaneously schedules a flow to aggregate the capacity across multiple paths.
The Internet provides no guarantee that different paths (e.g., using different
endpoint addresses) are disjoint. This introduces additional implications: A congestion
control algorithm proposal MUST evaluate the potential harm to other flows when
the multiple paths share a common congested bottleneck or share resources that
are coupled between different paths, such as an overall capacity limit). A proposal
SHOULD consider the potential for harm to other flows. Synchronisation of
congestion control mechanisms (e.g., where multiple flows change their behaviour
on similar timeframes) can also contribute to harm and/or reduce fairness. These
effects also ought to be evaluated.

At the time of writing (2024), there are currently no Standards Track RFCs for
concurrent multipath, but there is an Experimental RFC {{?RFC6356}} that
specifies a concurrent multipath congestion control algorithm for MPTCP
{{?RFC8684}}.

## Data Centers

Data centers are characterized by very low latencies (< 2 ms). Many workloads
involve bursty traffic where many nodes complete a task at the same time. As a
controlled environment, data centers often deploy fabrics that employ rich
signalling from switches to endpoints. Furthermore, the operator can often limit
the number of operating congestion control algorithms.

For these reasons, data center congestion controls are often distinct from those
running elsewhere on the Internet (see {{controlled-environments}}).  A proposed
congestion control need not coexist well with all other algorithms if it is
intended for data centers, but the proposal SHOULD indicate which are expected
to safely coexist with it.

# Security Considerations

This document does not represent a change to any aspect of the TCP/IP protocol
suite and therefore does not directly impact Internet security.  The
implementation of various facets of the Internet's current congestion control
algorithms do have security implications (e.g., as outlined in {{!RFC5681}}).

Proposed congestion control algorithms MUST examine any potential security or
privacy issues that may arise from their design.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

Sally Floyd and Mark Allman were the authors of this document's predecessor,
{{?RFC5033}}, which served the community well for over a decade.

Thanks to Richard Scheffenegger for helping to get this revision process
started.

The editors would like to thank Mohamed Boucadair, Neal Cardwell, Reese
Enghardt, Jonathan Lennox, Matt Mathis, Zahed Sarker, Juergen Schoenwaelder,
Dave Taht, Sean Turner, Michael Welzl, Magnus Westerlund, and Greg White for
suggesting improvements to this document.

Discussions with Lars Eggert and Aaron Falk seeded the original RFC5033. Bob
Briscoe, Gorry Fairhurst, Doug Leith, Jitendra Padhye, Colin Perkins, Pekka
Savola, members of TSVWG, and participants at the TCP Workshop at Microsoft
Research all provided feedback and contributions to that document. It also drew
from {{?RFC5166}}.

# Evolution of RFC5033bis
{:numbered="false"}

## Since draft-ietf-ccwg-rfc5033bis-06
{:numbered="false"}
- OPSDIR review
- ARTART review

## Since draft-ietf-ccwg-rfc5033bis-05
{:numbered="false"}
- AD evaluation comments

## Since draft-ietf-ccwg-rfc5033bis-04
{:numbered="false"}
- Editorial pass after shepherd review.

## Since draft-ietf-ccwg-rfc5033bis-03
{:numbered="false"}
- Harmonised the "proposed congestion control algorithm"
- Addressed issues.
- Examined RFC-2119 keywords and consistency with other RFCs.
- Added text on constrained environments/limited domains
- Added text on circuit breakers and aligned with other RFCs.
- Several editorial passes

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
- Mentioned CUBIC and BBR as motivation
- Include section to track updates between revisions
- Update references

## Since RFC5033
{:numbered="false"}

- converted to Markdown and xml2rfc v3
- various formatting changes.

