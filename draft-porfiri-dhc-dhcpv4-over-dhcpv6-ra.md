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
   RFC8415:

informative:
   RFC0951:
   RFC1542:
   RFC2132:
   RFC2131:

--- abstract

IPv4 connectivity is still needed as networks migrate towards IPv6.
Users require IPv4 configuration even if the uplink to their service
provider supports IPv6 only.
A mechanism exists for obtaining IPv4 configuration information dynamically
in IPv6 networks by carrying DHCPv4 messages over DHCPv6 transport.
This document describes how that mechanism is to be deployes in
Relay Agents.

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
specific configuration parameters across IPv6-only networks.

The deployment of {{RFC7341}} requires implementation in all
the legacy DHCP clients and at the DHCPv6 server.

In some cases, updating the clients may be not feasible due to
a number of technical or business reasons.

This document describes how the mechanisms needed for deploying
a {{RFC7341}} compliant solution can be implemented at the intermediate
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

* DHCPv4 over DHCPv6 (or 4o6)
   The architecture, the procedures and the protocols described in the
   DHCPv4-over-DHCPv6 document {{RFC7341}}.

* DHCPv4 over DHCPv6 Relay Agent (or 4o6RA)
   The 4o6 Relay Agent is the part of an LDRA implementing 4o6

* DHCP Relay Agent (or RA)

  This is a concept in all of the protocols, BOOTP {{RFC0951}} {{RFC1542}}, DHCPv4
  {{RFC2131}} {{RFC2132}}, and DHCPv6 {{RFC8415}}, although the details differ
  between the protocols.

* Lightweight DHCPv6 Relay Agent (or LDRA)

  This is an extension of the original DHCPv6 Relay Agent mechanism,
  to support also Layer 2 devices performing a Relay Agent function {{RFC6221}}.

* Relay Agent Information Option (or RAIO)

   This is a DHCP option defined in {{RFC3046}}. Also commonly referred
   to as "Option 82". RAIO options were later extended to be able to
   carry suboptions {{RFC6925}}.

{::boilerplate bcp14-tagged}

# Applicability {#applicability}

The mechanism described in this document is not universally
applicable.  This is intended as a special-purpose mechanism that
will be implemented where nodes that must obtain IPv4 configuration
information using DHCPv4 in specific environments exist and native DHCPv4
is not available. This mechanism may be enabled using an administrative
control, automatically or by other means that are beyond
the scope of this document.

# Architecture overview {#architecture_overview}

The architecture described here addresses a typical use case, where a
DHCP client's uplink supports IPv6 only and the Service Provider's
network supports IPv6 and limited IPv4 services.  In this scenario,
the client can only use the IPv6 network to access IPv4 services, so
IPv4 services must be configured using IPv6 as the underlying network
protocol.

Although the purpose of this document is to address the problem of
communication between the DHCPv4 client and the DHCPv4 server, the
mechanism that it describes does not restrict the transported
messages types to DHCPv4 only.  As the DHCPv4 message is a special
type of BOOTP message, BOOTP messages {{RFC0951}} MAY also be
transported using the same mechanism.

DHCP clients may be running on CPE devices, end hosts, or any other
device that supports the DHCP-client function.  This document uses
the CPE as an example for describing the mechanism.  This does not
preclude any end host, or other device requiring IPv4 configuration,
from implementing DHCPv4 over DHCPv6 in the future.

The basic mechanism for 4o6 is described in {{RFC7341}}, where
{{architecture_overview_fig1}} describes the mechanisms applicability
of {{RFC7341}}. The mechanism is described in details in section
5 of {{RFC7341}}.

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

The current document applies when the CPE cannot be upgraded, so that
4o6 cannot be implemented as strictly described in {{RFC7341}}.
The Architecture for the resulting case is shown in {{architecture_overview_fig2}}.

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

In {{architecture_overview_fig2}} the implementation of the encapsulation
and decapsulation described in {{RFC7341}} is accomplished in the Relay Agent
whereas the DHCP Client does not require any change.

All prerequisites and configuration that in section 5 of {{RFC7341}}
apply to the DHCP client shall be applied to the 4o6RA instead.

## DHCP scope limitation {#dhcpv4_scope}

Being the 4o6 mechanism implemented in the Relay Agent, special care
needs to be done at the nework node implementing the Relay Agent itself
when dealing with DHCP termination.
In order the 4o6 mechanism to work properly, it shall not be possible
for DHCP traffic generated from the DHCP client to reach any DHCP server
except by using the 4o6RA.
As a result, all the DHCP messages generated from the client MUST be
received only by the 4o6 Relay Agent. The rule applies both to the
configuration of the node implementing the 4o6RA and the L2 network
where DHCP client and 4o6RA are connected to.

## Topology considerations {#topology_considerations}

The scenario depicted in {{architecture_overview_fig1}} preserves the topology
infomation as the DHCPv6 Relay Agent has the knowledge of the interface where the
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
agent is involved are out of the scope of the present document.

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

## Considerations about 4o6RA deployment

Deployment of 4o6RA depends on the network architecture and the scope
of the node where the functionality is implemented.

In a simple case, where the same node hosts 4o6RA and the DHCP4o6 server,
it may be just enough deploying 4o6RA itself, bu in the general case
and in order to preserve the network topology information, it is
recommended the 4o6RA to be deployed in combination with a LDRA
as shown in {{architecture_overview_fig4}}.

## Considerations about DHCPv6 server {#dhcpv6_server}

The DHCPv6 server shall be compliant with 4o6 according to {{RFC7341}}.

## Considerations about L2 terminations at 4o6 {#l2_terminations}

Deploying 4o6RA at the network edge requires care in the network
design as well as in the desing of the 4o6RA parser.

The network configuration shall guarantee that no DHCP server
are reachable from the DHCP client and that at least one 4o6RA
can be reached.


# Security Considerations {#seccons}

This documents applies 4o6 DHCP in a scenario where legacy IPv4 clients are
connected to 4o6 DHCP Relay Agent that performs the en- and decapsulation. This document
does not change anything else in the 4o6 DHCP speacification and therefore the
security consideration of {{RFC7341}} still apply.

The legacy IPv4 client is not aware of this mechanism, however, even
when 4o6 DHCP is used, the client does not have any control about the
information provided by the Relay agent. As such this change does not
raise any additional secruity concerns.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments


The authors would also like to acknowledge interesting discussions in
this problem space with Sarah Gannon, Ines Ramadza and Siddharth Sharma.

# Topology Discovery at the RAN

## Example Use Case: Switched Fronthaul  {#usecase}

This section describes a case where topology knowledge is needed for
properly configuring the node. The case comes from a change of topology
by inserting a L2 switched network between the clients and the server.
One of the clients is responsible for the configuration of the other
clients based on their topology. Updating of SW on the clients
is not possible.

### Topology Based Configuration of RU

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

### Existing DHCP-based Solutions for Topology Discovery {#existing}

This section describes how existing technology can be used for providing topology information.

#### IPv6 Clients using DHCPv6 {#l2discipv6}

If the network is fully IPv6 enabled, DHCPv6 {{RFC8415}} can be used for Topology Discovery.
This solution uses DHCPv6 Relay Agent support in the server, whilst Lightweight DHCPv6
Relay Agents (LDRA) {{RFC6221}} are implemented in the L2 switches to inform DHCPv6 server
about the L2 Topology.

#### Clients with Dual Connectivity and 4o6 DHCP support {#l2discipv4}

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

## DHCPv4 over DHCPv6 in the Relay Agent {#dhcpv4v6ra}

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

### Layer 2 Topogoy Discovery using 4o6 DHCP with legacy IPv4 clients {#l2discipv44o6leg}

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

{:numbered="false"}
