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
    email: mirja.kuhlewind@ericsson.com

normative:
   RFC3046:
   RFC6221:
   RFC6925:
   RFC7341:
   RFC8415:

informative:
   RFC0951:
   RFC1542:
   RFC2132:
   RFC2131:

--- abstract

This document describes a general mechanism for networks
with legacy IPv4-only clients to use all services provided by
DHCPv6 DHCPv4-over-DHCPv6 (DHCP 4o6) like for instance
discovering information about network Topology.
To address this scenario, this document specifies an amendment
to RFC7341 that allows DHCP 4o6 to be integrated in any DHCPv6
Relay Agent (4o6RA) where 4o6 DHCP en- and decapsulation
will be implemented even when this is not possible at the client.

--- middle

# Introduction {#introduction}

In some networks it may be needed to have functionality provided by DHCPv6
such as topology discovery, but having the need to provide legacy IPv4
addresses and, at the same time, clients that for various reasons
cannot be updated for implementing DHCPv4 over DHCPv6 {{RFC7341}} or
any other feature.

As an example, in some networks the configuration of a client host depends
on the Topology.
However, when a new client host gets connected to the network, it may be unaware of
the Topology and respectively how it has to be configured.

In IPv6 networks, Topology discover can be realized using DHCPv6 Relay Agents {{RFC6221}}
that insert relay agent options in DHCPv6 message exchanges in order to identify
the client-facing interfaces, e.g. using the physical Port Number or other
hardcoded information.
Then, a reference host that is responsible for providing configuration to the client
host can obtain Topology information from the DHCP server.

In DHCPv6, a Relay Agent can encapsulate the DHCP message from the client in a new
DHCP message along with any options it chooses to add to provide information to the DHCP server.
This mode of operation also supports networks that include a hierarchy of switches.

However, if the client only supports IPv4 and cannot easily be replaced or updated,
this approach does not work, as DHCPv4 support for relay agents is much more
limited. For instance, there is no support in DHCPv4 for hierarchical
modes of deployment, as the specifications prohibit chaining of Relay
Agent Information Options (RAIOs) {{RFC3046}}.

A typical example where Topology Discovery is needed for host configuration is
the switched fronthaul in the Radio Access Network (see {{usecase}}).
However, the specified approach in this document is not limited to that example.

This document specifies how to provide Topology Discover using
Relay Agent functionality for legacy IPv4 clients using DHCPv4-over-DHCPv6
(DHCP 4o6) {{RFC7341}}. No new protocols or extensions are needed, instead
this document specifies an amendment to {{RFC7341}} that allows any Relay Agent
to perform the 4o6 DHCP en- and decapsulation instead of the client.


# Conventions and Definitions

The following terms and acronyms are used in this document:

* 4o6
   The architecture, the procedures and the protocols described in the
   DHCPv4-over-DHCPv6 document {{RFC7341}}.

* 4o6RA
   The 4o6 Relay Agent is the part of an LDRA implementing 4o6

* Baseband Unit (BB)

   This is the 3GPP defined node implementing the RU control
   and the mobility in a Mobile Network

* DHCP Relay Agent

  This is a concept in all of the protocols, BOOTP {{RFC0951}} {{RFC1542}}, DHCPv4
  {{RFC2131}} {{RFC2132}}, and DHCPv6 {{RFC8415}}, although the details differ
  between the protocols.

* Fronthaul Network (FH)

   This is the 3GPP defined section of network connecting RU and BB

* Lightweight DHCPv6 Relay Agent (LDRA)

  This is an extension of the original DHCPv6 Relay Agent mechanism,
  to support also Layer 2 devices performing a Relay Agent function {{RFC6221}}.

* Relay Agent Information Option (RAIO)

   This is a DHCP option defined in {{RFC3046}}. Also commonly referred
   to as "Option 82". RAIO options were later extended to be able to
   carry suboptions {{RFC6925}}.

* Radio Access Network (RAN)

   This is the part of the Network connecting nodes specified by 3GPP
   being part of a Mobile Network.

* Radio Unit (RU)

   This is the 3GPP defined node implementing Radio transmission and
   reception function within a Mobile Network

{::boilerplate bcp14-tagged}


# Example Use Case: Switched Fronthaul  {#usecase}

This section describes a case where topology knowledge is needed for
properly configuring the node. The case comes from a change of topology
by inserting a L2 switched network between the clients and the server.
One of the clients is responsible for the configuration of the other
clients based on their topology. Updating of SW on the clients
is not possible.

## Topology Based Configuration of RU

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
Traffic between Radio Units and Bandband Units is both IP-based and Layer-2-based
and may pass a hierarchy of L2 switches.

In order to properly address the Radio Unit, the Baseband Unit needs to associate
the Radio Unit's MAC address to the L2 switch and respective port
where the Radio Unit is connected. To realize this device configuration
in the Switched Fronthaul network, DHCPv6 can be used to discover the network Topology.

## Existing DHCP-based Solutions for Topology Discovery {#existing}

This section describes how existing technology can be used for providing topology information.

### IPv6 Clients using DHCPv6 {#l2discipv6}

If the network is fully IPv6 enabled, DHCPv6 {{RFC8415}} can be used for Topology Discovery.
This solution uses DHCPv6 Relay Agent support in the server, whilst Lightweight DHCPv6
Relay Agents (LDRA) {{RFC6221}} are implemented in the L2 switches to inform DHCPv6 server
about the L2 Topology.

### Clients with Dual Connectivity and 4o6 DHCP support {#l2discipv4}

If the client needs an IPv4 address, is connected to both IPv4 and IPv6 and supports
DHCPv4-over-DHCPv6 {{RFC7341}}, DHCPv6 {{RFC8415}} with a DHCPv4-over-DHCPv6 compliant DHCP server
can be used for Topology Discovery whereas DHCP is used still for IP address assignment.

An example network with 4o6 compliant clients can be sketched as follows:

~~~aasvg
     +---------+
     |   4o6   |    P1 +-+------+     |                   |  +---------+
     | Client  +-------| | LDRA |     |  +----+------+    |  |  4o6    |
     +---------+       | +------+     |  |    | L3RA |    +--|  DHCP   |
                       |  L2    |     +--|    +------+    |  | Server  |
     +---------+    P2 | switch |     |  |           |    |  |         |
     |   4o6   +-------|        +-----|  |   Router  +----|  +---------+
     | Client  |       +--------+     |  +-----------+    |
     +---------+                      |                   |


~~~
{: #l2_switched_4o6 title="Layer 2 Topology discover with 4o6" artwork-align="center"}

In {{l2_switched_4o6}} the client supports {{RFC7341}} by implementing the 4o6 encapsulation
whereas the intermediate nodes implement LDRA {{RFC6221}} or L3RA {{RFC8415}} and finally
the DHCP server is 4o6 DHCP capable {{RFC7341}}.

Still, {{RFC7341}} does not provide a solution for legacy IPv4 clients
that respectively do not support 4o6 encapsulation.

# DHCPv4 over DHCPv6 in the Relay Agent {#dhcpv4v6ra}

The current specifications for DHCPv6 Relay Agents such as LDRA {{RFC6221}}
or L3RA {{RFC8415}} doesn't foresee the possibility to handle legacy DHCP,
on the other hand this can be solved at the client as described in {{l2discipv4}}
when possible.

The specification for DHCPv4 over DHCPv6 {{RFC7341}} does only foresee the case
where en- and decapsulation are accomplished at the client.

This document proposes to extend the features of all DHCPv6 Relay Agents
by the addition of DHCPv4 over DHCPv6 feature, thus providing the
en- and decapsulation at the Relay Agent rather than at the client.

The proposal is aimed at solving all cases where a SW update of the DHCP client
is not possbile for any reason, still providing the same features as
described in {{RFC7341}}.

## Layer 2 Topogoy Discovery using 4o6 DHCP with legacy IPv4 clients {#l2discipv44o6leg}

This section provides an example of how the topology discovery use case
proposed in {{usecase}} can be solved by having the DHCPv4 over DHCPv6 feature
at the LDRA Agent.

~~~aasvg
     +---------+
     |   4o6   |    P1 +-+------+     |                   |  +---------+
     | Client  +-------| | LDRA |     |  +----+------+    |  |  4o6    |
     +---------+       | +------+     |  |    | L3RA |    +--|  DHCP   |
                       |  L2    |     +--|    +------+    |  | Server  |
     +---------+       | switch |     |  |           |    |  +---------+
     | Legacy  |    P2 +------+ +-----|  |   Router  +----|
     | Client  +-------+ 4o6  | |     |  +-----------+    |  +---------+
     +---------+       +------+-+     |                   |  | Legacy  |
                                                          +--|  DHCP   |
                                                          |  | Server  |
                                                          |  +---------+
~~~
{: #l2_switched_4o6_leg title="Layer 2 architecture with 4o6 and legacy client" artwork-align="center"}

The new scenario, not described in {{RFC7341}}, is shown in {{l2_switched_4o6_leg}}.
In such a scenario, the 4o6 encapsulation is implemented in the Relay Agent deployed
in the edge L2 switch, or in general in the edge device providing connectivity
to the legacy client. In this case it is up to the Relay Agent to provide the full
4o6 DHCP set of functionality whereas the legacy client is not aware of being served
via a 4o6 DHCP service.

This extended LDRA with 4o6 Relay Agent (4o6RA), exchanges DHCP messages
between clients and servers using the message formats established in {{RFC8415}}.
To maintain interoperability with existing DHCP relays and servers,
the message format is unchanged from {{RFC8415}}. The 4o6RA implements
the same message types as a normal DHCPv6 Relay Agent {{Section 6 of RFC7341}}. They are:
-  Relay-Forward Messages
-  Relay-Reply Messages

In this specification, the 4o6RA creates the DHCPV4-QUERY Message
and encapsulates the DHCP request message received from the legacy DHCPv4 client.

When DHCPV4-RESPONSE Message is received by the 4o6 Relay Agent,
it looks for the DHCPv4 Message option within this message.
If this option is not found, the DHCPv4-response message MUST be discarded.
If the DHCPv4 Message option is present, the 4o6RA MUST extract the DHCPv4
message and forward the encapsulated DHCPv4-response to the legacy DHCPv4 client.

Any Layer 2 Relay Agent receiving DHCPV4-QUERY or DHCPV4-RESPONSE messages
will handle them as specified in Section 6 of {{RFC6221}}.

# Security Considerations {#seccons}

This documents applies 4o6 DHCP in a scenario where legacy IPv4 clients are
connected to 4o6 DHCP Relay Agent that performs the en- and decapsulation. This document
does not change anything else in the 4o6 DHCP speacification and therefore the
security consideration of {{RFC7341}} still apply.

The legacy IPv4 client is not aware of this mechanism, however, even
when 4o6 DHCP is used, the client does not have any control about the
information provided by the Relay agent. As such this change does not
provide any additional secruity concerns.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}


The authors would also like to acknowledge interesting discussions in
this problem space with Sarah Gannon, Ines Ramadza and Siddharth Sharma.
