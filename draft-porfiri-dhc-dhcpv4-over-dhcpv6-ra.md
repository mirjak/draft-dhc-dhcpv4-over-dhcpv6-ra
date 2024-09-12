---
title: "DHCPv4 over DHCPv6 with Relay Agent Support"
abbrev: "DCHP 4o6 Relay Agent"
category: std

docname: draft-porfiri-dhc-dhcpv4-over-dhcpv6-ra-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Dynamic Host Configuration"
keyword:
 - dhcp
venue:
  group: "Dynamic Host Configuration"
  type: "Working Group"
  github: "mirjak/dhc-dhcpv4-over-dhcpv6-ra"

author:
 -
    ins: C. Porfiri
    name: Claudio Porfiri
    organization: Ericsson
    email: claudio.porfiri@ericsson.com
 -
    ins: S. Krishnan
    name: Suresh Krishnan
    organization: Cisco
    email: suresh.krishnan@gmail.com
 -
    ins: J. Arkko
    name: Jari Arkko
    organization: Ericsson
    email: jari.arkko@ericsson.com
 -
    ins: M. Kühlewind
    name: Mirja Kühlewind
    organization: Ericsson
    email: mirja.kuehlewind@ericsson.com

normative:
   RFC3046:
   RFC3315:
   RFC6221:
   RFC6925:
   RFC7341:
   RFC7969:
   RFC8415:

informative:
   RFC0951:
   RFC1542:
   RFC2132:
   RFC2131:

--- abstract

This document describes a mechanism for networks
with legacy IPv4-only clients to use services provided by
DHCPv6 using DHCPv4-over-DHCPv6 (DHCP 4o6) in a Relay Agent.
RFC7341 specifies use of DHCPv4-over-DHCPv6 in the client only.
This document specifies
a RFC7341-based approach that allows DHCP 4o6 to be deployed as a
Relay Agent (4o6RA) that implements the 4o6 DHCP en- and decapsulation
if this is not possible at the client.

--- middle

# Introduction {#introduction}

{{RFC7341}} describes a transport mechanism for carrying DHCPv4
messages using the DHCPv6 protocol for the dynamic provisioning
of IPv4 addresses and other DHCPv4 specific configuration parameters
across IPv6-only networks. The deployment of {{RFC7341}} requires implementation in
all DHCP clients and at the DHCPv6 server.

However, if the client only supports IPv4 and cannot easily be replaced or updated
due to a number of technical or business reasons, this approach does not work.

Similarly, the specifications for DHCPv6 Relay Agents such as LDRA {{RFC6221}}
or L3RA {{RFC8415}} do not foresee the possibility to handle legacy DHCP,
other than implementing 4o6 in client.

This document specifies an {{RFC7341}}-based solution that can be
implemented in intermediate nodes such as L2 switches or routers,
without putting any requirements on clients. No new protocols or extensions are needed,
instead this document specifies an amendment to [RFC7341] that allows
a Relay Agent to perform the 4o6 DHCP en- and decapsultion instead of
the client.


# Conventions and Definitions

The following terms and acronyms are used in this document:

* DHCPv4 over DHCPv6 (or 4o6):
   The architecture, the procedures and the protocols described in the
   DHCPv4-over-DHCPv6 document {{RFC7341}}.

* DHCP Relay Agent (or RA):
   This is a concept in all of the protocols, BOOTP {{RFC0951}} {{RFC1542}}, DHCPv4
   {{RFC2131}} {{RFC2132}}, and DHCPv6 {{RFC8415}}, although the details differ
   between the protocols.

* Lightweight DHCPv6 Relay Agent (or LDRA):
   This is an extension of the original DHCPv6 Relay Agent mechanism,
   to support also Layer 2 devices performing a Relay Agent function {{RFC6221}}.
* DHCPv4 over DHCPv6 Relay Agent (or 4o6RA):
   The 4o6 Relay Agent (as specified in this document)
   is the part of an RA implementing 4o6.

{::boilerplate bcp14-tagged}

# DHCPv4 over DHCPv6 Relay Agent (4o6RA)

This document assume an network, where IPv4-only clients are connected
to an uplink network that supports IPv6 only and limited IPv4 services.

To address such a network setup, this document proposes to extend
DHCPv6 Relay Agents with  DHCPv4 over DHCPv6, as
shown in {{fig_4o6RA}}.

~~~aasvg

                 .-----------.             .-----------.
                |             |           |             |
       +--------+-+    L2   +-+-----------+-+  IPv6   +-+--------+
       |  DHCPv4  | Network |    DHCPv6     | Network | DHCP 4o6 |
       |  Client  +---------+  Relay Agent  +---------+  Server  |
       |          |         |   with 4o6RA  |         |          |
       +--------+-+         +-+-----------+-+         +-+--------+
                |             |           |             |
                 '-----------'             '-----------'

~~~
{: #fig_4o6RA title="Architecture Overview with legacy DHCP client" artwork-align="center"}

This document specifies the encapsulation
and decapsulation described in {{RFC7341}} to be performed in the Relay Agent
whereas the DHCP Client does not require any change.
In this case it is up to the Relay Agent to provide the full
4o6 DHCP set of functionality whereas the legacy client is not aware of being served
via a 4o6 DHCP service.
All prerequisites and configuration that to the DHCP client in {{Section 5 of RFC7341}}
apply shall be applied to the 4o6RA instead.

To maintain interoperability with existing DHCP relays and servers,
the message format is unchanged from {{RFC8415}}. The 4o6RA implements
the same message types as a normal DHCPv6 Relay Agent {{Section 6 of RFC7341}}.

In this specification, the 4o6RA creates the DHCPV4-QUERY Message
and encapsulates the DHCP request message received from the legacy DHCPv4 client.

When DHCPV4-RESPONSE Message is received by the 4o6 Relay Agent,
it looks for the DHCPv4 Message option within this message.
If this option is not found, the DHCPv4-response message MUST be discarded.
If the DHCPv4 Message option is present, the 4o6RA MUST extract the DHCPv4
message and forward the encapsulated DHCPv4-response to the legacy DHCPv4 client.

Any Layer 2 Relay Agent receiving DHCPV4-QUERY or DHCPV4-RESPONSE messages
will handle them as specified in {{Section 6 of RFC6221}}.

The DHCPv6 server must be compliant with 4o6 according to {{RFC7341}}.
No additional requirements on DHCPv6 server are set by this specification.

# Using 4o6RA for Topology Discovery {#topology_considerations}

In some networks the configuration of a client host may depend on the
topology.  However, when a new client host gets connected to the
network, it may be unaware of the topology and respectively how it
has to be configured.

The DHCPv4 {{RFC2131}} and DHCPv6 {{RFC3315}} protocol specifications
describe how addresses can be allocated to clients based on network
topology information provided by the DHCP relay infrastructure.

In IPv6 networks, Topology discover can be realized using DHCPv6
Relay Agents {{RFC6221}} that insert relay agent options in DHCPv6
message exchanges in order to identify the client-facing interfaces,
e.g. using the Serial Number or other hardcoded information.  Then, a
reference host that is responsible for providing configuration to the
client host can obtain topology information from the DHCP server.

Address allocation decisions are integral to the allocation of
addresses and prefixes in DHCP. The argument is described in details
in {{RFC7969}}, here we want to guarantee that 4o6RA does not
break any legacy capability when related to the use of topology.

In the scenario described in {{RFC7341}} the DHCPv6 Relay Agent knows
the interface where the encapsulated DHCP request is received.

Moving 4o6 in the intermediate node rather than at the client breaks the topology
propagation as 4o6RA-only does not provide any interface information in the encapsulated message.

~~~aasvg

           .-----------.     .-------------------------.
          | L2 Network  |   |        IPv6 Network       |
 +--------+-+         +-+---+---+    +--------+       +-+--------+
 |  DHCPv4  |         |  4o6    |    | DHCPv6 |       | DHCP 4o6 |
 |  Client  +---------+  RA     +----+   RA   +-------+  Server  |
 |          |         |         |    |        |       |          |
 +--------+-+         +-+---+---+    +--------+       +-+--------+
          |             |   |                           |
           '-----------'     '-------------------------'

~~~
{: #fig_4o6RA_RA title="Topology broken path" artwork-align="center"}

As shown in {{fig_4o6RA_RA}}, the introduction of 4o6 at the
edge of the IPv6 network hides the L2 network from the DHCPv6 RA.

In order to preserve the topology information, it is recommended that the
implementation of 4o6RA is combined with the implementation of LDRA {{RFC6221}}
and that the implementation has a mechanism for LDRA to get interface information
that can be used for the Interface-ID option, as specified in
{{Section 5.3.2 of RFC6221}}.
The internal mechanisms to exchange interface information,
their format and whether the interface information contains an indication that a 4o6RA
is involved are out of the scope for this document.

~~~aasvg

           .-----------.     .---------------------------.
          | L2 Network  |   |          IPv6 Network       |
 +--------+-+         +-+---+--+---------+      +----------+
 |  DHCPv4  |         |  4o6   |  LDRA   |      | DHCP 4o6 |
 |  Client  +---------+  RA    + RFC6221 +------+  Server  |
 |          |         |        |         |      |          |
 +--------+-+         +-+---+--+---------+      +----------+
          |             |   |                             |
           '-----------'     '---------------------------'

~~~
{: #fig_4o6LDRA title="Topology path preserved with LDRA" artwork-align="center"}

The assumed architecture is shown in {{fig_4o6LDRA}} where the whole
RA is built up with cooperating 4o6RA and LDRA, and an internal interface to
propagated topology information from 4o6RA to LDRA.

In a simple case, where the same node hosts teh 4o6RA and the DHCP4o6 server,
it might be enough to only use 4o6RA, as shown in {{fig_4o6RAserver}}.

~~~aasvg

           .-----------.
          | L2 Network  |
 +--------+-+         +-+------+----------+
 |   DHCP   |         |  4o6   | DHCP 4o6 |
 |  Client  +---------+  RA    +  Server  |
 |  on CPE  |         |        |          |
 +--------+-+         +-+------+----------+
          |             |
           '-----------'

~~~
{: #fig_4o6RAserver title="Topology path preserved 4o6 RA in DHCP server" artwork-align="center"}

# Deployment Considerations

As the client is not aware of the 4o6RA, the network deployment needs to ensure that
all DHCPv4 broad- and unicast messages from the client are routed over the 4o6RA.
This can e.g. be achieved by placing the 4o6RA in a cetral position that can observe all traffic
from the clients or use of address translation with the 4o6RA address for unicast
messages.

# Security Considerations {#seccons}

This documents applies 4o6 DHCP in a scenario where legacy IPv4 clients are
connected to 4o6 DHCP Relay Agent that performs the en- and decapsulation. This document
does not change anything else in the 4o6 DHCP specification and therefore the
security consideration of {{RFC7341}} still apply.

The mechanism described differs from {{RFC7341}} as the DHCP client
actually sends and receveis DHCP messages, whereas in {{RFC7341}} it
only sends DHCPv6 messages. This makes it possible that
DHCP messages could reach a DHCP server without using the 4o6RA.
While this can cause errornous states in both the client and DHCP server
and potentially even lead to misconfigrations that impact reachability,
this is not seen as a security concern rather than a deloyment error.

More generally, the legacy IPv4 client is not aware of this mechanism, however, even
when 4o6 DHCP is used, the client does not have any control about the
information provided by the Relay agent. As such this change does not
raise any additional security concerns.


# IANA Considerations

This document has no IANA actions.


--- back

# Example Use Case: Topology Discovery for IPv4-only Radio Unit in the RAN Switched Fronthaul {#usecase}

In Radio Access Networks (RANs) the Fronthaul is the network segment
that connects Radio Units, the distributed radio elements in a mobile network,
to other network elements. The aggregation
of Radio Unit devices (also known as Switched Fronthaul) hides the relationship
between the Radio Units themselves and the physical ports where they are connected.
The Radio Units are the client hosts in the switched Fronthaul network and
need to be configured based on their Topology.

~~~aasvg
     +--------+
     |  RU1   |     P1 +-+------+     |                   |
     |        +--------| | L2RA |     |  +----+------+    |
     +--------+        | +------+     |  |    | L3RA |    |
                       |  L2    |     +--|    +------+    |
     +--------+     P2 | switch |     |  |           |    |
     |  RU2   +--------|  #1    +-----|  |   Router  +----|
     |        |        +--------+     |  +-----------+    |  +---------+
     +--------+                       |                   |  |         |
                                      |                   +--| DHCP    |
     +--------+                       |                   |  | Server  |
     |  RU3   |     P1 +-+------+     |                   |  |   #1    |
     |        +--------| | L2RA |     |  +-----------+    |  +---------+
     +--------+        | +------+     |  |           |    |
                       |  L2    |     +--| Baseband  |    |
     +--------+     P2 | switch |     |  |   Unit    |    |
     |  RU4   +--------|  #2    +-----|  |           +----|
     |        |        +--------+     |  +-----------+    |
     +--------+                       |                   |
~~~
{: #l2_switched_fh title="Layer 2 Switched Fronthaul Example" artwork-align="center"}

{{l2_switched_fh}} shows multiple Radio Units that are connected
to one Baseband Unit by means of a Layer 2 switched network.
The Baseband Unit is the central processing unit that handles baseband information.
A Baseband Unit is often placed rather centrally, while the Radio Units need to
be distributed to be co-located with or near the antennas.
Traffic between Radio Units and Baseband Units is both IP-based and Layer-2-based
and may pass a hierarchy of L2 switches.

In order to properly address the Radio Unit, the Baseband Unit needs to associate
the Radio Unit's MAC address to the L2 switch and respective port
where the Radio Unit is connected. To realize this device configuration
in the Switched Fronthaul network, DHCPv6 can be used to discover the network Topology.

With the L2 switched network between the clients and the server,
one of the clients is responsible for the configuration of the other
clients based on their topology. Updating of the software on the clients
is not possible often not possible and clients may be IPv4-only.

# Acknowledgments
{:numbered="false"}

The authors would also like to acknowledge interesting discussions in
this problem space with Sarah Gannon, Ines Ramadza and Siddharth Sharma.
