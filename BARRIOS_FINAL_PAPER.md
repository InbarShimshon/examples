# Network Architecture Design for Barrios Manufacturing & Solutions, Inc.

**Imbar Shimshon**  
Department of Data Science, Monroe University, King Graduate School  
26WN-CS640-153W — Computer Networks  
Professor David Gianna  
March 28th, 2026

---

## Introduction

As enterprise organizations expand globally, computer networks become critical infrastructure that enables communication, coordination, and business operations. Modern computer networks consist of billions of connected devices running network applications at the network edge, enabling communication and data exchange across organizational systems (Kurose & Ross, 2021). The Internet itself is a "network of networks" — a layered, packet-switched system in which packet switches forward chunks of data across communication links, and hosts run network applications at the edge (Kurose & Ross, 2021, Ch. 1). This layered architecture, known as the Internet protocol stack, provides the intellectual foundation for the design presented in this paper.

This paper proposes a network architecture design for the fictional global industrial automation company called ***Barrios Manufacturing & Solutions, Inc.*** The design addresses key challenges including inter-site connectivity, network resilience, secure access for both on-premises and remote users, and centralized infrastructure management. The proposed architecture follows a structured, top-down approach, and each section examines a core component of the design, forming a cohesive blueprint for enterprise network implementation. Intra-domain routing is provided by OSPF, inter-domain routing by BGP, WAN connectivity by a hybrid MPLS and SD-WAN architecture, and security is enforced at every protocol layer using IPsec, TLS, firewalls, and 802.1X network access control — all concepts grounded in Kurose & Ross (2021).

---

## Company Overview & Site Inventory

Barrios Manufacturing & Solutions, Inc. is a fictitious mid-sized industrial automation company specializing in smart manufacturing systems, IoT-enabled sensors, and factory control platforms. The organization operates across North America and Europe, supporting both enterprise IT environments and operational technology (OT) systems within manufacturing facilities. The company maintains a distributed global footprint consisting of headquarters, regional offices, satellite locations, manufacturing facilities, and third-party-hosted data centers across multiple geographic regions, as summarized in Figure 1 below.

The organization includes a U.S. headquarters supporting approximately **5,000 users**, a European main office with **2,500 users**, and **eight satellite offices** across the U.S. and Europe, each supporting **50–250 users**. The company operates **seven manufacturing facilities** (two in the U.S. and five in Europe) that integrate both IT and OT environments, as well as multiple data centers supporting primary and disaster recovery operations. A remote workforce of approximately **3,000 users** requires secure and reliable access to corporate resources.

**Figure 1 — Site Inventory**

| Site Type | Region | Count | Users |
|-----------|--------|-------|-------|
| Headquarters | US — Austin, TX | 1 | 5,000 |
| Main Regional Office | Europe — Frankfurt | 1 | 2,500 |
| Satellite Offices | US | 5 | 50–250 each |
| Satellite Offices | Europe | 3 | 50–250 each |
| Manufacturing Facilities | US | 2 | IT + OT |
| Manufacturing Facilities | Europe | 5 | IT + OT |
| Third-Party Data Centers | US — Dallas, Chicago | 2 | Colocation |
| HQ Data Center | US — Austin | 1 | On-premises |
| EU Data Center | Europe — Frankfurt | 1 | On-premises |
| Remote Users (VPN) | Worldwide | — | ~3,000 |

---

## Network Architecture Overview

The Barrios enterprise network follows a hierarchical design model consisting of three layers: core, distribution, and access. The core layer provides high-speed backbone connectivity, the distribution layer aggregates traffic and performs inter-VLAN routing, and the access layer connects end-user devices such as workstations, IP phones, and IoT systems (Kurose & Ross, 2021, Ch. 1). This hierarchy appears in Figure 2 below.

**Figure 2 — Three-Tier Campus Hierarchy**

```
 ┌──────────────────────────────────────────────────────────┐
 │                     CORE LAYER                           │
 │          High-speed backbone — redundant fiber           │
 │     [Core Switch A] ════════════ [Core Switch B]         │
 └─────────────────────────┬────────────────────────────────┘
                           │
 ┌─────────────────────────▼────────────────────────────────┐
 │                  DISTRIBUTION LAYER                      │
 │     Inter-VLAN routing  •  OSPF  •  Policy enforcement   │
 │   [Dist-1]          [Dist-2]          [Dist-3]           │
 └──────┬──────────────────┬─────────────────────┬──────────┘
        │                  │                     │
 ┌──────▼──────────────────▼─────────────────────▼──────────┐
 │                    ACCESS LAYER                          │
 │           End-user device connectivity (PoE)             │
 │  [Sw] [Sw] [Sw] [Sw] [Sw] [Sw] [Sw] [Sw] [Sw] [Sw]     │
 │   │    │    │    │    │    │    │    │    │    │          │
 │  PCs VoIP  APs  PCs  IoT  PCs VoIP  APs  PCs  IoT       │
 └──────────────────────────────────────────────────────────┘
```

At the WAN level, the enterprise adopts a hybrid architecture. Communication is based on IPv4 addressing. Internally, the organization uses private IPv4 address ranges — the 10/8, 172.16/12, and 192.168/16 prefixes — which allow devices to communicate within the enterprise without requiring globally unique addresses (Kurose & Ross, 2021, Ch. 4). To enable communication with the Internet, Network Address Translation (NAT) is implemented at the network edge; NAT translates internal private addresses to a public IP address, replacing the source IP and port in every outgoing datagram and maintaining a translation table to route return traffic correctly (Kurose & Ross, 2021, Ch. 4).

For inter-site connectivity the network uses both MPLS and SD-WAN. MPLS forwards packets through a carrier's private backbone based on labels rather than IP destination addresses, providing predictable latency and guaranteed bandwidth. SD-WAN applies Software-Defined Networking principles to the WAN: a centralized controller computes forwarding policies and pushes them to edge appliances, separating the control plane from the data plane (Kurose & Ross, 2021, Ch. 4).

The enterprise network follows a hub-and-spoke topology, with the U.S. headquarters as the primary hub and the European office as the regional hub. **OSPF (Open Shortest Path First)**, a link-state intra-domain routing protocol, is used within the enterprise backbone; every router floods link-state advertisements to build a complete topology map and runs Dijkstra's shortest-path algorithm locally (Kurose & Ross, 2021, Ch. 5). **BGP (Border Gateway Protocol)** is used at each Internet edge to manage multi-homed ISP connectivity; it is the inter-domain protocol that interconnects the Internet's autonomous systems (Kurose & Ross, 2021, Ch. 5).

**Figure 3 — Global WAN Topology**

```
        ╔══════════════════════════════════════════════╗
        ║             CARRIER MPLS BACKBONE            ║
        ╚═══════════════════╤══════════════════════════╝
                            │  Transatlantic MPLS Circuit
             ┌──────────────┴──────────────┐
             │                             │
  ┌──────────▼──────────┐       ┌──────────▼──────────┐
  │    US HQ — Austin   │       │  EU HQ — Frankfurt  │
  │  5,000 users        │       │  2,500 users        │
  │  BGP — Dual ISP     │       │  BGP — Dual ISP     │
  └──────────┬──────────┘       └──────────┬──────────┘
             │  SD-WAN IPsec tunnels                  │
     ┌───────┴──────┬────────┐         ┌──────────────┼────┐
     │              │        │         │              │    │
 [SAT-US×5]  [MFG-US×2]  [US-DC1]  [SAT-EU×3]  [MFG-EU×5] [EU-DC]
  SD-WAN      SD-WAN      MPLS      SD-WAN       SD-WAN  On-prem
                       [US-DC2]
                        MPLS

  ⊕ Remote users (3,000) → SSL/TLS VPN or IPsec VPN → US-VPN or EU-VPN
```

---

## Headquarters Network Design (US HQ)

The U.S. headquarters in Austin, Texas supports approximately 5,000 users and serves as the central hub of the enterprise network. The campus network follows the three-tier hierarchical design described above.

The core layer provides high-speed backbone connectivity using redundant Layer 3 switches connected via high-capacity fiber. The distribution layer aggregates traffic from access switches and performs inter-VLAN routing via Layer 3 switching running OSPF. The access layer connects end-user devices — workstations, IP phones, and IoT devices — via PoE switches.

Traffic is segmented using VLANs based on department and function. Standard VLANs include VLAN 10 for user workstations, VLAN 20 for VoIP, VLAN 30 for wireless clients, VLAN 40 for management, VLAN 50 for guest Internet (isolated), and VLAN 60 for the server DMZ. Inter-VLAN routing is performed at the distribution layer.

At the network edge, the headquarters connects to two ISPs using BGP for redundancy. A pair of next-generation firewalls provides perimeter security, intrusion prevention, and traffic inspection. The headquarters also hosts the primary VPN concentrators and the HQ data center.

DNS is deployed locally — local DNS servers resolve hostnames for 5,000 users without relying on external lookups for every query, following the hierarchical DNS resolution model in which resolvers cache responses and only contact authoritative servers when a cached answer is unavailable (Kurose & Ross, 2021, Ch. 2). DHCP servers assign IP addresses, subnet masks, default gateways, and DNS server options to all LAN hosts automatically (Kurose & Ross, 2021, Ch. 2). Quality of Service policies at the distribution and WAN edge prioritize latency-sensitive UDP traffic — particularly VoIP — over bulk TCP transfers, consistent with the transport-layer service requirements discussed in Kurose & Ross (2021, Ch. 3).

**Figure 4 — US HQ Network Edge**

```
  [ISP-A — Tier 1]          [ISP-B — Regional]
         │  eBGP                     │  eBGP
  ┌──────▼───────────────────────────▼──────┐
  │        EDGE ROUTERS (HA pair)           │
  │   Multi-homed BGP  •  NAT at edge       │
  └─────────────────────┬───────────────────┘
                        │
  ┌─────────────────────▼───────────────────┐
  │     NEXT-GEN FIREWALLS (Active/Standby) │
  │  Stateful inspection  •  IPS  •  VPN    │
  └──────────┬──────────────────────────────┘
             │
  ┌──────────▼────────────────────────────┐
  │           DMZ                         │
  │  Web Servers  •  Email Gateway        │
  │  Partner APIs  •  VPN Concentrators   │
  └──────────┬────────────────────────────┘
             │
  ┌──────────▼────────────────────────────┐
  │   INTERNAL NGFW  (App-layer DPI)      │
  └──────────┬────────────────────────────┘
             │
  ┌──────────▼────────────────────────────┐
  │     CORE SWITCHES  →  Campus + WAN    │
  └───────────────────────────────────────┘
```

---

## European Main Office Design

The European main office in Frankfurt, Germany serves as the regional hub for all European operations, supporting **2,500 users**. Frankfurt was selected because it is one of the world's largest Internet exchange points, providing access to multiple ISPs with low-latency connectivity to both the U.S. headquarters and the European satellite offices.

The campus mirrors the three-tier hierarchical design of the U.S. headquarters. The European office is assigned a dedicated private subnet (10.20.0.0/16), separate from the U.S. HQ address space (10.10.0.0/16), preventing address overlap across the enterprise. NAT is performed at the European edge router for outbound Internet traffic (Kurose & Ross, 2021, Ch. 4). Local DNS and DHCP servers serve 2,500 users without relying on transatlantic WAN links for every query or address assignment (Kurose & Ross, 2021, Ch. 2).

The European office connects to the U.S. headquarters via a dedicated MPLS private WAN circuit traversing a transatlantic carrier network. A secondary SD-WAN over broadband provides failover capacity. The SD-WAN controller monitors link quality continuously — measuring latency, jitter, and packet loss — and reroutes traffic to the best available path when degradation is detected. Dual ISP connections at the Frankfurt edge are managed by eBGP (Kurose & Ross, 2021, Ch. 5). All critical components — core switches, edge routers, firewalls — are deployed in redundant pairs with automatic failover.

---

## Data Center Design

Barrios operates **four data centers**: HQ-DC (Austin, on-premises), US-DC1 (Dallas, colocation), US-DC2 (Chicago, colocation), and EU-DC (Frankfurt, on-premises). This distributed architecture provides geographic redundancy and low-latency access to compute and storage resources across all sites.

Each data center uses a **leaf-spine topology** internally. Leaf switches connect directly to servers and uplink to every spine switch; spine switches form the high-speed backbone interconnecting all leaf switches. Any server reaches any other server in exactly two hops — leaf to spine to leaf — providing consistent, low-latency forwarding consistent with the network-layer forwarding concepts in Kurose & Ross (2021, Ch. 4).

**Figure 5 — Data Center Leaf-Spine Architecture**

```
        ┌─────────────┐          ┌─────────────┐
        │   Spine 1   │◄────────►│   Spine 2   │
        └──┬──────┬───┘          └───┬──────┬──┘
           │      │                  │      │
      ┌────▼─┐  ┌─▼────┐       ┌────▼─┐  ┌─▼────┐
      │Leaf A│  │Leaf B│       │Leaf C│  │Leaf D│
      └──┬───┘  └──┬───┘       └──┬───┘  └──┬───┘
         │         │              │          │
    [App Srvs] [DB Srvs]     [Backup]   [Storage]

  Any server → any other server = exactly 2 hops
  (Leaf → Spine → Leaf)
```

Each data center is assigned its own private address range. OSPF distributes routing information among leaf and spine switches within each data center (Kurose & Ross, 2021, Ch. 5). US-DC1 and US-DC2 connect to the enterprise WAN via dedicated MPLS circuits from the colocation facility's meet-me room — data center traffic never crosses the public Internet. Each data center edge implements a **dual-firewall DMZ**: an external firewall faces the Internet, and an internal firewall separates the DMZ from the internal server farm. As described in Kurose & Ross (2021, Ch. 8), this ensures that an attacker who compromises a DMZ server still faces the internal firewall before reaching core systems.

**Figure 6 — Dual-Firewall DMZ Architecture**

```
  ══════════════════ INTERNET ══════════════════
                         │
                ┌────────▼────────┐
                │  External NGFW  │
                │  Stateful + IPS │
                └────────┬────────┘
                         │
  ╔═════════════════════════════════════╗
  ║               D M Z                 ║
  ║  Web/App Servers  •  Email Gateway  ║
  ║  Partner APIs  •  VPN Concentrators ║
  ╚═════════════════════════════════════╝
                         │
                ┌────────▼────────┐
                │  Internal NGFW  │
                │  App-layer DPI  │
                └────────┬────────┘
                         │
  ╔═════════════════════════════════════╗
  ║         INTERNAL SERVER FARM        ║
  ║  Databases  •  ERP  •  LDAP         ║
  ╚═════════════════════════════════════╝
```

DNS zone data for the internal domain is mastered in HQ-DC and replicated to EU-DC and US-DC1, consistent with the DNS hierarchy described in Kurose & Ross (2021, Ch. 2). US-DC1 and US-DC2 provide active-active failover for U.S. workloads; EU-DC provides active-standby for European workloads. All replication traffic between data centers is encrypted with IPsec (Kurose & Ross, 2021, Ch. 8).

---

## Satellite Office Design

Barrios operates **eight satellite offices** — five in the U.S. and three in Europe — each supporting **50–250 users**. These offices use a **collapsed two-tier architecture**: core and distribution functions are combined into a single redundant Layer 3 switch pair, with a separate access layer below it. A full three-tier design would add unnecessary cost and complexity at this scale.

**Figure 7 — Satellite Office Collapsed Architecture**

```
  ┌────────────────────────────────────────────────────────┐
  │              SATELLITE OFFICE (50–250 users)           │
  │                                                        │
  │  [Broadband ISP 1]       [Broadband ISP 2 / 4G backup] │
  │         │                        │                     │
  │  ┌──────▼────────────────────────▼──────┐             │
  │  │   SD-WAN Edge Appliance              │             │
  │  │   IPsec VPN to Regional Hub          │             │
  │  │   Local Internet breakout            │             │
  │  │   Integrated firewall                │             │
  │  └──────────────────────┬───────────────┘             │
  │                         │                             │
  │  ┌──────────────────────▼───────────────┐             │
  │  │   Collapsed Core (Layer 3 Switch)    │             │
  │  │   OSPF  •  Inter-VLAN routing  •  ACLs│            │
  │  └──────┬──────────────┬──────────┬─────┘             │
  │         │              │          │                   │
  │  [Access Sw 1]  [Access Sw 2]  [Access Sw 3]          │
  │   VLAN 10/20    VLAN 10/30    VLAN 40/50               │
  │   Workstations  Wireless APs  Mgmt / Guest             │
  └────────────────────────────────────────────────────────┘
```

Each satellite office is assigned a dedicated /24 subnet from the enterprise private address space, providing up to 254 usable host addresses — sufficient for the maximum 250-user site. DHCP services are provided by the regional hub's central DHCP server via a **DHCP relay agent** on the local Layer 3 switch, which forwards DHCP Discover messages across the WAN to the central server (Kurose & Ross, 2021, Ch. 2). DNS queries are proxied to the regional hub's DNS servers.

Satellite offices connect to their regional hub using SD-WAN over broadband as the primary model, with 4G/5G as a backup link. All WAN traffic is encrypted using IPsec (Kurose & Ross, 2021, Ch. 8). SD-WAN policy routes VoIP and ERP traffic through the encrypted tunnel to the hub, while Internet-bound SaaS traffic breaks out locally at the SD-WAN appliance — reducing WAN bandwidth consumption and improving latency for cloud applications. Guest VLAN traffic always exits directly to the Internet and never enters the enterprise WAN.

---

## Manufacturing Site Design

Barrios operates **seven manufacturing facilities** — two in the U.S. and five in Europe. Manufacturing sites present unique networking challenges because of the co-existence of **Information Technology (IT)** and **Operational Technology (OT)** systems. OT systems include PLCs (Programmable Logic Controllers), SCADA (Supervisory Control and Data Acquisition) servers, and IoT-enabled sensors. As Kurose & Ross (2021, Ch. 1) note, the Internet now increasingly connects not just computers but sensors, actuators, and industrial devices — creating both operational opportunity and significant security risk. These systems have strict real-time requirements — deterministic, sub-millisecond communication — that cannot tolerate the congestion or jitter acceptable in standard IT environments (Kurose & Ross, 2021, Ch. 4).

The manufacturing network is divided into three strictly separated zones. The **IT Zone** follows the same collapsed two-tier architecture as satellite offices and connects to the enterprise WAN. The **OT Zone** is an isolated network segment dedicated to all industrial control equipment, using physically separate switches and cabling. The **IT/OT DMZ** — implemented via a next-generation firewall with deep packet inspection — sits between the two zones and hosts data historians, MES integration interfaces, and remote-access jump servers for vendor engineers.

**Figure 8 — Manufacturing IT/OT Segmentation**

```
  Enterprise WAN
       │
  ┌────▼─────────────────────────────────────────┐
  │                   IT ZONE                    │
  │  Standard LAN  •  VLAN 10/20/30  •  OSPF     │
  │  Admin workstations  •  IP phones            │
  └────┬─────────────────────────────────────────┘
       │  (only historian protocols permitted)
  ┌────▼─────────────────────────────────────────┐
  │               IT/OT DMZ                      │
  │  NGFW with Deep Packet Inspection             │
  │  ┌─────────────┐  ┌──────────┐  ┌─────────┐ │
  │  │Data Historian│  │   MES   │  │  Jump   │ │
  │  │(OPC-UA data) │  │Interface│  │ Server  │ │
  │  └─────────────┘  └──────────┘  └─────────┘ │
  └────┬─────────────────────────────────────────┘
       │  (read-only OPC-UA  •  no direct IT access)
  ┌────▼─────────────────────────────────────────┐
  │                  OT ZONE                     │
  │  Industrial Ethernet  •  Static IP addressing│
  │  PLCs / RTUs   •   SCADA Servers             │
  │  IoT Sensors   •   Actuators                 │
  │  (never reachable from IT network directly)  │
  └──────────────────────────────────────────────┘
```

This three-zone architecture directly implements the defense-in-depth security model from Kurose & Ross (2021, Ch. 8): multiple overlapping boundaries ensure that a compromise in one zone does not cascade into adjacent zones. OT traffic never traverses the enterprise WAN — all industrial control communication stays local. The OT network uses IEEE 802.3-based Industrial Ethernet on ruggedized switches, with QoS priority marking to ensure real-time control traffic is not delayed by lower-priority data flows. IoT sensor data is aggregated by the data historian in the IT/OT DMZ and forwarded to the analytics platform in the data center. The 5 GHz 802.11 band is preferred for any wireless access on the factory floor to avoid interference from industrial equipment such as variable-frequency drives (Kurose & Ross, 2021, Ch. 7).

---

## Remote Access (VPN)

Barrios employs approximately **3,000 remote employees and sales representatives** worldwide, all requiring secure access to corporate resources. The remote access solution is built on **IPsec VPN** and **TLS-based VPN** technologies, both described in Kurose & Ross (2021, Ch. 8). These protocols provide the three core security properties required: **confidentiality** (encryption prevents eavesdropping), **authentication** (verifying the identity of both parties), and **message integrity** (ensuring data is not altered in transit).

VPN concentrators are deployed in active-active pairs at two locations: US-VPN-1 and US-VPN-2 at the U.S. headquarters, and EU-VPN-1 and EU-VPN-2 at the European main office. Global DNS-based load balancing directs each user to the nearest concentrator based on the geographic origin of their DNS query (Kurose & Ross, 2021, Ch. 2).

The primary remote access method is an **SSL/TLS VPN**: a lightweight agent on the user's device establishes an encrypted TLS tunnel over HTTPS (TCP port 443). TLS provides confidentiality via symmetric encryption, integrity via cryptographic hashing, and authentication via the server's digital certificate — all techniques described in Kurose & Ross (2021, Ch. 8). TLS 1.3, standardized as RFC 8446 in 2018, is required for all connections; it uses AES-GCM for encryption and HMAC-SHA for integrity, and reduces the handshake to a single round-trip (Kurose & Ross, 2021, Ch. 8).

For company-managed laptops, **IPsec in tunnel mode** is used. Tunnel mode encapsulates the entire original IP packet inside a new encrypted outer packet, so an eavesdropper sees only an encrypted payload between two public IP addresses (Kurose & Ross, 2021, Ch. 8). The **ESP (Encapsulating Security Payload)** protocol — RFC 4303 — provides both encryption and authentication for each datagram. The **AH (Authentication Header)** protocol — RFC 4302 — provides integrity and authentication without encryption. IKEv2 handles session establishment and re-keying.

**Figure 9 — IPsec Tunnel Mode Encapsulation**

```
  ORIGINAL PACKET (before IPsec):
  ┌──────────────┬───────────┬───────────────────────────┐
  │ Original IP  │  TCP/UDP  │      Payload (data)       │
  │   Header     │   Header  │                           │
  └──────────────┴───────────┴───────────────────────────┘

  AFTER IPsec TUNNEL MODE — ESP (RFC 4303):
  ┌─────────────┬───────┬══════════════════════════════════╗
  │  New IP Hdr │  ESP  │  ENCRYPTED: [Original IP Hdr]   ║
  │ (public IPs)│  Hdr  │  [TCP/UDP Hdr][Data]            ║
  │             │       │  + ESP Auth Trailer (HMAC)      ║
  └─────────────┴───────┴══════════════════════════════════╝

  Eavesdropper sees only: encrypted blob between two public IPs
  Neither original addresses nor payload are readable
```

All VPN sessions require **multi-factor authentication (MFA)**: corporate credentials verified against Active Directory, plus a time-based one-time password from an authenticator app. This implements the authentication principle in Kurose & Ross (2021, Ch. 8) — authentication based on something the user knows and something the user possesses — protecting access even if a password is stolen.

**Split tunneling** is the default policy: enterprise traffic routes through the encrypted tunnel to the hub, while Internet-bound SaaS traffic (email, collaboration tools) exits locally from the user's broadband connection. Full tunnel mode is enforced for users accessing manufacturing or financial systems, ensuring all traffic passes through enterprise security controls.

**Figure 10 — Remote Access VPN Architecture**

```
  Remote Users (3,000 worldwide)
  ┌────────────────────────────────────────┐
  │  Home WiFi  •  Hotel  •  4G/5G Mobile  │
  └──────────────────┬─────────────────────┘
                     │  TLS 1.3 or IPsec Tunnel Mode
           ┌─────────┴─────────┐
           │                   │
   ┌───────▼──────┐    ┌───────▼──────┐
   │  US-VPN-1/2  │    │  EU-VPN-1/2  │
   │ Active-Active│    │ Active-Active│
   │  Austin, TX  │    │  Frankfurt   │
   └───────┬──────┘    └───────┬──────┘
           │  MFA required     │
           │  (password + TOTP)│
           └─────────┬─────────┘
                     │
  ┌──────────────────▼──────────────────────┐
  │          ENTERPRISE NETWORK             │
  │  US HQ ←→ EU HQ ←→ Data Centers        │
  └─────────────────────────────────────────┘

  SPLIT TUNNEL:
  ├── *.barrios.internal  →  VPN tunnel → internal resources
  └── SaaS / Internet     →  local breakout (bypasses VPN)
```

---

## Connectivity Between Sites

Inter-site connectivity is the backbone of the Barrios global enterprise. All sites are connected through a **hybrid WAN** combining three technologies.

**MPLS** serves the most critical links: HQ to data centers, HQ to EU office. MPLS assigns labels to packets at ingress and forwards them along predetermined Label Switched Paths through the carrier's private backbone — packets never traverse the public Internet. This provides carrier-grade latency and bandwidth guarantees. The forwarding mechanism directly relates to the network-layer data plane concepts in Kurose & Ross (2021, Ch. 4), where MPLS uses a label as the lookup key in the forwarding table rather than an IP destination address.

**SD-WAN over broadband** serves satellite offices and manufacturing sites. The SD-WAN control plane — a centralized controller — computes and distributes forwarding policies to edge appliances, a direct application of the Software-Defined Networking architecture described in Kurose & Ross (2021, Ch. 4, Ch. 5). SD-WAN provides: application-aware routing (VoIP gets the lowest-latency path, bulk backup traffic is scheduled off-peak), dynamic path selection (automatic rerouting when a link degrades below threshold), and zero-touch provisioning (new sites brought online without on-site engineers). All SD-WAN traffic is encrypted using IPsec tunnel mode (Kurose & Ross, 2021, Ch. 8).

**OSPF** is the intra-domain routing protocol across the enterprise backbone (Kurose & Ross, 2021, Ch. 5). The network is divided into OSPF areas: Area 0 (backbone, interconnecting HQ, data centers, and EU office via MPLS), Area 1 (U.S. satellite and manufacturing sites), and Area 2 (European satellite and manufacturing sites). Area partitioning reduces the volume of link-state advertisement flooding and improves scalability. If any WAN link fails, OSPF reconverges within seconds and automatically reroutes traffic.

**BGP** manages multi-homed Internet connectivity at the headquarters and EU office edges (Kurose & Ross, 2021, Ch. 5). eBGP sessions are maintained with each of two ISPs at each site. BGP attributes — AS path, local preference — are tuned to distribute traffic across both ISPs during normal operation and achieve automatic failover if an ISP link fails.

The transatlantic MPLS circuit between Austin and Frankfurt traverses undersea fiber infrastructure with a round-trip time of approximately 130–160 ms — a physical constraint set by the speed of light. TCP applications must accommodate this latency through window scaling and other optimization techniques (Kurose & Ross, 2021, Ch. 3).

**Figure 11 — BGP Dual-ISP at HQ Edge**

```
  [ISP-A — Tier 1]         [ISP-B — Regional]
        │  eBGP                    │  eBGP
  ┌─────▼──────────────────────────▼─────┐
  │       EDGE ROUTERS (HA pair)         │
  │  eBGP to both ISPs                   │
  │  AS Path + Local-Pref tuning         │
  │  Automatic failover on ISP loss      │
  └─────────────────┬────────────────────┘
                    │
  ┌─────────────────▼────────────────────┐
  │       NEXT-GEN FIREWALLS (HA)        │
  └─────────────────┬────────────────────┘
                    │
  ┌─────────────────▼────────────────────┐
  │          CORE SWITCHES               │
  │        US HQ Internal LAN            │
  └──────────────────────────────────────┘
```

---

## Network Management & Monitoring

Managing a globally distributed enterprise network requires comprehensive, centralized visibility. Without systematic monitoring, administrators have no awareness of performance degradation, configuration drift, or security incidents until users report problems — by which time business impact has already occurred.

**SNMP (Simple Network Management Protocol)** is the foundational network management protocol described in Kurose & Ross (2021, Ch. 5). SNMP uses a manager-agent architecture: agents on every managed device maintain a **Management Information Base (MIB)** containing device state variables — interface utilization, error counters, CPU and memory usage, routing table entries. The central SNMP manager polls agents with GET requests and receives asynchronous TRAP notifications when significant events occur (Kurose & Ross, 2021, Ch. 5). Barrios deploys **SNMPv3** exclusively. SNMPv1 and SNMPv2c use plaintext community strings for authentication, which is inadequate for a security-conscious enterprise. SNMPv3 adds HMAC-based message authentication and AES encryption, consistent with the security principles in Kurose & Ross (2021, Ch. 8).

**Figure 12 — SNMP Manager-Agent Model**

```
  ╔══════════════════════════════════════╗
  ║   NETWORK MANAGEMENT SYSTEM (NMS)   ║
  ║   SNMPv3 Manager — HQ Data Center   ║
  ║                                     ║
  ║  GET requests (poll every 5 min)    ║
  ║  TRAP receiver (async alerts)       ║
  ║  Topology map  •  Dashboards        ║
  ╚══════════════╤═══════════════════════╝
                 │  SNMPv3 (auth + encrypted)
       ┌─────────┼──────────────────┐
       │         │                  │
  ┌────▼───┐ ┌───▼────┐       ┌────▼───┐
  │ Agent  │ │ Agent  │  ...  │ Agent  │
  │Core Sw │ │WAN Rtr │       │Remote  │
  │        │ │        │       │Site AP │
  │  MIB:  │ │  MIB:  │       │  MIB:  │
  │ifOctets│ │ifUtil  │       │clients │
  │cpuUsage│ │bgpState│       │channel │
  └────────┘ └────────┘       └────────┘

  TRAP example: interface goes down → agent sends
  unsolicited TRAP to NMS → NMS pages on-call engineer
```

**NetFlow/IPFIX** supplements SNMP by providing traffic flow analysis. Routers export flow records — summarizing every network conversation by source and destination IP, protocol, port, and byte counts — to a flow collector. This identifies top bandwidth consumers, detects anomalous traffic patterns indicative of security incidents, and informs long-term capacity planning.

**Syslog** aggregates event logs from all network devices — configuration changes, login events, error conditions — to centralized syslog servers. Log retention meets compliance requirements for relevant regulations.

A centralized **NMS** deployed at HQ-DC provides topology discovery, fault monitoring, performance dashboards, configuration backup with version control, and threshold alerting. The **SD-WAN controller** provides its own management console for all SD-WAN sites, presenting real-time WAN link quality and application performance — applying the SDN centralized control plane principle of Kurose & Ross (2021, Ch. 4) to day-to-day operations.

A **SIEM (Security Information and Event Management)** platform aggregates logs from all firewalls, IDS sensors, VPN concentrators, and endpoint agents, correlating events across devices to detect attack patterns invisible in any single device's logs. This operational security function aligns with the role of firewalls and IDS described in Kurose & Ross (2021, Ch. 8). A failed VPN login followed immediately by a successful login from a different country, for example, generates a SIEM alert that neither system would catch independently.

All network device configuration is managed via SSH with public-key authentication — Telnet is disabled globally. Role-based access control ensures only authorized engineers can push changes. An **out-of-band (OOB) management network** connects to the console ports of all critical infrastructure at major sites, ensuring that if a misconfiguration disables the production network, administrators can still reach all devices to diagnose and recover without a site visit.

---

## Security Design

Network security at Barrios is a comprehensive design philosophy applied at every layer of the protocol stack, not a perimeter product. Kurose & Ross (2021, Ch. 8) define the core properties of network security: **confidentiality** (only intended parties can read data), **authentication** (verifying the identity of communicating parties), **message integrity** (ensuring data is not altered in transit), and **access and availability** (ensuring services remain accessible to authorized users). The Barrios architecture addresses all four at each protocol layer through a **defense-in-depth** model — multiple overlapping layers of control ensure that a failure in any single control does not result in a complete breach (Kurose & Ross, 2021, Ch. 8).

**Firewalls** are deployed at every network boundary. As described in Kurose & Ross (2021, Ch. 8), stateless packet filters examine individual packets by source/destination IP and port without memory of prior packets, while stateful packet filters track connection state and permit only packets belonging to established legitimate sessions. Barrios deploys next-generation firewalls (NGFWs) that extend stateful inspection with Layer 7 application awareness. All firewall rule sets follow the principle of least privilege: all traffic is denied by default, and only explicitly permitted flows are allowed, each documented with a business justification and review schedule.

**IDS/IPS (Intrusion Detection and Prevention Systems)** — described in Kurose & Ross (2021, Ch. 8) — monitor traffic for signatures of known attacks and anomalous behavior. IDS passively detects and alerts; IPS actively blocks inline. Barrios deploys network IPS inline at headquarters and European perimeters. Passive IDS sensors are used at the IT/OT boundary in manufacturing sites, where inline blocking could disrupt real-time control systems. All IDS/IPS alerts feed the SIEM for correlation.

**Figure 13 — Defense-in-Depth Security Architecture**

```
  🌐 Internet
       │
  ┌────▼─────────────────────────────────────────┐
  │  PERIMETER LAYER                             │
  │  External NGFW  •  Stateful + IPS            │
  └────┬─────────────────────────────────────────┘
       │
  ┌────▼─────────────────────────────────────────┐
  │  DMZ LAYER                                   │
  │  Public-facing servers (web, email, API)     │
  └────┬─────────────────────────────────────────┘
       │
  ┌────▼─────────────────────────────────────────┐
  │  INTERNAL LAYER                              │
  │  Internal NGFW  •  802.1X NAC  •  VLANs/ACLs│
  └────┬─────────────────────────────────────────┘
       │
  ┌────▼─────────────────────────────────────────┐
  │  ENDPOINT LAYER                              │
  │  Host IDS  •  Disk encryption  •  MDM        │
  └────┬─────────────────────────────────────────┘
       │
  ┌────▼─────────────────────────────────────────┐
  │  MANAGEMENT LAYER                            │
  │  SIEM correlation  •  SNMPv3  •  SSH only    │
  └──────────────────────────────────────────────┘

  Attacker must breach EVERY layer to reach core systems.
  A failure in any one layer is contained by the next.
```

**IPsec** protects all WAN traffic. All site-to-site SD-WAN tunnels and remote access VPN sessions use IPsec ESP (RFC 4303) in tunnel mode with AES-GCM encryption and HMAC-SHA integrity (Kurose & Ross, 2021, Ch. 8). IPsec provides datagram-level protection — every IP packet is individually encrypted and authenticated — distinct from TLS, which protects only the payload of a TCP connection.

**TLS 1.3** (RFC 8446) protects all internal and external web-based applications and APIs (Kurose & Ross, 2021, Ch. 8). AES-GCM provides confidentiality; HMAC-SHA provides integrity; digital certificates enable server authentication. Certificates are issued by the corporate PKI. HTTP Strict Transport Security (HSTS) is enforced on all web properties to prevent protocol downgrade attacks.

**802.1X port-based Network Access Control (NAC)** is deployed on all wired and wireless access ports. Before any device is granted network access, it must authenticate with the RADIUS server — using certificate-based EAP-TLS for managed devices, or credential-based EAP-MSCHAPv2 for personal devices. Unauthenticated devices are placed in a quarantine VLAN with Internet-only access, preventing them from reaching any internal resource.

**Figure 14 — 802.1X Authentication Flow**

```
  End Device          Access Switch          RADIUS Server
 (Supplicant)        (Authenticator)        (Auth Server)
      │                    │                      │
      │── EAPOL Start ────►│                      │
      │                    │── RADIUS Request ───►│
      │◄── EAP Identity ───│◄─ EAP Challenge ─────│
      │── EAP Response ───►│── RADIUS Request ───►│
      │   (cert or creds)  │   (with credentials) │
      │                    │◄─ Access-Accept ──────│
      │◄── EAP Success ────│                      │
      │   (port opened)    │                      │
      │                    │
      Port assigned to VLAN 10/20/30/etc.
      or quarantine VLAN if auth fails
```

**VLAN segmentation** limits lateral movement: an infected workstation in VLAN 10 cannot directly reach database servers in VLAN 60 without traversing a firewall and ACL-controlled Layer 3 hop.

**Manufacturing site security** adds OT-specific controls: the IT/OT DMZ firewall permits only historian protocols (OPC-UA) from OT to the DMZ — all other IT-to-OT traffic is blocked. Vendor remote access to OT systems requires a supervised, session-recorded jump server in the DMZ with time-limited credentials. USB ports are locked down on all OT workstations to prevent removable-media attacks.

**Security controls by layer:**

| Layer | Control | Grounding |
|-------|---------|-----------|
| Link | 802.1X NAC, VLAN segmentation | Kurose & Ross, Ch. 6, 8 |
| Network | IPsec ESP (RFC 4303), NGFW ACLs, IPS | Kurose & Ross, Ch. 8 |
| Transport | TLS 1.3 (RFC 8446) | Kurose & Ross, Ch. 8 |
| Application | MFA (password + TOTP) | Kurose & Ross, Ch. 8 |
| Management | SNMPv3 (auth + AES), SSH only, SIEM | Kurose & Ross, Ch. 5, 8 |

---

## Wireless Network Design

Wireless LAN connectivity is required across all Barrios facilities. Kurose & Ross (2021, Ch. 7) describe IEEE 802.11 as the dominant wireless LAN standard, covering the physical and link-layer behavior of wireless networks, the CSMA/CA medium access protocol, and the challenges of the wireless medium including the hidden node problem and multipath fading.

Barrios deploys **IEEE 802.11ax (Wi-Fi 6)** for all new wireless infrastructure deployments, with Wi-Fi 6E access points (adding the 6 GHz band) in high-density areas such as headquarters conference rooms and large open-plan offices where the less-congested 6 GHz spectrum reduces co-channel interference.

As described in Kurose & Ross (2021, Ch. 7), wireless networks use **CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance)** for medium access. Because wireless devices are half-duplex — they cannot simultaneously transmit and receive — collision detection as used in wired Ethernet is not possible. Instead, a device senses the channel before transmitting and waits a random backoff interval to reduce the probability of collisions. The **hidden node problem** — where two devices are each in range of the AP but not of each other — is addressed by the optional RTS/CTS (Request to Send / Clear to Send) mechanism.

**Figure 15 — 802.11 CSMA/CA Medium Access**

```
  Device A           Channel             Device B
     │                                      │
     │── Sense channel (idle?) ─────────────│
     │   [Channel idle > DIFS interval]     │
     │── Random backoff timer ──────────────│
     │── TRANSMIT FRAME ──────────────────► │
     │                        [B senses BUSY, defers]
     │◄── ACK (success) ──────────────────── │
                                             │
                              [Channel idle again]
                              [B counts down its backoff]
                              [B transmits]

  RTS/CTS (Hidden Node solution):
  A sends RTS → AP replies CTS → all nodes hear CTS
  → only A transmits → hidden node problem resolved
```

Barrios uses a **centralized wireless LAN controller (WLC)** architecture. Lightweight access points (LWAPs) forward all wireless frames to the WLC via CAPWAP tunnels; the WLC handles all authentication, roaming, radio resource management, and policy enforcement. This mirrors the SDN separation of control plane and data plane described in Kurose & Ross (2021, Ch. 4): the WLC is the centralized controller, and APs are simple forwarding devices.

Each facility broadcasts multiple SSIDs mapped to dedicated VLANs. The corporate SSID uses **WPA3-Enterprise** with 802.1X/RADIUS authentication. WPA3 addresses known vulnerabilities in WPA2 that were exploited by attacks such as KRACK (Kurose & Ross, 2021, Ch. 8). A BYOD SSID supports employee personal devices. A guest SSID places visitors in isolated VLAN 50 with Internet-only access through a captive portal, with client isolation preventing guest-to-guest communication.

**IEEE 802.11r (Fast BSS Transition)** is deployed on corporate and voice SSIDs to support seamless roaming. Without it, roaming between APs triggers a full 802.1X re-authentication taking 50–300 ms — long enough to drop a VoIP call. 802.11r reduces this to under 50 ms by pre-caching authentication credentials at neighboring APs, consistent with the low-latency requirements of voice traffic discussed in Kurose & Ross (2021, Ch. 3).

The WLC performs continuous **Radio Resource Management (RRM)**: automatically adjusting transmit power to fill coverage holes, assigning channels dynamically to minimize co-channel interference, and load-balancing clients across APs — applying the SDN automation principles of Kurose & Ross (2021, Ch. 4, Ch. 5) to the wireless layer.

At manufacturing sites, 5 GHz 802.11 is preferred to avoid interference from industrial equipment. Wireless OT infrastructure is kept physically separate from the IT Wi-Fi network to maintain OT zone isolation.

---

## Discussion

The Barrios network architecture was designed explicitly around the **protocol layer model** from Kurose & Ross (2021, Ch. 1). Each layer of the stack is addressed: physical cabling and link-layer VLANs; RFC 1918 private addressing, NAT, OSPF, and BGP at the network layer; TCP for reliable communication and UDP for latency-sensitive traffic at the transport layer; DNS, DHCP, and HTTPS at the application layer; and security controls native to every level — 802.1X at the link layer, IPsec at the network layer, TLS at the transport/application layer. This structured approach ensures security and connectivity are built into the architecture at each layer rather than added as an afterthought.

One of the most significant trade-offs in the design is the choice to use **both MPLS and SD-WAN**. MPLS provides carrier-grade SLA guarantees and private backbone routing essential for real-time applications on the most critical hub-to-hub and hub-to-data center paths. SD-WAN provides dramatically lower cost per Mbps, rapid deployment via zero-touch provisioning, and centralized SDN-style management — ideal for cost-sensitive satellite and manufacturing site connectivity. The hybrid approach mirrors the public Internet's own best-effort model (Kurose & Ross, 2021, Ch. 1): where the IP network makes no delivery guarantees, higher-layer overlays (MPLS, SD-WAN) create the reliability and predictability that applications require.

The **IT/OT convergence challenge** at manufacturing sites reflects a broader reality identified in Kurose & Ross (2021, Ch. 1): the Internet increasingly connects industrial sensors, actuators, and control systems. The strict network segmentation between IT and OT zones resolves the fundamental tension between IP networking's flexibility and industrial control's deterministic requirements. The IT/OT DMZ historian pattern allows data to flow from OT to IT analytics without exposing OT systems to IT network threats. This design requires ongoing governance — as pressure grows to integrate manufacturing data with cloud analytics platforms, the temptation to relax segmentation boundaries must be actively managed.

**DNS as critical infrastructure** is a recurring theme across the architecture. As Kurose & Ross (2021, Ch. 2) establish, virtually every application-layer connection begins with a DNS lookup — the mapping from human-readable hostname to IP address that makes the Internet usable. A DNS outage is effectively a complete network outage from the user's perspective even if all other infrastructure is functioning. The investment in redundant, regionally distributed DNS servers with full zone replication reflects this criticality. Treating DNS as a commodity rather than a high-availability infrastructure component is a common enterprise mistake that the Barrios design explicitly avoids.

The **3,000-user remote access** requirement illustrates the operational importance of scalable VPN infrastructure. Split tunneling balances security and performance: routing all Internet traffic through enterprise concentrators would triple WAN bandwidth requirements and add latency to cloud applications. The compromise — enterprise traffic through the VPN, SaaS traffic direct to Internet — is the pragmatic choice for most users, with full tunnel enforced only for the highest-sensitivity systems.

The **SNMP/NetFlow/Syslog/SIEM management stack** demonstrates that network management is not just an operational convenience but a security control. The SIEM's ability to correlate events across dozens of devices enables detection of multi-stage attacks invisible when each device is monitored in isolation — consistent with Kurose & Ross (2021, Ch. 8)'s treatment of operational security as an active, ongoing discipline rather than a one-time configuration task.

---

## Conclusion

This paper has presented a comprehensive network architecture for Barrios Manufacturing & Solutions, Inc., addressing all assignment specifications: U.S. headquarters (5,000 users), European main office (2,500 users), four data centers, eight satellite offices, seven manufacturing facilities, remote access for 3,000 users, inter-site WAN connectivity, network management, security, and wireless networking.

Every design decision is grounded in Kurose & Ross (2021). The three-tier hierarchical campus model provides scalability at headquarters and regional offices (Ch. 1, Ch. 6). Collapsed two-tier architecture scales to satellite offices. OSPF provides dynamic intra-domain routing with automatic convergence (Ch. 5). BGP manages multi-homed ISP connectivity (Ch. 5). MPLS and SD-WAN provide hybrid WAN connectivity appropriate to each site tier (Ch. 4, Ch. 5). DNS and DHCP provide foundational application-layer infrastructure for all users (Ch. 2). TCP and UDP transport enterprise and real-time traffic respectively (Ch. 3). TLS 1.3 (RFC 8446), IPsec ESP (RFC 4303), 802.1X, NGFWs, IDS/IPS, and SIEM provide layered security (Ch. 8). IEEE 802.11 with WPA3-Enterprise and fast roaming provides wireless connectivity (Ch. 7). SNMPv3 and centralized NMS provide network management visibility (Ch. 5).

The Barrios enterprise network is, at its core, an application of the Internet's fundamental design philosophy: a layered, packet-switched network built from standardized protocols at every layer, interconnected by routing protocols, and secured by cryptographic mechanisms applied at multiple levels. As Kurose & Ross (2021, Ch. 1) establish, the Internet is a "network of networks" — and the Barrios enterprise is a private overlay on that public infrastructure, using the same protocols, the same layered model, and the same engineering trade-offs as the global Internet at large. What distinguishes enterprise design is the addition of centralized management, enforced security boundaries, and guaranteed service levels — requirements addressed systematically throughout this paper using the foundational concepts of Computer Networks (CS640).

---

## References

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

RFC 8446 — *The Transport Layer Security (TLS) Protocol Version 1.3*. (2018). Internet Engineering Task Force. *(Cited in Kurose & Ross, Ch. 8 as the current TLS standard.)*

RFC 4303 — *IP Encapsulating Security Payload (ESP)*. (2005). Internet Engineering Task Force. *(Cited in Kurose & Ross, Ch. 8 as one of the two core IPsec protocols.)*

RFC 4302 — *IP Authentication Header (AH)*. (2005). Internet Engineering Task Force. *(Cited in Kurose & Ross, Ch. 8 alongside RFC 4303 as the second core IPsec protocol.)*
