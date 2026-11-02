---
title: "First Hop Redundancy in IPv6 Networks: Operational Considerations"
abbrev: "v6fhr"
category: info

docname: draft-calink-v6ops-fhrd-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "IPv6 Operations Working Group"
keyword:
 - ipv6
 - nd
 - vrrp
 - ra
venue:
  group: "IPv6 Operations"
  type: "Working Group"
  mail: v6ops@ietf.org
  github: furry13/fhrc
  arch: "https://mailarchive.ietf.org/arch/browse/v6ops/"

author:
 -
    fullname: Ondrej Caletka
    organization: RIPE NCC
    email: ondrej.caletka@ripe.net

 -
    fullname: Jen Linkova
    organization: Google, LLC
    email: furry13@gmail.com

normative:
  RFC2119:
  RFC4191:
  RFC4861:
  RFC8174:
  RFC9568:

informative:
  RFC8475:
  RFC9099:

...

--- abstract

This document provides operational guidance for configuring first-hop routers and first-hop redundancy in IPv6 networks. It also discusses design choices for improving recovery time after a renumbering event. 


--- middle

# Introduction {#intro}

First-hop routers face hosts and provide hosts with information to configure hosts IPv6 stack via StateLess Address AutoConfiguration, SLAAC {{RFC4861}}.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Design Choices to Provide First Hop Redundancy

## Two (or more) independent Routers

One of the posisble deployment scenarios is to deploy multiple routers in the same link.
A host can choose one of those routers as a default router and if the selected router becomes unreachable, switch to another reachable one.
If multiple routers are present on a link, Section 6.3.4 of {{RFC4861}} requiresthat an IPv6 host MUST retain at least two router addresses and SHOULD retain more.

If the administrator requires all hosts to use the same router, the following mechanisms can be utilized:

* Configure the default router preference {{RFC4191}}.
* Advertize backup routers with Router Lifetime set to zero, changing that value to non-zero automatically when a primary router fails. This approach is similar to one described in {{RFC8475}}.

The main disadvantace of this approach is time to recover: a host would only switch to another default router from the list after the Nighbor Unreachability Detection (NUD, {{RFC4861}}) marks the router as unreachable.
The default value would be (without taking randomisation factors into account): ReachableTime milliseconds (to move from REACHABLE to STALE) + DELAY_FIRST_PROBE_TIME + MAX_UNICAST_SOLICIT*RetransTimer = 30 seconds + 5 second + 3*1 = 38 seconds.


## Virtual Router Redundancy Protocol (VRRP) or Similar Protocols

VRRPv3 {{RFC9568}} specifies an election protocol that dynamically assigns responsibility for a Virtual Router to one of the VRRP Routers on a LAN.
A Virtual Router has one or more dedicated virtual IPv6 addresses and a virtual MAC address.
Hosts are provided with the virtual IPv6 address as the default router via SLAAC.

Section 1.4 of {{RFC9568}} compares convergence time between NUD (as discussed in the section above) and VRRPv3.
Using VRRPv3 default parameters, a backup router can take over a failed default router in three seconds, with the switchover being transparent to hosts.



## Anycast Default Gateway

An Anycast Default Gateway assigns the identical IPv6 address and Virtual MAC (VMAC) address across multiple first-hop routers attached to the same broadcast domain or overlay network (e.g., EVPN-VXLAN).

Hosts configure that IPv6 address as a default router via SLAAC.
Because every participating router concurrently processes and forwards first-hop ingress traffic, this approach provides active-active first-hop redundancy and multi-path forwarding.
In the event of a router failure, convergence is dictated solely by local physical link-down detection or underlying L2/L3 topology reconvergence.

## Routing to a Host

TODO: add text

# Timers

Questions:

* How often to send RAs?
* How other times in RA should be re: to default router lifetime?


# Host Mobility and Renumbering Evens

AI: copy some text from draft-link-6man-gulla


# Security Considerations {#security}

This document does not introduce any new security considerations.
It inherits security considerations of IPv6 (see {{RFC9099}}). 

# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
