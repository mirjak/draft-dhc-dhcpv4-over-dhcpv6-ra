---
title: "DHCPv4-over-DHCPv6 (DHCP 4o6) with Relay Agent Support"
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
   RFC3315:
   RFC6221:
   RFC7341:
   RFC7969:
   RFC8415:

informative:
   RFC0951:
   RFC1542:
   RFC2131:
   RFC2132:

--- abstract

This document describes a mechanism for networks
with legacy IPv4-only clients to use services provided by
DHCPv6 using DHCPv4-over-DHCPv6 (DHCP 4o6) in a Relay Agent.
RFC7341 specifies use of DHCPv4-over-DHCPv6 in the client only.
This document specifies
a RFC7341-based approach that allows DHCP 4o6 to be deployed as a
Relay Agent (4o6RA) that implements the 4o6 DHCP encapsulation
and decapsulation in contexts where it is not possible at the client.

--- middle

# Introduction {#introduction}

{{RFC7341}} describes a transport mechanism for carrying DHCPv4 {{RFC2131}}
messages using DHCPv6 {{RFC8415}} for the dynamic provisioning
of IPv4 addresses and other DHCPv4 specific configuration parameters
across IPv6-only networks. The deployment of {{RFC7341}} requires support in
DHCP clients and at the DHCPv6 server.

However, if a client is embedded in a host that only supports IPv4 and cannot
easily be replaced or updated due to a number of technical or business reasons,
this approach does not work.

Similarly, the specifications for DHCPv6 Relay Agents such as LDRA {{RFC6221}}
or L3RA {{RFC8415}} do not foresee the possibility to handle legacy DHCPv4,
other than implementing DHCP 4o6 in the client.

This document specifies an {{RFC7341}}-based solution that can be
implemented in intermediate nodes such as switches or routers,
without putting any requirements on clients. No new protocols or extensions are needed,
instead this document specifies an amendment to [RFC7341] that allows
a Relay Agent to perform the DHCP 4o6 encapsulation and decapsultion instead of
the client.

## Applicability Scope {#applicability}

The mechanisms described in this document apply to the configuration phase
of the host, specifically when a DHCP server for IPv4 {{RFC2131}} is not
reacheable directly from the host, the host needs to receive an IPv4 address
and there's a DHCPv6 server than can provide IPv4 addresses by means of
the mechanisms described in {{RFC7341}} and the host doesn't implement
a DHCP client conform to {{RFC7341}} itself.

# Conventions and Definitions

The following terms and acronyms are used in this document:

* DHCP:
   In this document when not otherwise specified it referres to DHCPv4 and/or DHCPv6

* DHCPv4:
   DHCP as defined in {{RFC2131}}

* DHCPv4 over DHCPv6 (or 4o6):
   The architecture, the procedures and the protocols described in the
   DHCPv4-over-DHCPv6 document {{RFC7341}}.

* DHCP Relay Agent:
   This is a concept in all of the protocols, BOOTP {{RFC0951}} {{RFC1542}}, DHCPv4
   {{RFC2131}} {{RFC2132}}, and DHCPv6 {{RFC8415}}, although the details differ
   between the protocols.

* Lightweight DHCPv6 Relay Agent (or LDRA):
   This is an extension of the original DHCPv6 Relay Agent,
   to support also Layer 2 devices performing a Relay Agent function {{RFC6221}}.

* DHCPv4 over DHCPv6 Relay Agent (or 4o6RA):
   Refers to a Relay Agent that implements the 4o6
   specified in this document.

{::boilerplate bcp14-tagged}

# DHCPv4 over DHCPv6 Relay Agent (4o6RA)

This document assumes a network, where IPv4-only hosts are connected
to a network that supports IPv6 and limited IPv4 services.

To address such a network setup, this document extends
DHCPv6 Relay Agents with  DHCPv4-over-DHCPv6, as
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
{: #fig_4o6RA title="Architecture Example with Legacy DHCP Client" artwork-align="center"}

This document specifies the encapsulation
and decapsulation described in {{RFC7341}} to be performed in the Relay Agent
whereas the DHCPv4 client does not require any change.
In this case it is up to the Relay Agent to provide the full DHCP
4o6 support whereas a legacy client is not aware of being served
via a DHCP 4o6 service.
All prerequisites and configuration that apply to the DHCP client in
{{Section 5 of RFC7341}} shall be applied to 4o6RA instead.

{{RFC7341}} specifies that before applying for an IPv4 address via a
DHCPv4-query message, the client must identify a suitable network interface
for the address. In the case described in this document where the
client functionality described in {{RFC7341}} is replaced by 4o6RA,
it's 4o6RA that shall also identify a suitable interface, that can be
a network interface or another Relay Agent.

To maintain interoperability with existing DHCPv6 relays and servers,
the message format is unchanged from {{RFC8415}}. The 4o6RA implements
the same message types as a normal DHCPv6 Relay Agent {{Section 6 of RFC7341}}.

In this specification, the 4o6RA creates the DHCPV4-QUERY Message
and encapsulates the DHCP request message received from the legacy DHCPv4 client.

When DHCPV4-RESPONSE Message is received by the 4o6 Relay Agent,
it looks for the DHCPv4 Message option within this message.
If this option is not found, the DHCPv4-response message MUST be discarded.
If the DHCPv4 Message option is present, the 4o6RA MUST extract the DHCPv4
message and forward the encapsulated DHCPv4-response to the requesting DHCPv4 client.

Layer 2 Relay Agents receiving DHCPV4-QUERY or DHCPV4-RESPONSE messages
are expected to handle them as specified in {{Section 6 of RFC6221}}.

DHCPv6 servers must be compliant with 4o6 according to {{RFC7341}}.
No additional requirements on DHCPv6 servers are set by this specification.

## Intermediate relays

Intermediate relays shall behave according to section 10 of {{RFC7341}}.

## 4o6RA and Topology Discovery {#topology_considerations}

In some networks the configuration of a host may depend on the
topology.  However, when the new host attaches to a
network, it may be unaware of the topology and respectively how it
has to be configured.

DHCPv4 {{RFC2131}} and DHCPv6 {{RFC3315}} specifications
describe how addresses can be allocated to clients based on network
topology information provided by a DHCP relay, typically.

Address/prefix allocation decisions are integral to the allocation of
addresses and prefixes in DHCP. The argument is described in details
in {{RFC7969}}, here we want to guarantee that 4o6RA does not
break any legacy capability when related to the use of topology.

The topology discovery as described in {{RFC7969}} differs between
IPv4 and IPv6 as in IPv4 it's only the first Relay Agent that can
set the giaddr field (section 3.1 of {{RFC7969}}) thus in a generic
network involving more than one Relay Agent only part of the topology
is transported via DHCPv4.

When in IPv6 context and using DHCPv6, all Relay Agents can fill
link-address and Interface-ID options, that allows transporting
to the DHCPv6 server the information about the complete path
between the DHCPv6 client and the DHCPv6 server.
When in a L2 network, Lightway DHCPv6 Relay Agents {{RFC6221}}
can be used. Then, topology information for the given IP address
can be obtained from the DHCPv6 server and used, for configuration
or other purposes.

The adoption of {{RFC7341}} at the client permits to use the DHCPv6
mechanisms for topology discovery even in case of DHCPv4, because
of the DHCPv6 encapsulation and because the DHCPv6 Relay Agent knows
the interface where the encapsulated DHCP request is received.

Moving 4o6 in the intermediate node rather than at the client breaks
the topology propagation as 4o6RA-only does not provide any interface
information in the encapsulated message.

~~~aasvg

           .-----------.     .-------------------------.
          | L2 Network  |   |        IPv6 Network       |
 +--------+-+         +-+---+---+    +--------+       +-+--------+
 |  DHCPv4  |         |  4o6    |    | DHCPv6 |       | DHCP 4o6 |
 |  Client  +---------+  Relay  +----+ Relay  +-------+  Server  |
 |          |         |  Agent  |    | Agent  |       |          |
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
 |  Client  +---------+  Relay + RFC6221 +------+  Server  |
 |          |         |  Agent |         |      |          |
 +--------+-+         +-+---+--+---------+      +----------+
          |             |   |                             |
           '-----------'     '---------------------------'

~~~
{: #fig_4o6LDRA title="Topology path preserved with LDRA" artwork-align="center"}

The assumed architecture is shown in {{fig_4o6LDRA}} where the whole
Relay Agent is built up with cooperating 4o6RA and LDRA, and an internal interface to
propagate topology information from 4o6RA to LDRA.

In a simple case, where the same node hosts the 4o6RA and the DHCP4o6 server,
it might be enough to only use 4o6RA, as shown in {{fig_4o6RAserver}}.

~~~aasvg

           .-----------.
          | L2 Network  |
 +--------+-+         +-+------+----------+
 |   DHCP   |         |  4o6   | DHCP 4o6 |
 |  Client  +---------+  Relay +  Server  |
 |  on CPE  |         |  Agent |          |
 +--------+-+         +-+------+----------+
          |             |
           '-----------'

~~~
{: #fig_4o6RAserver title="Topology path preserved 4o6 Relay Agent in DHCP server" artwork-align="center"}

# Deployment Considerations

As clients are not aware of the presence of 4o6RA, the network deployment needs to ensure that
all DHCPv4 broadcast and unicast messages from clients are steered to a 4o6RA.
This can, e.g., be achieved by placing the 4o6RA in a central position that can observe all traffic
from the clients or use of address translation with the 4o6RA address for unicast
messages.

# Security Considerations {#seccons}

This documents specifies the applicability of 4o6 DHCP in a scenario where legacy IPv4 clients are
connected to 4o6 DHCP Relay Agent that performs the en- and decapsulation. This document
does not change anything else in the 4o6 DHCP specification and therefore the
security consideration of {{RFC7341}} still apply.

This mechanism differs from {{RFC7341}} as the DHCP client
sends and receives DHCPv4 messages, whereas in {{RFC7341}} it
only sends DHCPv6 messages. This makes it possible that
DHCPv4 messages could reach a DHCPv4 server without using the 4o6RA.
While this can cause erroneous state in both clients and servers
and potentially even lead to misconfigurations that impact reachability,
this is not seen as a security concern rather than a deployment error
and even though it may be used for attacks from within the network,
it's not new and it's not being introduced because of this specification.

More generally, legacy IPv4 clients are not aware of this mechanism, however, even
when DHCP 4o6 is used, the client does not have any control about the
information provided by the Relay agent. As such this change does not
raise any additional security concerns.


# IANA Considerations

This document has no IANA actions.


--- back

# Example Use Case: Topology Discovery for IPv4-only Radio Unit in the RAN Switched Fronthaul {#usecase}

The Radio Fronthaul Network (FH) is built up with Radio Units (RU) and
Baseband Units (BB), each being an IP host.
Each RU is unique as it's tied to a set of antennas, each antennas
is serving a specific Cell and Sector.
RU is configured by BB depending to wich Cell and Sectors it serves,
that dependency is only specified by the cabling between RU and antennas.

In the legacy, BB is direcly cabled to a set of RUs so that the
BB can recognize the relationship between RU and Cell/Sectors
by relating the cabling towards the RU and the information
about cabling of RU and antennas.

The introduction of a switched network between RUs and BBs has
added a level of complexity that requires the BBs to have a deeper
knowledge of the topology, involving all the cabling between
the RUs and the switched network in order to properly configure the RUs.

The switched network can be generic and the level of complexity can
be related to the examples described in section 3 of {{RFC7969}}.
An example of FH is depicted in {{l2_switched_fh}}.

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

Among the various alternatives, DHCP topology knowledge can be used
for solving the RU configuration problem when FH is IPv6. Such solution
would use the topology discovery mechanisms described in section 3.2
of {{RFC7969}}. The same mechanisms applies when RU are IPv4 and
implement 4o6 according to {{RFC7341}}.

In order to extend the solution described above also to the case where
RU is IPv4 but it cannot support {{RFC7341}}, it's possible to adopt
the mechanisms described in this document by introducing 4o6RA at
the switches.

# Acknowledgments
{:numbered="false"}

The authors would also like to acknowledge interesting discussions in
this problem space with Sarah Gannon, Ines Ramadza and Siddharth Sharma.
