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

This document describes a general mechanism for networks
with legacy IPv4-only clients to use services provided by
DHCPv6 DHCPv4-over-DHCPv6 (DHCP 4o6) like for instance
discovering information about network Topology.
To address this scenario, this document specifies 
a RFC7341-based approach that allows DHCP 4o6 to be deployed as a
Relay Agent (4o6RA) where 4o6 DHCP en- and decapsulation
will be implemented even when this is not possible at the client.

--- middle

# Introduction {#introduction}

As the migration towards IPv6 continues, IPv6-only networks will
become more prevalent.  In such networks, IPv4 connectivity will
continue to be provided as a service over IPv6-only networks.  In
addition to provisioning IPv4 addresses for clients of this service,
other IPv4 configuration parameters may also be needed (e.g.,
addresses of IPv4-only services).

The transport mechanism for carrying DHCPv4
messages using the DHCPv6 protocol is described in {{RFC7341}}
for the dynamic provisioning of IPv4 addresses and other DHCPv4
specific configuration parameters across IPv6-only networks
However, the deployment of {{RFC7341}} requires implementation in
all DHCP clients and at the DHCPv6 server, as shown in {{architecture_overview_fig1}}.
In some cases, updating the clients may be not feasible due to
a number of technical or business reasons.

~~~aasvg

                 .-----------.             .-----------.
                |             |           |             |
       +--------+-+  IPv6   +-+-----------+-+  IPv6   +-+--------+
       | DHCP 4o6 | Network |    DHCPv6     | Network | DHCP 4o6 |
       |  Client  +---------+  Relay Agent  +---------+  Server  |
       |  on CPE  |         |               |         |          |
       +--------+-+         +-+-----------+-+         +-+--------+
                |             |           |             |
                 '-----------'             '-----------'

~~~
{: #architecture_overview_fig1 title="RFC7341 Architecture Overview" artwork-align="center"}

Similarly, the specifications for DHCPv6 Relay Agents such as LDRA {{RFC6221}}
or L3RA {{RFC8415}} do not foresee the possibility to handle legacy DHCP,
other than implementing 4o6 in client.

This document specifies an {{RFC7341}}-based solution that can be implemented in intermediate
nodes such as L2 switches or routers, without putting any requirement
on clients.


# Conventions and Definitions

The following terms and acronyms are used in this document:

* CPE:
      Customer Premises Equipment (also known as Customer Provided
      Equipment), which provides access for devices connected to a Local
      Area Network (LAN), typically at the customer's site/home, to the
      Internet Service Provider's (ISP's) network.

* DHCP 4o6 client (or client):
      A DHCP client supporting both the DHCPv6 protocol [RFC3315] as
      well as the DHCPv4 over DHCPv6 protocol described in this
      document.  Such a client is capable of requesting IPv6
      configuration using DHCPv6 and IPv4 configuration using DHCPv4
      over DHCPv6.

* DHCP 4o6 server (or server):
      A DHCP server that is capable of processing DHCPv4 packets
      encapsulated in the DHCPv4 Message option (defined below).

* DHCPv4 over DHCPv6 (or 4o6):
   The architecture, the procedures and the protocols described in the
   DHCPv4-over-DHCPv6 document {{RFC7341}}.

* DHCPv4 over DHCPv6 Relay Agent (or 4o6RA):
   The 4o6 Relay Agent is the part of an LDRA implementing 4o6

* DHCP Relay Agent (or RA):

  This is a concept in all of the protocols, BOOTP {{RFC0951}} {{RFC1542}}, DHCPv4
  {{RFC2131}} {{RFC2132}}, and DHCPv6 {{RFC8415}}, although the details differ
  between the protocols.

* Lightweight DHCPv6 Relay Agent (or LDRA):

  This is an extension of the original DHCPv6 Relay Agent mechanism,
  to support also Layer 2 devices performing a Relay Agent function {{RFC6221}}.

* LLDP:
   Link Layer Discovery Protocol

* Relay Agent Information Option (or RAIO):

   This is a DHCP option defined in {{RFC3046}}. Also commonly referred
   to as "Option 82". RAIO options were later extended to be able to
   carry suboptions {{RFC6925}}.

{::boilerplate bcp14-tagged}

# Architecture overview

This document assume an architecture, where a
DHCP client's uplink network supports IPv6 only and the Service Provider's
network supports IPv6 and limited IPv4 services.  In this scenario,
the client can only use the IPv6 network to access IPv4 services, so
IPv4 services must be configured using IPv6 as the underlying network
protocol.

DHCP clients may be running on CPE devices, end hosts, or any other
device that supports the DHCP-client function.  This document uses
the CPE as an example for describing the mechanism.  This does not
preclude any end host, or other device requiring IPv4 configuration,
from implementing DHCPv4 over DHCPv6 in the future.

The current document applies when the CPE cannot be upgraded, so that
4o6 cannot be implemented as specified in {{RFC7341}}

To address such a network setup, this document proposes to extend the features of all DHCPv6 Relay Agents
by the addition of DHCPv4 over DHCPv6 feature, thus providing the
en- and decapsulation at the Relay Agent rather than at the client, as
shown in {{architecture_overview_fig2}}.

~~~aasvg

                 .-----------.             .-----------.
                |             |           |             |
       +--------+-+    L2   +-+-----------+-+  IPv6   +-+--------+
       |   DHCP   | Network |    DHCPv6     | Network | DHCP 4o6 |
       |  Client  +---------+  Relay Agent  +---------+  Server  |
       |  on CPE  |         |   with 4o6RA  |         |          |
       +--------+-+         +-+-----------+-+         +-+--------+
                |             |           |             |
                 '-----------'             '-----------'

~~~
{: #architecture_overview_fig2 title="Architecture Overview with legacy DHCP client" artwork-align="center"}

Thi document specifes the encapsulation
and decapsulation described in {{RFC7341}} to be performed in the Relay Agent
whereas the DHCP Client does not require any change.
In this case it is up to the Relay Agent to provide the full
4o6 DHCP set of functionality whereas the legacy client is not aware of being served
via a 4o6 DHCP service.
All prerequisites and configuration that in section 5 of {{RFC7341}}
apply to the DHCP client shall be applied to the 4o6RA instead.

This extended 4o6 Relay Agent (4o6RA) exchanges DHCP messages
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

# Deployment Considerations

## DHCPv6 server {#dhcpv6_server}

The DHCPv6 server must be compliant with 4o6 according to {{RFC7341}}.
No additional requirements on DHCPv6 server are needey by this specification.

## Reachability {#network_design}

In order to make 4o6RA behave properly, the L2 network connecting
CPEs shall not allow DHCP traffic to reach any DHCP server directly.
Furthermore, at least one 4o6RA shall be reachable in that
L2 network so that the reacheability of a DHCP server is granted
by means of 4o6RA.

## L2 terminations at 4o6RA {#l2_terminations}

The 4o6RA must be able to process all types of DHCP requests and replies.

## Topology  {#topology_considerations}

The DHCPv4 {{RFC2131}} and DHCPv6 {{RFC3315}} protocol specifications
describe how addresses can be allocated to clients based on network
topology information provided by the DHCP relay infrastructure.
Address allocation decisions are integral to the allocation of
addresses and prefixes in DHCP. The argument is described in details
in {{RFC7969}}, here we want to guarantee that 4o6RA doesn't
break any legacy capability when related to the use of topology.

The scenario depicted in {{architecture_overview_fig1}} preserves the topology
information as the DHCPv6 Relay Agent has the knowledge of the interface where the
encapsulated DHCP request comes from.

Moving 4o6 in the intermediate node rather than at the client breaks the topology
propagation as 4o6 doesn't forward the interface information in the encapsulated message.

~~~aasvg

           .-----------.     .-------------------------.
          | L2 Network  |   |        IPv6 Network       |
 +--------+-+         +-+---+---+    +--------+       +-+--------+
 |   DHCP   |         |  4o6    |    | DHCPv6 |       | DHCP 4o6 |
 |  Client  +---------+  RA     +----+   RA   +-------+  Server  |
 |  on CPE  |         |         |    |        |       |          |
 +--------+-+         +-+---+---+    +--------+       +-+--------+
          |             |   |                           |
           '-----------'     '-------------------------'

~~~
{: #architecture_overview_fig3 title="Topology broken path" artwork-align="center"}

As shown in {{architecture_overview_fig3}}, the introduction of 4o6 at the
edge of the IPv6 network hides the L2 network from the DHCPv6 Relay agent.

In order to preserve the topology information, it is recommended that the
implementation of 4o6RA is combined with the implementation of LDRA {{RFC6221}}
and that the implementation has a mechanism for LDRA to get interface information
so that that they can be properly used for Interface-ID option as specified in
section 5.3.2 of {{RFC6221}}.
The internal mechanisms of an implementation for transporting the interface information,
their format and whether the interface information contains indication that a 4o6 RA
agent is involved are out of the scope for this document.

~~~aasvg

           .-----------.     .---------------------------.
          | L2 Network  |   |          IPv6 Network       |
 +--------+-+         +-+---+---+      +---------+      +-+--------+
 |   DHCP   |         |  4o6    |      |  LDRA   |      | DHCP 4o6 |
 |  Client  +---------+  RA     +------+ RFC6221 +------+  Server  |
 |  on CPE  |         |         |      |         |      |          |
 +--------+-+         +-+---+-+-+      +---------+      +-+--------+
          |             |   | | interface ^               |
          |             |   | |   info    |               |
          |             |   |  '---------'                |
           '-----------'     '---------------------------'

~~~
{: #architecture_overview_fig4 title="Topology path preserved with LDRA" artwork-align="center"}

The recommended architecture is shown in {{architecture_overview_fig4}} where the whole
Relay Agent is built up with cooperating 4o6 and LDRA, and interface information is
propagated from 4o6 to LDRA with implementation specific mechanism.

In a simple case, where the same node hosts 4o6RA and the DHCP4o6 server,
it may be just enough deploying 4o6RA itself, but in the general case
and in order to preserve the network topology information, it is
recommended the 4o6RA to be deployed in combination with a LDRA
as shown in {{architecture_overview_fig4}}.

# Topology Discovery with 4o6RA

The reason why Topology is important is well described in {{RFC7969}},
among the motivation there's the possibility to provide a dedicated
configuration based on where and how the CPE is connected to the
network. The {{RFC7969}} also describes how topology information is made available
at the DHCP Server in IPv4 and IPv6 networks.

Besides the cases where topology is used directly at the DHCP Server,
we also consider cases where the Topology is used by a third agent that
queries the DHCP Server later than in the DHCP configuration phase.
In this case, the agent receives the IP address information of the CPE
and then queries the DHCP server using that IP address for obtaining
the topology information.

## IPv6 Clients using DHCPv6 {#l2discipv6}

If the network is fully IPv6 enabled, DHCPv6 {{RFC8415}} can be used for Topology Discovery.
This solution uses DHCPv6 Relay Agent support in the server, whilst Lightweight DHCPv6
Relay Agents (LDRA) {{RFC6221}} are implemented in the L2 switches to inform DHCPv6 server
about the L2 Topology.

## Clients with Dual Connectivity and 4o6 DHCP Support {#l2discipv4}

If CPE needs an IPv4 address, it is connected to both IPv4 and IPv6 and supports
DHCPv4-over-DHCPv6 {{RFC7341}}, DHCPv6 {{RFC8415}} with a DHCPv4-over-DHCPv6 compliant DHCP server
can be used for Topology Discovery whereas DHCP is used still for IP address assignment.

An example network with 4o6 compliant clients can be sketched as follows:

~~~aasvg
     +---------+
     |   4o6   |    P1 +-+------+     |                   |  +---------+
     |   CPE   +-------| | LDRA |     |  +----+------+    |  |  4o6    |
     +---------+       | +------+     |  |    | L3RA |    +--|  DHCP   |
                       |  L2    |     +--|    +------+    |  | Server  |
     +---------+    P2 | switch |     |  |           |    |  |         |
     |   4o6   +-------|        +-----|  |   Router  +----|  +---------+
     |   CPE   |       +--------+     |  +-----------+    |
     +---------+                      |                   |


~~~
{: #l2_switched_4o6 title="Layer 2 Topology discover with 4o6" artwork-align="center"}

In {{l2_switched_4o6}} the client supports {{RFC7341}} by implementing the 4o6 encapsulation
whereas the intermediate nodes implement LDRA {{RFC6221}} or L3RA {{RFC8415}} and finally
the DHCP server is 4o6 DHCP capable {{RFC7341}}.

## IPv4-only Clients using 4o6RA

If the CPE is IPv4 and is connected to a DHCP Server via the 4o6RA
as shown in {{architecture_overview_fig4}}, the DHCP 4o6 Server
takes the aggregate information of the Topology for both IPv4
and IPv6 segments of the network.

Ane example of such scenario is depicted here:

~~~aasvg
     +---------+
     |   4o6   |    P1 +-+------+     |                   |  +---------+
     |   CPE   +-------| | LDRA |     |  +----+------+    |  |  4o6    |
     +---------+       | +------+     |  |    | L3RA |    +--|  DHCP   |
                       |  L2    |     +--|    +------+    |  | Server  |
     +---------+       | switch |     |  |           |    |  +---------+
     | Legacy  |    P2 +------+ +-----|  |   Router  +----|
     |   CPE   +-------+ 4o6  | |     |  +-----------+    |  +---------+
     +---------+       +------+-+     |                   |  | Legacy  |
                                                          +--|  DHCP   |
                                                          |  | Server  |
                                                          |  +---------+
~~~
{: #l2_switched_4o6_leg title="Layer 2 architecture with 4o6 and legacy client" artwork-align="center"}

If the DHCP 4o6 Server is built up with standalone DHCP and DHCPv6
servers as shown in {{l2_switched_4o6_leg}}, each of them knows only part of the Topology.
In such case a third agent willing to provide the configuration to CPE
must be aware about the network
and query both DHCP and DHCPv6 server in order to get the complete
topology for configuration.

## Other configuration alternatives

A third agent may also use other mechanism than DHCP for Topology
Discovery such as LLDP, such mechanism requires implementation at
the CPE and is not considered in this document.

# Security Considerations {#seccons}

This documents applies 4o6 DHCP in a scenario where legacy IPv4 clients are
connected to 4o6 DHCP Relay Agent that performs the en- and decapsulation. This document
does not change anything else in the 4o6 DHCP specification and therefore the
security consideration of {{RFC7341}} still apply.

The legacy IPv4 client is not aware of this mechanism, however, even
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
