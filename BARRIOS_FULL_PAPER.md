# Network Architecture Design for Barrios Manufacturing & Solutions, Inc.

**Imbar Shimshon**  
Department of Data Science, Monroe University, King Graduate School  
26WN-CS640-153W — Computer Networks  
Professor David Gianna  
March 28th, 2026

---

## Introduction

As enterprise organizations expand globally, computer networks become critical infrastructure that enables communication, coordination, and business operations. Modern computer networks consist of billions of connected devices running network applications at the network edge, enabling communication and data exchange across organizational systems (Kurose & Ross, 2021). The Internet itself is best understood as a "network of networks" — a layered, packet-switched system built from standardized protocols at every level of the protocol stack, from the physical medium up through the application layer (Kurose & Ross, Ch. 1). This layered architecture, known as the Internet protocol stack, provides the intellectual foundation for the design presented in this paper.

This paper proposes a network architecture design for the fictional global industrial automation company called ***Barrios Manufacturing & Solutions, Inc.*** The design addresses key challenges including inter-site connectivity, network resilience, secure access for both on-premises and remote users, and centralized infrastructure management. The proposed architecture follows a structured, top-down approach, and each section examines a core component of the design, forming a cohesive blueprint for enterprise network implementation. Intra-domain routing is provided by OSPF, inter-domain routing by BGP, WAN connectivity by a hybrid MPLS and SD-WAN architecture, and security is enforced at every protocol layer using IPsec, TLS, firewalls, and 802.1X network access control — all concepts grounded in the Kurose & Ross textbook.

---

## Company Overview & Site Inventory

Barrios Manufacturing & Solutions, Inc. is a fictitious mid-sized industrial automation company specializing in smart manufacturing systems, IoT-enabled sensors, and factory control platforms. The organization operates across North America and Europe, supporting both enterprise IT environments and operational technology (OT) systems within manufacturing facilities. As illustrated in **Figure 1** (see DIAGRAMS.md), the company maintains a distributed global footprint consisting of headquarters, regional offices, satellite locations, manufacturing facilities, and third-party-hosted data centers across multiple geographic regions. A detailed breakdown of site types, user distribution, and infrastructure components is provided in **Figure 2** below.

The organization includes a U.S. headquarters supporting approximately **5,000 users**, a European main office with **2,500 users**, and **eight satellite offices** across the U.S. and Europe, each supporting **50–250 users**. The company operates **seven manufacturing facilities** (two in the U.S. and five in Europe) that integrate both IT and OT environments, as well as multiple data centers supporting primary and disaster recovery operations. A significant remote workforce of approximately **3,000 users** requires secure and reliable access to corporate resources.

The network must support high availability, secure inter-site communication, and scalable connectivity for both on-premises and remote users. Given the diversity of environments — including office, data center, and manufacturing networks — the design must also ensure segmentation, centralized management, and consistent security controls across all locations.

**Figure 2 — Site Inventory Table**

| Site Type | Region | Count | Users / Notes |
|-----------|--------|-------|--------------|
| Headquarters | US (Austin, TX) | 1 | 5,000 users |
| Main Regional Office | Europe (Frankfurt) | 1 | 2,500 users |
| Satellite Offices | US | 5 | 50–250 users each |
| Satellite Offices | Europe | 3 | 50–250 users each |
| Manufacturing Facilities | US | 2 | IT + OT environments |
| Manufacturing Facilities | Europe | 5 | IT + OT environments |
| Third-Party Data Centers | US (Dallas, Chicago) | 2 | Colocation hosted |
| HQ Data Center | US (Austin) | 1 | On-premises at HQ |
| EU Data Center | Europe (Frankfurt) | 1 | On-premises at EU HQ |
| Remote Users (VPN) | Worldwide | — | ~3,000 concurrent |

---

## Network Architecture Overview

The Barrios enterprise network follows a hierarchical design model consisting of three layers: core, distribution, and access. This design aligns with standard enterprise networking principles, where each layer has a defined role in ensuring scalability, performance, and fault isolation. The core layer provides high-speed backbone connectivity, the distribution layer aggregates traffic and performs inter-VLAN routing, and the access layer connects end-user devices such as workstations, IP phones, and IoT systems (see **Figure 3** in DIAGRAMS.md).

At the wide-area network (WAN) level, the enterprise adopts a hybrid architecture integrating both traditional and modern networking technologies. Communication across the network is based on IPv4 addressing. Internally, the organization uses private IPv4 address ranges as defined in RFC 1918, which allow devices to communicate within the enterprise without requiring globally unique addresses. To enable communication with external networks, Network Address Translation (NAT) is implemented at the network edge (Kurose & Ross, Ch. 4). NAT translates internal private IP addresses into public IP addresses, conserving global address space while providing a layer of abstraction between internal and external networks.

For inter-site connectivity, the network incorporates both Multiprotocol Label Switching (MPLS) and Software-Defined Wide Area Networking (SD-WAN). MPLS provides reliable, low-latency communication between critical sites by forwarding traffic based on predefined labels rather than traditional IP routing (Ch. 4). SD-WAN enables flexible and cost-effective connectivity by routing traffic dynamically over Internet-based links, applying Software-Defined Networking (SDN) principles to the WAN: a centralized controller computes forwarding policies and distributes them to edge appliances, separating the control plane from the data plane (Ch. 4, Ch. 5).

The enterprise network follows a hub-and-spoke topology, with the U.S. headquarters acting as the primary hub and the European office serving as a regional hub. Dynamic routing protocols underpin automatic failover and efficient traffic management. **OSPF (Open Shortest Path First)**, a link-state intra-domain routing protocol, is used within the enterprise backbone; each router builds a complete topology map by flooding link-state advertisements and runs Dijkstra's shortest-path algorithm locally (Ch. 5). **BGP (Border Gateway Protocol)**, the inter-domain routing protocol that connects the Internet's autonomous systems, is used at the headquarters and European office edges to manage multi-homed ISP connectivity (Ch. 5).

---

## Headquarters Network Design (US HQ)

The U.S. headquarters in Austin, Texas supports approximately 5,000 users and serves as the central hub of the enterprise network. The campus network follows the three-tier hierarchical design — core, distribution, and access (see **Figure HQ-1** in DIAGRAMS.md).

The core layer provides high-speed backbone connectivity using redundant Layer 3 switches interconnected via 10/40 Gbps fiber links. The distribution layer aggregates traffic from access switches and performs inter-VLAN routing using Layer 3 switching running OSPF. The access layer connects end-user devices — workstations, IP phones, and IoT devices — via Power-over-Ethernet (PoE) switches that simultaneously deliver data and power to connected devices.

Traffic is segmented using VLANs based on department and function, consistent with IEEE 802.1Q tagging at the link layer (Kurose & Ross, Ch. 6). Standard VLANs deployed include: VLAN 10 for user workstations, VLAN 20 for VoIP, VLAN 30 for wireless clients, VLAN 40 for management and infrastructure, VLAN 50 for guest Internet access (isolated), and VLAN 60 for the server DMZ. Inter-VLAN routing is performed at the distribution layer.

At the network edge, the headquarters connects to two ISPs using BGP for redundancy. If one ISP fails, BGP converges and all traffic shifts to the surviving link. A pair of next-generation firewalls (NGFWs) provides stateful perimeter security, intrusion prevention, and deep packet inspection. The headquarters also hosts the primary VPN concentrator, the HQ data center, and centralized infrastructure services including DNS and DHCP for the local campus.

DNS is critical infrastructure at the HQ: local DNS servers resolve hostnames for 5,000 users without relying on external lookups for every query, following the hierarchical DNS model described in Kurose & Ross (Ch. 2). DHCP servers automatically assign IP addresses, subnet masks, default gateway, and DNS server options to all LAN hosts. Quality of Service (QoS) policies at the distribution layer prioritize latency-sensitive UDP traffic — particularly VoIP — consistent with the discussion of transport-layer service requirements in Ch. 3.

---

## European Main Office Design

The European main office, located in Frankfurt, Germany, serves as the regional hub for all European operations, supporting **2,500 users**. Frankfurt was selected as the European hub because it is one of the world's largest Internet exchange points (IXP), providing access to multiple Tier-1 and Tier-2 ISPs with low-latency, high-bandwidth connectivity to both the U.S. headquarters and European satellite offices.

The European campus mirrors the three-tier hierarchical design of the U.S. headquarters. The core layer uses redundant Layer 3 switches interconnected via fiber. The distribution layer aggregates floor-level traffic and enforces VLAN segmentation and inter-VLAN routing via OSPF. The access layer connects end-user devices with PoE switches throughout the building.

The European office is assigned a unique RFC 1918 private subnet (10.20.0.0/16), separate from the U.S. HQ address space (10.10.0.0/16), preventing address overlap across the enterprise. NAT is performed at the European edge router for outbound Internet traffic. A pair of local DNS servers is deployed to resolve hostnames for 2,500 users without relying on transatlantic WAN links for every DNS query (Ch. 2). Local DHCP servers assign addressing automatically to all LAN hosts.

The European office connects to the U.S. headquarters via a dedicated MPLS private WAN circuit traversing a transatlantic carrier network. A secondary SD-WAN over broadband connection provides failover capacity. The SD-WAN controller monitors link quality continuously and reroutes application traffic to the best-available path when degradation is detected. Dual ISP connections — one from a European Tier-1 carrier and one from a regional broadband provider — provide Internet redundancy managed by eBGP.

All critical components including core switches, edge routers, and firewalls are deployed in redundant pairs. HSRP or VRRP provides first-hop redundancy so that if the primary default gateway fails, clients automatically failover to the standby gateway without manual reconfiguration (see **Figure EU-1** in DIAGRAMS.md).

---

## Data Center Design

Barrios operates **four data centers**: HQ-DC (on-premises in Austin), US-DC1 (colocation in Dallas), US-DC2 (colocation in Chicago), and EU-DC (on-premises in Frankfurt). This distributed architecture provides geographic redundancy, business continuity, and low-latency access to compute and storage resources for users across all sites.

Each data center uses a **leaf-spine topology** internally. In this architecture, leaf switches connect directly to servers and uplink to every spine switch; spine switches form the high-speed backbone interconnecting all leaf switches. Any server can reach any other server in exactly two hops (leaf → spine → leaf), providing consistent, low-latency, high-bandwidth communication within the data center (see **Figure DC-2** in DIAGRAMS.md). This directly reflects the forwarding principles of Kurose & Ross (Ch. 4), where each switch makes forwarding decisions at line rate.

Each data center is assigned its own private address range: HQ-DC uses 10.100.0.0/16, US-DC1 uses 10.101.0.0/16, US-DC2 uses 10.102.0.0/16, and EU-DC uses 10.200.0.0/16. OSPF is run within each data center to distribute routing information among leaf and spine switches (Ch. 5).

US-DC1 and US-DC2 connect to the enterprise WAN via dedicated MPLS circuits from the colocation facility's meet-me room, ensuring data center traffic never traverses the public Internet. Each data center edge implements a **dual-firewall DMZ**: an external firewall faces the Internet, and an internal firewall separates the DMZ from the internal server farm. Public-facing services are placed in the DMZ; internal databases and application servers sit behind the second firewall. As described in Kurose & Ross (Ch. 8), this dual-firewall architecture ensures that an attacker who compromises a DMZ server still faces the internal firewall before reaching core systems (see **Figure DC-3** in DIAGRAMS.md).

Primary DNS zone data for `barrios.internal` is mastered in HQ-DC and replicated to EU-DC and US-DC1, consistent with the DNS hierarchy described in Ch. 2. US-DC1 and US-DC2 provide active-active failover for U.S. workloads; EU-DC provides active-standby for European workloads with HQ-DC as the standby. All data replication between data centers is encrypted with IPsec (Ch. 8).

---

## Satellite Office Design

Barrios operates **eight satellite offices** — five in the U.S. and three in Europe — each supporting **50–250 users**. These offices use a **collapsed two-tier architecture**, combining core and distribution functions into a single redundant Layer 3 switch pair, with a separate access layer for end-user connectivity (see **Figure SAT-1** in DIAGRAMS.md). A full three-tier design would add unnecessary cost and complexity at this scale.

The collapsed core switch performs inter-VLAN routing, runs a local OSPF instance, and connects directly to the WAN SD-WAN appliance. A small number of access switches (2–4 per office) connect workstations, IP phones, and wireless access points. Standard VLANs match the enterprise template: VLAN 10 for users, VLAN 20 for VoIP, VLAN 30 for wireless, VLAN 40 for management, and VLAN 50 for guest Internet (isolated).

Each satellite office is assigned a dedicated /24 subnet from the enterprise RFC 1918 space, providing up to 254 usable host addresses — sufficient for the maximum 250-user site. DHCP services are provided by the regional hub's central DHCP server via a **DHCP relay agent** configured on the local Layer 3 switch, which forwards DHCP Discover messages across the WAN to the central server (Ch. 2). Local DNS resolution proxies queries to the regional hub's DNS servers.

Satellite offices connect to their regional hub using SD-WAN over broadband as the primary model, with a 4G/5G cellular backup link for sites where a second broadband provider is unavailable. All WAN traffic is encrypted using IPsec (Ch. 8). SD-WAN policy routes VoIP and ERP traffic through the encrypted tunnel to the hub, while Internet-bound traffic (SaaS applications, web browsing) breaks out locally at the SD-WAN appliance — a technique known as split tunneling — reducing WAN bandwidth consumption and improving latency for cloud applications.

---

## Manufacturing Site Design

Barrios operates **seven manufacturing facilities** — two in the U.S. and five in Europe. Manufacturing sites present unique networking challenges due to the co-existence of **Information Technology (IT)** and **Operational Technology (OT)** systems. OT systems include PLCs (Programmable Logic Controllers), SCADA (Supervisory Control and Data Acquisition) servers, manufacturing execution systems (MES), and IoT-enabled sensors on the factory floor. These systems often run legacy protocols (Modbus, DNP3, EtherNet/IP, PROFINET) and have strict real-time requirements — sub-millisecond deterministic communication — that cannot tolerate the congestion or jitter acceptable in standard IT environments (Kurose & Ross, Ch. 4).

The manufacturing network is divided into three strictly separated zones. The **IT Zone** follows the same collapsed two-tier architecture as satellite offices, supporting administrative users and connecting to the enterprise WAN. The **OT Zone** is an isolated network segment dedicated to all industrial control and automation equipment, using dedicated Layer 2/Layer 3 infrastructure physically separated from the IT network. The **IT/OT DMZ** — implemented via a next-generation firewall with deep packet inspection — sits between the two zones and hosts data historians (which collect OPC-UA time-series data from OT devices), MES integration interfaces, and remote-access jump servers for vendor engineers (see **Figure MFG-1** in DIAGRAMS.md).

This three-zone architecture directly implements the defense-in-depth security model from Kurose & Ross (Ch. 8): multiple overlapping security boundaries ensure that a compromise in one zone does not cascade into adjacent zones.

OT traffic never traverses the enterprise WAN — all industrial control communication stays local. The OT network uses **Industrial Ethernet** (IEEE 802.3-based ruggedized switches) and IEEE 802.1p QoS markings to prioritize real-time control traffic (Ch. 6). Rapid Spanning Tree Protocol (RSTP, IEEE 802.1w) provides link-layer redundancy with sub-50ms failover times.

IoT sensors on the factory floor connect via either wired Industrial Ethernet or 802.11 wireless access points in ruggedized, weather-resistant enclosures. Sensor data is aggregated by the data historian in the IT/OT DMZ and forwarded to the analytics platform in the data center. The 5 GHz 802.11 band is preferred for industrial wireless to avoid interference from variable-frequency drives and welding equipment common in manufacturing environments (Ch. 7).

---

## Remote Access (VPN)

Barrios employs approximately **3,000 sales representatives and remote employees** worldwide, all requiring secure access to corporate resources from home networks, hotels, co-working spaces, and mobile devices. The remote access solution is built on **IPsec VPN** and **TLS-based VPN** technologies — both described in detail in Kurose & Ross, Ch. 8.

VPN concentrators are deployed in geographically redundant pairs at two locations: US-VPN-1 and US-VPN-2 at the U.S. headquarters (serving North American remote users), and EU-VPN-1 and EU-VPN-2 at the European main office (serving European remote users). Each pair operates in active-active load balancing with automatic session failover.

The primary remote access method for most users is an **SSL/TLS VPN**. A lightweight VPN agent on the user's device establishes an encrypted TLS tunnel to the concentrator over HTTPS (TCP port 443). TLS 1.3 — standardized in RFC 8446 and described in Ch. 8 — is required for all connections. TLS 1.3 provides: **confidentiality** via AES-256-GCM symmetric encryption, **integrity** via SHA-384 cryptographic hashing, and **authentication** via the concentrator's digital certificate (signed by the corporate PKI), allowing the client to verify it is connecting to a legitimate Barrios gateway before transmitting credentials.

For company-managed laptops requiring a higher security profile, **IPsec VPN in tunnel mode** is used. As described in Ch. 8, IPsec tunnel mode encapsulates the entire original IP packet in a new encrypted outer packet, so an eavesdropper sees only an encrypted blob between two public IP addresses. The **ESP (Encapsulating Security Payload)** protocol within IPsec provides both AES-256-GCM encryption and HMAC-SHA-256 integrity (see **Figure VPN-2** in DIAGRAMS.md). IKEv2 handles session establishment and automatic re-keying.

All VPN sessions require **multi-factor authentication (MFA)**: corporate credentials verified against Active Directory plus a time-based one-time password (TOTP) from an authenticator app. This implements the multi-factor authentication principle described in Ch. 8 — authentication based on something the user knows and something the user has — providing protection even if credentials are stolen.

**Split tunneling** is the default VPN policy: enterprise traffic (internal applications, file servers, data centers) routes through the encrypted tunnel, while Internet-bound SaaS traffic (Microsoft 365, Salesforce, Zoom) exits locally from the user's broadband connection. This reduces bandwidth load on the concentrators and improves performance for cloud applications. Full tunnel mode is enforced for users accessing manufacturing or financial systems, ensuring all traffic passes through enterprise security controls (see **Figure VPN-1** in DIAGRAMS.md).

---

## Connectivity Between Sites

Inter-site connectivity is the backbone of the Barrios global enterprise. All distributed sites — headquarters, regional offices, satellite offices, manufacturing facilities, data centers, and remote users — are connected through a **hybrid WAN** architecture combining three technologies (see **Figure WAN-1** in DIAGRAMS.md).

**MPLS (Multiprotocol Label Switching)** serves the most critical links: HQ to data centers, HQ to EU Office, and larger manufacturing sites. MPLS assigns labels to packets at ingress and forwards them along predetermined Label Switched Paths (LSPs) through the carrier's private backbone — packets are never exposed to the public Internet. This provides carrier-grade SLA guarantees for bandwidth and latency. The forwarding mechanism directly relates to the network-layer forwarding concepts of Ch. 4, where MPLS simply uses labels as the forwarding table lookup key instead of IP destination addresses.

**SD-WAN over broadband Internet** serves satellite offices and smaller manufacturing sites. The SD-WAN control plane (a centralized controller) computes and distributes forwarding policies to SD-WAN edge appliances at each site — a direct application of the SDN architecture described in Ch. 4 and Ch. 5 (see **Figure WAN-2** in DIAGRAMS.md). SD-WAN provides: application-aware routing (VoIP gets the lowest-latency path, bulk backup traffic gets scheduled off-peak), dynamic path selection (automatic failover when a link degrades), and zero-touch provisioning (new sites brought online by shipping a pre-configured appliance that dials home to the controller).

All SD-WAN traffic and all traffic over public Internet links is encrypted using **IPsec tunnel mode** (Ch. 8), protecting enterprise data even when crossing shared infrastructure.

**OSPF** serves as the intra-domain routing protocol across the enterprise backbone (Ch. 5). The network is divided into OSPF areas: Area 0 (backbone, interconnecting HQ, data centers, and EU office via MPLS), Area 1 (U.S. satellite and manufacturing sites), and Area 2 (European satellite and manufacturing sites). If any WAN link fails, OSPF reconverges within seconds, automatically rerouting traffic to the next-best path.

**BGP** manages multi-homed Internet connectivity at the HQ and EU office edges (Ch. 5). eBGP sessions are maintained with each of two ISPs at each site. BGP attributes (AS path, local preference) are tuned to distribute traffic across ISPs during normal operation and achieve automatic failover within minutes if an ISP link fails (see **Figure WAN-3** in DIAGRAMS.md).

The transatlantic MPLS circuit between Austin and Frankfurt traverses undersea fiber infrastructure, with a round-trip time (RTT) of approximately 130–160 ms — a physical constraint determined by the speed of light in fiber. Application-layer protocols running over TCP must be designed to tolerate this latency; WAN optimization (TCP window scaling) is applied for latency-sensitive enterprise applications (Ch. 3).

---

## Network Management & Monitoring

Managing a globally distributed enterprise network requires comprehensive, centralized visibility and control. Without systematic monitoring, administrators have no awareness of performance degradation, configuration drift, or security incidents until users report problems.

**SNMP (Simple Network Management Protocol)** is the foundational network management protocol described in Kurose & Ross (Ch. 5). SNMP uses a manager-agent architecture: agents running on every managed device (routers, switches, firewalls, access points) maintain a Management Information Base (MIB) containing device state variables — interface utilization, error counters, CPU and memory usage, routing table entries. The central SNMP manager periodically polls agents with GET requests and receives asynchronous TRAP notifications when significant events occur (interface failure, threshold exceeded). Barrios deploys **SNMPv3** exclusively — SNMPv1 and SNMPv2c use plaintext community strings for authentication, which is unacceptable for a security-conscious enterprise. SNMPv3 adds HMAC-based message authentication and AES encryption (Ch. 8), consistent with the principle that management traffic must be as well protected as user traffic (see **Figure MGMT-1** in DIAGRAMS.md).

**NetFlow/IPFIX** supplements SNMP by providing traffic flow analysis. Routers export flow records — summarizing every network conversation (source/destination IP, protocol, port, byte counts) — to a centralized flow collector. This allows the network team to identify top bandwidth consumers, detect anomalous traffic patterns that may indicate security incidents, and perform long-term capacity planning.

**Syslog** aggregates event logs from all network devices to centralized syslog servers, providing an audit trail of configuration changes, login events, and error conditions. Log retention is 90 days on hot storage and 1 year on archive, meeting regulatory compliance requirements.

A centralized **Network Management System (NMS)** deployed at HQ-DC (with a standby at US-DC1) provides topology discovery, fault monitoring, performance dashboards, configuration backup, and threshold alerting across all sites. The **SD-WAN controller** provides its own management plane for all SD-WAN-connected sites, presenting a single-pane-of-glass view of WAN link quality and application performance — applying the SDN centralized control principle of Ch. 4 to day-to-day operations.

A **SIEM (Security Information and Event Management)** platform aggregates logs from all firewalls, IDS sensors, VPN concentrators, and endpoint agents, correlating events across devices to detect attack patterns invisible in individual device logs (see **Figure MGMT-2** in DIAGRAMS.md). For example, a failed VPN authentication followed by a successful authentication from a different country five minutes later generates a high-priority SIEM alert for investigation. This operational security function is described in Kurose & Ross (Ch. 8) as a key component of enterprise network defense.

All network device configuration is managed via a Network Configuration Management (NCM) system with role-based access control, automated compliance checking against baseline templates, and full version-controlled change tracking. SSH with public-key authentication is the exclusive management access protocol; Telnet is disabled globally (Ch. 8 symmetric/asymmetric key principles).

A dedicated **out-of-band (OOB) management network** connects to the console ports of all critical infrastructure at major sites. If a misconfiguration disables the production network, administrators can still reach all devices via the OOB network to diagnose and recover the outage — a direct application of redundancy principles established in Ch. 1.

---

## Security Design

Network security at Barrios is not a single product but a comprehensive design philosophy applied at every layer of the protocol stack. Kurose & Ross (Ch. 8) define the core properties of network security as: **confidentiality** (only intended parties can read data), **authentication** (verifying the identity of communicating parties), **message integrity** (ensuring data is not altered in transit), and **access and availability** (ensuring services remain accessible to authorized users). The Barrios architecture addresses all four at each protocol layer through a **defense-in-depth** model.

**Perimeter Firewalls:** Next-generation firewalls are deployed at every network boundary. As described in Ch. 8, stateless packet filters examine individual packets by source/destination IP and port, while stateful packet filters track connection state and only permit packets belonging to established legitimate sessions. Barrios NGFWs extend stateful inspection with Layer 7 application awareness, identifying applications by behavioral signature rather than port number alone. Firewall rule sets follow the principle of least privilege: all traffic is denied by default, and only explicitly permitted flows are allowed. At the headquarters and data center perimeters, a dual-firewall DMZ ensures that an attacker who compromises a DMZ server still faces the internal firewall before reaching core network resources (see **Figure SEC-2** in DIAGRAMS.md).

**Intrusion Detection and Prevention (IDS/IPS):** As described in Ch. 8, IDS devices passively monitor traffic for signatures of known attack patterns or anomalous behavior; IPS devices extend this by blocking detected threats inline. Barrios deploys network IPS inline at the headquarters and European office Internet perimeters, and passive IDS sensors at the IT/OT boundary in manufacturing sites (where inline blocking could disrupt real-time control traffic). Host-based IDS agents on all data center servers monitor for unauthorized file changes and privilege escalation. All IDS/IPS alerts feed the SIEM (see **Figure SEC-1** in DIAGRAMS.md).

**IPsec for WAN Encryption:** All site-to-site SD-WAN tunnels and all remote access VPN sessions use IPsec ESP in tunnel mode with AES-256-GCM encryption and HMAC-SHA-256 integrity (Ch. 8). IPsec provides datagram-level protection — every IP packet is individually encrypted and authenticated — distinct from TLS, which protects only the payload of a TCP session.

**TLS 1.3 for Application Traffic:** All internal and external web-based applications and APIs use TLS 1.3 (Ch. 8). AES-256-GCM provides confidentiality, HMAC-SHA-384 provides integrity, and RSA-2048 or ECDSA-256 digital certificates enable server authentication. Certificates are issued by the corporate PKI anchored by a Hardware Security Module. HTTP Strict Transport Security (HSTS) is enforced on all web properties.

**Network Access Control (802.1X):** IEEE 802.1X port-based authentication is deployed on all wired and wireless access ports. Before any device gains network access, it must authenticate with the RADIUS server — using EAP-TLS (certificate-based, for managed devices) or EAP-MSCHAPv2 (credential-based, for BYOD). Unauthenticated devices are placed in a quarantine VLAN with Internet-only access (see **Figure SEC-3** in DIAGRAMS.md).

**VLAN Segmentation as Access Control:** VLAN segmentation (Ch. 6) limits the blast radius of any compromise. An infected workstation in VLAN 10 cannot directly reach database servers in VLAN 60 without traversing a firewall and ACL-controlled Layer 3 hop. This reduces the ability of an attacker to move laterally through the network after an initial breach.

**Manufacturing Site Security:** The IT/OT DMZ firewall permits only specific historian protocols (OPC-UA) from the OT zone to the DMZ; all other IT-to-OT traffic is blocked. Vendor remote access to OT systems is only possible via the jump server in the DMZ, with session recording and time-limited credentials. USB ports are locked down on all OT workstations to prevent removable-media attacks. Passive OT network monitoring (using industrial security platforms with deep packet inspection of industrial protocols) operates without introducing traffic that could interfere with real-time control systems.

**Encryption at Rest:** Full disk encryption (BitLocker/FileVault) is enforced on all company-managed laptops. Database encryption protects all PII, financial data, and manufacturing intellectual property. All backup data is encrypted with AES-256 before being written to storage.

**Security Controls by Layer:**

| Layer | Control | Technology |
|-------|---------|-----------|
| Physical | Locked network rooms, badge access | — |
| Link | 802.1X NAC, VLAN segmentation | IEEE 802.1X, 802.1Q |
| Network | IPsec, firewall ACLs, IPS | IPsec ESP, NGFWs |
| Transport | TLS 1.3 for all app traffic | TLS, PKI |
| Application | MFA, WAF, secure coding | RADIUS, TOTP |
| Management | SNMPv3, SSH only, SIEM | SNMPv3, AES |

---

## Wireless Network Design

Wireless LAN connectivity is required across all Barrios facilities. Kurose & Ross (Ch. 7) describe IEEE 802.11 as the dominant wireless LAN standard, covering the physical and link-layer behavior of wireless networks, the CSMA/CA medium access protocol, and the challenges of the wireless medium (shared medium, hidden node problem, multipath fading).

Barrios deploys **IEEE 802.11ax (Wi-Fi 6)** as the standard for all new wireless infrastructure. Wi-Fi 6 introduces OFDMA (enabling simultaneous transmission to multiple clients on different subcarriers), BSS Coloring (reducing interference between neighboring APs), and Target Wake Time (reducing power consumption for IoT devices). Wi-Fi 6E access points — which add the 6 GHz band — are deployed in high-density areas such as headquarters conference rooms and large open-plan offices, where the less-congested 6 GHz spectrum dramatically reduces interference.

As described in Kurose & Ross (Ch. 7), wireless networks use **CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance)** for medium access. Because wireless devices cannot simultaneously transmit and receive on the same channel, collision detection is not possible; instead, devices sense the channel before transmitting and use random backoff timers to reduce the probability of simultaneous transmissions (see **Figure WIFI-3** in DIAGRAMS.md). The hidden node problem — where two devices are both in range of the AP but not each other — is addressed by the optional RTS/CTS (Request to Send / Clear to Send) mechanism.

Barrios uses a **centralized wireless LAN controller (WLC)** architecture. Lightweight access points (LWAPs) forward all wireless frames to the WLC via CAPWAP tunnels; the WLC handles all authentication, roaming decisions, radio resource management, and policy enforcement. This directly mirrors the SDN separation of control plane and data plane described in Ch. 4: the WLC is the centralized controller, and APs are forwarding devices (see **Figure WIFI-1** in DIAGRAMS.md).

Each facility broadcasts multiple SSIDs mapped to dedicated VLANs. The corporate SSID (`BARRIOS-CORP`) uses **WPA3-Enterprise** with 802.1X/RADIUS authentication — the strongest currently available Wi-Fi security mode, addressing known vulnerabilities in WPA2 that were exploited by KRACK and similar attacks (Ch. 8). A BYOD SSID provides access for employee personal devices with MDM certificate enforcement. A guest SSID places visitors in isolated VLAN 50 with Internet-only access through a captive portal.

**IEEE 802.11r (Fast BSS Transition)** is deployed on corporate and voice SSIDs to support seamless roaming. Without 802.11r, roaming between APs triggers a full 802.1X re-authentication (50–300 ms) — long enough to drop a VoIP call. 802.11r reduces re-authentication time to under 50 ms by pre-caching authentication credentials at neighboring APs, consistent with the VoIP latency requirements discussed in Ch. 3.

The WLC continuously performs **Radio Resource Management (RRM)**: automatically adjusting transmit power to fill coverage holes, assigning channels dynamically to minimize co-channel interference, and load-balancing clients across APs. This autonomous management capability reflects the SDN automation principles of Ch. 4 and Ch. 5.

At manufacturing sites, 5 GHz 802.11 is preferred to avoid interference from industrial equipment. OT wireless infrastructure is kept physically separate from the IT Wi-Fi network to prevent interference and maintain OT zone isolation.

---

## Discussion

The Barrios network architecture was designed explicitly around the **protocol layer model** from Kurose & Ross (Ch. 1). Each layer of the stack is addressed: physical cabling and 802.1Q VLANs at the link layer; IP addressing with RFC 1918, NAT, OSPF, and BGP at the network layer; TCP for reliable application communication and UDP for latency-sensitive traffic at the transport layer; DNS, DHCP, and HTTPS at the application layer; and security controls embedded at every level from 802.1X at the link layer to TLS at the transport/application layer and IPsec at the network layer. This structured, top-down approach ensures that security and connectivity are native to every layer of the architecture.

One of the most significant design decisions is the choice to use **both MPLS and SD-WAN** rather than either one exclusively. MPLS provides carrier-grade SLA guarantees, predictable latency, and private backbone routing — essential for critical hub-to-hub and hub-to-data center paths where real-time applications demand sub-50ms latency. SD-WAN provides dramatically lower cost per Mbps, rapid deployment via zero-touch provisioning, and centralized SDN-style management — ideal for the cost-sensitive satellite office and manufacturing connectivity. The hybrid approach reflects the real-world enterprise trade-off between performance guarantees and cost efficiency, and mirrors the "best-effort service" model of the public Internet (Ch. 1): where the IP network itself makes no guarantees, higher-layer overlays (MPLS, SD-WAN) create the reliability that applications require.

The **IT/OT convergence challenge** at manufacturing sites highlights one of the most pressing issues in modern enterprise networking. As Ch. 1 establishes, the Internet increasingly connects industrial sensors, actuators, and control systems — creating both operational opportunities and significant security risks. The strict segmentation between IT and OT zones resolves the fundamental tension between the flexibility of IP networking and the deterministic requirements of industrial control. The IT/OT DMZ historian pattern allows data to flow from OT to IT analytics without exposing OT systems to IT network threats. This design requires ongoing governance, however: as organizations push for greater integration (feeding manufacturing data into cloud analytics), the temptation to relax segmentation boundaries must be actively resisted.

**DNS as critical infrastructure** is a theme that spans the entire architecture. As Ch. 2 emphasizes, virtually every application-layer connection begins with a DNS lookup; a DNS outage is effectively a network outage from the user's perspective. The investment in redundant DNS servers in every region, with zone replication and split-horizon DNS, reflects this criticality. A common enterprise mistake is to treat DNS as a commodity service rather than a high-availability infrastructure component — the Barrios design treats it as the latter.

The **3,000-user remote access** requirement underscores the operational importance of scalable VPN infrastructure. Split tunneling balances security with performance: routing all Internet traffic through the enterprise concentrators would triple WAN bandwidth requirements while adding latency to cloud applications. The compromise — enterprise traffic through the VPN, SaaS traffic direct to Internet — is the pragmatic choice for general users, with full tunnel reserved for the highest-sensitivity systems.

Finally, the **SNMP/NetFlow/Syslog/SIEM management stack** demonstrates that network management is not just an operational convenience — it is a security control. The SIEM's ability to correlate events across dozens of devices enables detection of multi-stage attacks that would be invisible if each device were monitored in isolation. This aligns with Ch. 8's treatment of operational security as an active, ongoing discipline rather than a one-time configuration task.

---

## Conclusion

This paper has presented a comprehensive network architecture design for Barrios Manufacturing & Solutions, Inc., a fictional global industrial automation company operating across the United States and Europe. The design addresses all components of the assignment specifications: U.S. headquarters, European main office, data centers (one at each main office plus two U.S. colocation facilities), satellite offices (five U.S., three European), manufacturing facilities (two U.S., five European), remote access for 3,000 users, inter-site connectivity, network management, security, and wireless networking.

Every design decision is grounded in the networking concepts presented in Kurose & Ross (2021), drawing specifically from Chapters 1 through 8. The three-tier hierarchical campus model (Ch. 1, Ch. 6) provides scalability at headquarters and regional offices. Collapsed two-tier architecture scales to satellite offices. OSPF (Ch. 5) provides dynamic intra-domain routing with automatic convergence. BGP (Ch. 5) manages multi-homed Internet connectivity. MPLS and SD-WAN (Ch. 4, Ch. 5) provide hybrid WAN connectivity appropriate to each site tier. DNS and DHCP (Ch. 2) provide the foundational application-layer infrastructure for all users. TCP and UDP (Ch. 3) transport enterprise and real-time traffic respectively. TLS 1.3, IPsec, 802.1X, NGFWs, IDS/IPS, and SIEM (Ch. 8) provide layered security. IEEE 802.11ax with WPA3-Enterprise and 802.11r (Ch. 7) provide enterprise wireless connectivity with seamless roaming. SNMPv3, NetFlow, and centralized NMS (Ch. 5) provide comprehensive network management visibility.

The architecture of the Barrios network is, at its core, an application of the Internet's fundamental design philosophy: a layered, packet-switched network built from standardized protocols, interconnected by routing protocols, and secured by cryptographic mechanisms applied at multiple levels. As Kurose & Ross (2021) establish from the very first chapter, the Internet is a "network of networks" — and the Barrios enterprise network is itself a private overlay on that public infrastructure, using the same protocols, the same layered model, and the same engineering trade-offs as the global Internet at large. What distinguishes enterprise network design is the addition of centralized management, enforced security boundaries, and guaranteed service levels — requirements addressed systematically throughout this paper using the concepts taught in Computer Networks (CS640) as the intellectual foundation.

---

## References

Kurose, J. F., & Ross, K. W. (2021). *Computer networking: A top-down approach* (8th ed.). Pearson.

IEEE Standard 802.1Q-2018 — *Bridges and Bridged Networks (VLANs)*. IEEE.

IEEE Standard 802.11ax-2021 — *Wireless LAN Medium Access Control (MAC) and Physical Layer (PHY) Specifications (Wi-Fi 6)*. IEEE.

RFC 4271 — *A Border Gateway Protocol 4 (BGP-4)*. Internet Engineering Task Force.

RFC 2328 — *OSPF Version 2*. Internet Engineering Task Force.

RFC 4301 — *Security Architecture for the Internet Protocol (IPsec)*. IETF.

RFC 8446 — *The Transport Layer Security (TLS) Protocol Version 1.3*. IETF.

RFC 1918 — *Address Allocation for Private Internets*. IETF.
