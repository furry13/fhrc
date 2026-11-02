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
  RFC4862:
  RFC7772:
  RFC8475:
  RFC9099:

...

--- abstract

This document provides operational guidance for configuring first-hop routers and first-hop redundancy in IPv6 networks. It also discusses design choices for improving recovery time after a renumbering event.


--- middle

# Introduction {#intro}

Unlike IPv4, the IPv6 protocol supports operation with multiple gateways as a native part of the Neigbor Discovery Protocol {{RFC4861}}. On the other hand, first hop redundancy protocols used in IPv4 like Virtual Router Redundancy Protocol {{RFC9568}} can be used for IPv6 as well, making the redundant gateways appear as a single logical gateway.

This document discusses different options for deploying redundant first-hop IPv6 gateways to ensure high availability and compatibility with various client implementations. First-hop routers face hosts and provide them with information to configure routing as well as well other network parameters, including but not limited to stateless address autoconfiguration, SLAAC {{RFC4862}}.


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

Since the timers are configured locally on each host, it is not possible to influence them from the network side. It is namely not possible to achieve shorter failover time by shortening the Router Lifetime of the active router to a very low value. Not only that leads to frequent sending of Router Advertisements, which has an impact on battery-powered devices {{RFC7772}}, some operating systems are known to ignore routers announced with a very short lifetime.


## Virtual Router Redundancy Protocol (VRRP) or Similar Protocols

VRRPv3 {{RFC9568}} specifies an election protocol that dynamically assigns responsibility for a Virtual Router to one of the VRRP Routers on a LAN.
A Virtual Router has one or more dedicated virtual IPv6 addresses and a virtual MAC address.
Hosts are provided with the virtual IPv6 address as the default router via Router Advertisements.
Since the hosts only see a single high available logical gateway, the solution is also useful for hosts with broken IPv6 stacks which cannot correctly process Router Advertisements from multiple independent routers.


Section 1.4 of {{RFC9568}} compares convergence time between NUD (as discussed in the section above) and VRRPv3.
Using VRRPv3 default parameters, a backup router can take over a failed default router in three seconds, with the switchover being transparent to hosts.



## Anycast Default Gateway

An Anycast Default Gateway assigns the identical IPv6 address and Virtual MAC (VMAC) address across multiple first-hop routers attached to the same broadcast domain or overlay network (e.g., EVPN-VXLAN).

Similarly to the previous solution, hosts will configure a single high available logical gateway using Router Advertisements.
Because every participating router concurrently processes and forwards first-hop ingress traffic, this approach provides active-active first-hop redundancy and multi-path forwarding.
In the event of a router failure, convergence is dictated solely by local physical link-down detection or underlying L2/L3 topology reconvergence.

## Routing return traffic back to Hosts

All redundant gateways must correctly forward packets destined to the subnet where the hosts are connected, usually by means of an Internal Gateway Protocol (IGP). By default, this can lead to a routing asymmetry where return traffic uses a different gateway than the forward traffic. This is not a problem, unless some stateful processing like firewall rules or connection tracking takes place on the gateways.

If routing asymmetry is not desirable and gateways are operated in active-standby mode, symmetric routing can be achieved by influencing the route metric of the active gateway announced in the IGP together with virtual IP/MAC address failover so that the currently active gateway will provide a more preferred route for the return path.

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
