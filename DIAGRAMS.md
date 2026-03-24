# DIAGRAMS.md — Barrios Manufacturing & Solutions Network Architecture
## All Network Diagrams for Term Paper

> Each diagram below corresponds to a `Figure` reference in the paper sections.  
> Mermaid diagrams can be rendered in VS Code, GitHub, Obsidian, or any Mermaid-compatible viewer.  
> ASCII diagrams can be pasted directly into Word/Google Docs as monospace (Courier New) code blocks.

---

## Figure 1 — Global Site Distribution (Company Overview)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  BARRIOS MANUFACTURING & SOLUTIONS, INC.                │
│                        Global Network Footprint                         │
├─────────────────────────────────┬───────────────────────────────────────┤
│        UNITED STATES            │              EUROPE                   │
│                                 │                                       │
│  ★ HQ — Austin, TX (5,000)     │  ★ EU HQ — Frankfurt (2,500)         │
│                                 │                                       │
│  ● Satellite Office ×5          │  ● Satellite Office ×3               │
│    (50–250 users each)          │    (50–250 users each)               │
│                                 │                                       │
│  ▲ Manufacturing ×2             │  ▲ Manufacturing ×5                  │
│                                 │                                       │
│  ■ Data Center HQ (Austin)      │  ■ Data Center EU (Frankfurt)        │
│  ■ Data Center US-DC1 (Dallas)  │                                       │
│  ■ Data Center US-DC2 (Chicago) │                                       │
│                                 │                                       │
│  ⊕ Remote Users (worldwide): 3,000 sales reps & remote employees       │
└─────────────────────────────────┴───────────────────────────────────────┘
```

---

## Figure 2 — Site Inventory Table (Company Overview)

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

## Figure 3 — Hierarchical Network Design (Network Architecture Overview)

```
                    THREE-TIER CAMPUS HIERARCHY
                    (Kurose & Ross, Ch. 1, Ch. 6)

    ┌──────────────────────────────────────────────────────────┐
    │                    CORE LAYER                            │
    │         High-speed backbone (fiber, redundant)           │
    │    [Core Switch A] ════════════ [Core Switch B]          │
    │         (10/40 Gbps fiber interconnects)                 │
    └──────────────────────────┬───────────────────────────────┘
                               │
    ┌──────────────────────────▼───────────────────────────────┐
    │               DISTRIBUTION LAYER                         │
    │    Inter-VLAN routing • Policy enforcement • OSPF        │
    │  [Dist Switch 1]  [Dist Switch 2]  [Dist Switch 3]       │
    │       (Layer 3 switches — one per building/zone)         │
    └──────────────────────────┬───────────────────────────────┘
                               │
    ┌──────────────────────────▼───────────────────────────────┐
    │                   ACCESS LAYER                           │
    │           End-user device connectivity (PoE)             │
    │  [Acc Sw] [Acc Sw] [Acc Sw] [Acc Sw] [Acc Sw] [Acc Sw]  │
    │     │        │        │        │        │        │       │
    │  PCs/VoIP  PCs/VoIP  APs    PCs/VoIP  IoT   PCs/VoIP   │
    └──────────────────────────────────────────────────────────┘
```

---

## Figure WAN-1 — Enterprise Global WAN Topology

```mermaid
graph TB
    subgraph US["🇺🇸 United States"]
        HQ["★ US HQ\nAustin, TX\n5,000 users"]
        DC1["■ HQ-DC\nAustin"]
        UDC1["■ US-DC1\nDallas"]
        UDC2["■ US-DC2\nChicago"]
        SAT_US["● US Satellite\nOffices ×5"]
        MFG_US["▲ US Manufacturing\nFacilities ×2"]
    end

    subgraph EU["🇪🇺 Europe"]
        EUHQ["★ EU HQ\nFrankfurt\n2,500 users"]
        EUDC["■ EU-DC\nFrankfurt"]
        SAT_EU["● EU Satellite\nOffices ×3"]
        MFG_EU["▲ EU Manufacturing\nFacilities ×5"]
    end

    REMOTE["⊕ Remote Users\n3,000 worldwide"]

    HQ <-->|"MPLS Primary\nSD-WAN Backup"| EUHQ
    HQ <-->|"Private Fiber"| DC1
    HQ <-->|"MPLS"| UDC1
    HQ <-->|"MPLS"| UDC2
    EUHQ <-->|"Private Link"| EUDC
    HQ <-->|"SD-WAN / IPsec"| SAT_US
    EUHQ <-->|"SD-WAN / IPsec"| SAT_EU
    HQ <-->|"SD-WAN / IPsec"| MFG_US
    EUHQ <-->|"SD-WAN / IPsec"| MFG_EU
    REMOTE <-->|"SSL/TLS VPN\nor IPsec VPN"| HQ
    REMOTE <-->|"SSL/TLS VPN\nor IPsec VPN"| EUHQ
```

---

## Figure WAN-2 — MPLS Hub-and-Spoke with SD-WAN Overlay

```
                    ╔══════════════════════════════════════╗
                    ║         CARRIER MPLS CLOUD           ║
                    ║   (Label Switched Paths — Ch. 4, 5)  ║
                    ╚══════════╤═══════════╤═══════════════╝
                               │           │
               ┌───────────────┘           └─────────────────────┐
               │                                                  │
    ┌──────────▼──────────┐                         ┌────────────▼───────────┐
    │    US HQ (Hub)      │                         │   EU HQ (Regional Hub) │
    │  Austin, TX         │ ◄─── Transatlantic ───► │   Frankfurt            │
    │  BGP Dual-ISP       │      MPLS Circuit        │   BGP Dual-ISP         │
    └──────────┬──────────┘                         └────────────┬───────────┘
               │  SD-WAN IPsec Tunnels (Spokes)                  │
         ┌─────┴─────┬──────────┬──────────┐           ┌─────────┴──────┬────────┐
         │           │          │          │           │                │        │
    [SAT-US1]  [SAT-US2]  [SAT-US3]  [MFG-US1]  [SAT-EU1]      [SAT-EU2]  [MFG-EU1]
    SD-WAN     SD-WAN     SD-WAN     SD-WAN      SD-WAN          SD-WAN     SD-WAN
    Appliance  Appliance  Appliance  Appliance   Appliance       Appliance  Appliance
```

---

## Figure WAN-3 — BGP Dual-ISP at HQ Edge

```
                    ┌───────────┐         ┌───────────┐
                    │   ISP-A   │         │   ISP-B   │
                    │  (Tier 1) │         │ (Regional)│
                    └─────┬─────┘         └─────┬─────┘
                          │ BGP eBGP             │ BGP eBGP
                    ┌─────▼─────────────────────▼─────┐
                    │        EDGE ROUTERS (HA pair)    │
                    │   eBGP to both ISPs              │
                    │   AS Path, Local-Pref tuning     │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │     NEXT-GEN FIREWALLS (HA)     │
                    │   Stateful inspection • IPS     │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │          CORE SWITCHES          │
                    │       US HQ Internal LAN        │
                    └─────────────────────────────────┘
```

---

## Figure HQ-1 — US Headquarters Three-Tier Network with Edge

```mermaid
graph TB
    ISP_A["ISP-A\nTier-1 Carrier"]
    ISP_B["ISP-B\nRegional Broadband"]
    EDGE["Edge Routers (HA)\nBGP Dual-ISP"]
    FW["NGFWs (Active/Standby)\nIPS • NAT • VPN Concentrator"]
    DMZ["DMZ\nWeb Servers • Email Gateway\nPartner APIs"]
    CORE["Core Switches (HA)\n10/40 GbE Fiber"]
    DIST1["Dist Switch 1\nBuilding A — L3 OSPF"]
    DIST2["Dist Switch 2\nBuilding B — L3 OSPF"]
    DIST3["Dist Switch 3\nData Center Wing"]
    ACC1["Access Switches\nVLAN 10/20/30/40"]
    ACC2["Access Switches\nVLAN 10/20/30/40"]
    DC["HQ Data Center\nServers • DNS • DHCP • NMS"]
    WAN["WAN / MPLS\nto EU, Data Centers"]

    ISP_A --> EDGE
    ISP_B --> EDGE
    EDGE --> FW
    FW --> DMZ
    FW --> CORE
    CORE --> DIST1
    CORE --> DIST2
    CORE --> DIST3
    DIST1 --> ACC1
    DIST2 --> ACC2
    DIST3 --> DC
    CORE --> WAN
```

---

## Figure EU-1 — European Office Three-Tier Network Diagram

```
         ┌──────────────────────────────────────────────────────┐
         │              EUROPEAN MAIN OFFICE — FRANKFURT        │
         │                                                      │
         │  [ISP-DE1: Deutsche Telekom]  [ISP-EU2: Broadband]  │
         │         │ BGP                       │ BGP            │
         │  ┌──────▼────────────────────────▼──────┐          │
         │  │        EDGE ROUTERS (HA) + NGFW       │          │
         │  │     NAT • IPS • VPN Concentrator      │          │
         │  └─────────────────────┬─────────────────┘          │
         │                        │                             │
         │  ┌─────────────────────▼─────────────────┐          │
         │  │         CORE SWITCHES (10 GbE HA)      │          │
         │  └──────┬────────────┬──────────┬────────┘          │
         │         │            │           │                   │
         │  [Dist-EU1]    [Dist-EU2]   [Dist-EU3]              │
         │  Floor 1-3     Floor 4-6    DC Wing                  │
         │      │              │            │                   │
         │  [Access Sw]   [Access Sw]   [EU-DC Servers]        │
         │  VLAN 10/20/30  VLAN 10/20    DNS/DHCP              │
         │                                                      │
         └──────────────────────────────────────────────────────┘
```

---

## Figure DC-1 — Enterprise Data Center Topology and WAN Connectivity

```mermaid
graph LR
    subgraph USDC1["■ US-DC1 — Dallas (Colocation)"]
        SP1["Spine 1"] --- SP2["Spine 2"]
        L1A["Leaf 1A"] --- SP1
        L1A --- SP2
        L1B["Leaf 1B"] --- SP1
        L1B --- SP2
        SRV1["Servers\nERP / Apps"]
        SRV1 --- L1A
    end

    subgraph USDC2["■ US-DC2 — Chicago (Colocation)"]
        SP3["Spine 1"] --- SP4["Spine 2"]
        L2A["Leaf 1A"] --- SP3
        L2A --- SP4
        SRV2["Servers\nMES / Backup"]
        SRV2 --- L2A
    end

    subgraph HQDC["■ HQ-DC — Austin (On-Prem)"]
        HQSRV["Core Business\nApps + DNS/DHCP"]
    end

    subgraph EUDC["■ EU-DC — Frankfurt (On-Prem)"]
        EUSRV["EU Apps\n+ DNS/DHCP"]
    end

    HQ["★ US HQ\nAustin"]
    EUHQ["★ EU HQ\nFrankfurt"]

    HQ <-->|MPLS| USDC1
    HQ <-->|MPLS| USDC2
    HQ <-->|Private Fiber| HQDC
    EUHQ <-->|Private Fiber| EUDC
    HQDC <-->|"MPLS + IPsec\n(replication)"| USDC1
    USDC1 <-->|"MPLS + IPsec"| USDC2
    USDC2 <-->|"MPLS + IPsec"| EUDC
```

---

## Figure DC-2 — Data Center Leaf-Spine Architecture

```
        ┌─────────────┐          ┌─────────────┐
        │   Spine 1   │◄────────►│   Spine 2   │
        │  (100 GbE)  │          │  (100 GbE)  │
        └──┬──────┬───┘          └───┬──────┬──┘
           │      │                  │      │
      ┌────▼─┐  ┌─▼────┐       ┌────▼─┐  ┌─▼────┐
      │Leaf A│  │Leaf B│       │Leaf C│  │Leaf D│
      │25GbE │  │25GbE │       │25GbE │  │25GbE │
      └──┬───┘  └──┬───┘       └──┬───┘  └──┬───┘
         │         │              │          │
    [Servers]  [Servers]     [Servers]   [Storage]
    App Tier   DB Tier       Backup      SAN/NAS

    Any server → any server = exactly 2 hops
    (Leaf → Spine → Leaf)
    Ch. 4: Consistent forwarding path, no bottleneck
```

---

## Figure DC-3 — DMZ Dual-Firewall Architecture

```
    ═══════════════════ INTERNET ═══════════════════
                             │
                    ┌────────▼────────┐
                    │  External NGFW  │
                    │  Stateful filter│
                    │  IPS (inbound)  │
                    └────────┬────────┘
                             │
    ╔══════════════════════════════════════╗
    ║              D M Z                   ║
    ║  ┌──────────┐  ┌────────────────┐   ║
    ║  │ Web/App  │  │ Email Gateway  │   ║
    ║  │ Servers  │  │ Partner APIs   │   ║
    ║  └──────────┘  └────────────────┘   ║
    ╚══════════════════════════════════════╝
                             │
                    ┌────────▼────────┐
                    │  Internal NGFW  │
                    │  App-layer DPI  │
                    │  ACL enforcement│
                    └────────┬────────┘
                             │
    ╔══════════════════════════════════════╗
    ║         INTERNAL SERVER FARM         ║
    ║  Databases • ERP Backend • LDAP      ║
    ╚══════════════════════════════════════╝

    (Kurose & Ross Ch. 8: Defense-in-depth,
     attacker must breach TWO firewalls to
     reach internal servers)
```

---

## Figure SAT-1 — Satellite Office Collapsed Two-Tier Architecture

```
    ┌────────────────────────────────────────────────────────┐
    │            SATELLITE OFFICE (50–250 users)             │
    │                                                        │
    │  [Broadband ISP 1]    [Broadband ISP 2 / 4G backup]   │
    │         │                      │                       │
    │  ┌──────▼──────────────────────▼──────┐               │
    │  │   SD-WAN Edge Appliance             │               │
    │  │   IPsec VPN to Regional Hub         │               │
    │  │   Local Internet Breakout           │               │
    │  │   Integrated Firewall               │               │
    │  └─────────────────────────┬──────────┘               │
    │                            │                           │
    │  ┌─────────────────────────▼──────────┐               │
    │  │   Collapsed Core (Layer 3 Switch)   │               │
    │  │   OSPF • Inter-VLAN routing • ACLs  │               │
    │  └────┬──────────────┬────────────┬───┘               │
    │       │              │            │                    │
    │  [Access Sw 1]  [Access Sw 2]  [Access Sw 3]          │
    │   VLAN 10/20     VLAN 10/30    VLAN 40/50              │
    │   Workstations   Wireless APs  Mgmt / Guest            │
    └────────────────────────────────────────────────────────┘
```

---

## Figure MFG-1 — Manufacturing Site IT/OT Network Segmentation

```mermaid
graph TB
    WAN["Enterprise WAN\n(SD-WAN to Regional Hub)"]

    subgraph IT["IT Zone — Standard Enterprise Network"]
        IT_FW["IT Edge Firewall"]
        IT_SW["IT LAN Switches\nVLAN 10/20/30"]
        IT_USERS["Admin Workstations\nIP Phones • Laptops"]
    end

    subgraph DMZ["IT/OT DMZ — Controlled Bridge Zone"]
        NGFW["IT/OT NGFW\n(Deep Packet Inspection)"]
        HIST["Data Historian\n(OPC-UA, time-series)"]
        JUMP["Jump Server\n(Vendor Remote Access)"]
        MES["MES Interface\n(ERP Integration)"]
    end

    subgraph OT["OT Zone — Industrial Control Network (Isolated)"]
        OT_SW["Industrial Ethernet\nSwitches (ruggedized)"]
        PLC["PLCs / RTUs"]
        SCADA["SCADA Servers"]
        SENSOR["IoT Sensors\nActuators"]
    end

    WAN --> IT_FW
    IT_FW --> IT_SW
    IT_SW --> IT_USERS
    IT_SW <-->|"Restricted:\nHistorian data only"| NGFW
    NGFW --> HIST
    NGFW --> JUMP
    NGFW --> MES
    HIST <-->|"OPC-UA read-only"| OT_SW
    JUMP <-->|"Supervised access\nonly via jump host"| OT_SW
    OT_SW --- PLC
    OT_SW --- SCADA
    OT_SW --- SENSOR
```

---

## Figure VPN-1 — Remote Access VPN Architecture

```
   ┌─────────────────────────────────────────────────────────┐
   │                   REMOTE USERS (3,000)                   │
   │                                                         │
   │  [Home WiFi]  [Hotel Network]  [4G/5G Mobile]  [BYOD]  │
   └──────────────────────────┬──────────────────────────────┘
                               │ TLS 1.3 or IPsec Tunnel Mode
                               │ (Encrypted — Ch. 8)
           ┌────────────────────┴────────────────────┐
           │                                         │
    ┌──────▼──────────┐                    ┌─────────▼───────┐
    │  US VPN         │                    │  EU VPN         │
    │  Concentrators  │                    │  Concentrators  │
    │  US-VPN-1 + 2   │                    │  EU-VPN-1 + 2   │
    │  Active-Active  │                    │  Active-Active  │
    └──────┬──────────┘                    └─────────┬───────┘
           │ MFA: user+password + TOTP              │
           │ Device cert (managed devices)          │
           │                                        │
    ┌──────▼──────────────────────────────▼─────────┐
    │              ENTERPRISE NETWORK                │
    │     US HQ ←→ EU HQ ←→ Data Centers            │
    │     All internal application resources         │
    └────────────────────────────────────────────────┘

   SPLIT TUNNEL POLICY:
   ├── *.barrios.internal → VPN tunnel → internal DNS → app server
   ├── SaaS (M365, Salesforce) → local Internet breakout (direct)
   └── Sensitive systems (finance, MES) → full tunnel enforced
```

---

## Figure VPN-2 — IPsec Tunnel Mode Encapsulation

```
   ORIGINAL PACKET (before IPsec):
   ┌──────────────┬───────────────┬────────────────────────────────┐
   │ Original IP  │  TCP/UDP Hdr  │          Payload (data)        │
   │ Header       │               │                                │
   └──────────────┴───────────────┴────────────────────────────────┘

   AFTER IPsec TUNNEL MODE (ESP) ENCAPSULATION (Ch. 8):
   ┌─────────────┬─────┬══════════════════════════════════════════╗
   │ New IP Hdr  │ ESP │  ENCRYPTED: [Original IP][TCP/UDP][Data] ║
   │ (public IPs)│ Hdr │  + ESP Authentication Trailer            ║
   └─────────────┴─────┴══════════════════════════════════════════╝

   • New outer IP header: src = VPN appliance public IP,
                          dst = VPN concentrator public IP
   • Original IP packet: fully encrypted by AES-256-GCM
   • ESP Authentication: HMAC-SHA-256 integrity check
   • Eavesdropper sees only: encrypted blob between two public IPs
```

---

## Figure SEC-1 — Defense-in-Depth Security Architecture

```mermaid
graph TB
    INTERNET["🌐 Internet / Untrusted Network"]

    subgraph PERIM["Perimeter Layer"]
        EFW["External NGFW\nStateful + IPS"]
        IDS_EXT["Network IPS\n(Inline — Ch. 8)"]
    end

    subgraph DMZ_LAYER["DMZ Layer"]
        DMZ_SRV["DMZ Servers\nWeb • Email • API"]
    end

    subgraph INT_LAYER["Internal Layer"]
        IFW["Internal NGFW\nApp-Layer DPI"]
        NAC["802.1X NAC\nRADIUS Auth"]
        VLAN_SEG["VLAN Segmentation\nACLs at Dist Layer"]
    end

    subgraph ENDPOINT["Endpoint Layer"]
        EPP["Endpoint Protection\nHIDS • DLP • FDE"]
    end

    subgraph MGMT["Management Layer"]
        SIEM["SIEM\nLog Correlation"]
        NMS["NMS + SNMPv3\nNetwork Monitoring"]
    end

    INTERNET --> EFW
    EFW --> IDS_EXT
    IDS_EXT --> DMZ_SRV
    DMZ_SRV --> IFW
    IFW --> NAC
    NAC --> VLAN_SEG
    VLAN_SEG --> EPP
    EFW -.->|Logs| SIEM
    IDS_EXT -.->|Alerts| SIEM
    IFW -.->|Logs| SIEM
    NAC -.->|Auth events| SIEM
    VLAN_SEG -.->|NetFlow| NMS
```

---

## Figure SEC-3 — 802.1X NAC Authentication Flow

```
  End-User Device          Access Switch          RADIUS Server
  (Supplicant)             (Authenticator)        (Auth Server)
       │                        │                      │
       │── EAPOL Start ────────►│                      │
       │                        │── RADIUS Access ────►│
       │                        │   Request (EAP)      │
       │◄── EAP Request ────────│◄─ RADIUS Challenge ──│
       │    (Identity)          │                      │
       │── EAP Response ───────►│── RADIUS Access ────►│
       │   (username/cert)      │   Request w/creds    │
       │                        │                      │
       │                        │◄─ RADIUS Access ─────│
       │                        │   Accept/Reject      │
       │◄── EAP Success ────────│                      │
       │    (port opened)       │                      │
       │                        │
       │    Port placed in appropriate VLAN (VLAN 10, 30, 50...)
       │    or quarantine VLAN if authentication fails
```

---

## Figure WIFI-1 — Wireless Controller Architecture (WLC + LWAP)

```
                    ┌──────────────────────────────┐
                    │   WIRELESS LAN CONTROLLER     │
                    │   (WLC — Centralized)         │
                    │   SDN-style: Policy + Control │
                    │   Ch. 4: Separation of planes │
                    └──────────────┬───────────────┘
                                   │ CAPWAP Tunnels
              ┌────────────────────┼────────────────────┐
              │                    │                    │
    ┌─────────▼────┐    ┌──────────▼─────┐   ┌────────▼───────┐
    │  LWAP-1      │    │   LWAP-2       │   │   LWAP-3       │
    │  Floor 1     │    │   Floor 2      │   │  Conf. Room    │
    │  802.11ax    │    │   802.11ax     │   │  Wi-Fi 6E      │
    │  2.4+5 GHz   │    │   2.4+5 GHz   │   │  2.4+5+6 GHz   │
    └──────┬───────┘    └──────┬─────────┘   └───────┬────────┘
           │                   │                     │
    ┌──────▼────────────────────▼─────────────────────▼────────┐
    │                  WIRED LAN BACKBONE                       │
    │              (PoE Access Switches — Ch. 6)                │
    └───────────────────────────────────────────────────────────┘
```

---

## Figure WIFI-3 — 802.11 CSMA/CA Medium Access (Textbook Concept)

```
   CSMA/CA: Carrier Sense Multiple Access with Collision Avoidance
   (Kurose & Ross, Ch. 7)

   Device A           Channel              Device B
      │                                       │
      │── Sense channel (idle?) ──────────────│
      │   [Channel idle > DIFS time]          │
      │── Random Backoff Timer ───────────────│
      │   [Wait random N slots]               │
      │── TRANSMIT FRAME ──────────────────►  │
      │                                       │
      │                         [B senses channel BUSY]
      │                         [B defers, waits]
      │◄── ACK received ──────────────────── │
      │   (success confirmed)                 │
                                              │
                                 [Channel idle again]
                                 [B counts down backoff]
                                 [B transmits]

   RTS/CTS optional extension (solves Hidden Node problem):
   A sends RTS → AP sends CTS → all hear CTS → only A transmits
```

---

## Figure MGMT-1 — SNMP Manager-Agent Architecture

```
                    ╔═══════════════════════════════╗
                    ║    NETWORK MANAGEMENT SYSTEM  ║
                    ║    (NMS — SNMP Manager)        ║
                    ║    HQ Data Center             ║
                    ║                               ║
                    ║  [GET/SET Requests]           ║
                    ║  [TRAP Receiver]              ║
                    ║  [MIB Browser]                ║
                    ║  [Topology Map]               ║
                    ╚═════════════┬═════════════════╝
                                  │ SNMPv3 (authenticated + encrypted)
              ┌───────────────────┼──────────────────────────────┐
              │                   │                              │
    ┌─────────▼────┐   ┌──────────▼──────┐         ┌────────────▼─────┐
    │  SNMP Agent  │   │   SNMP Agent    │         │   SNMP Agent     │
    │  Core Switch │   │  WAN Router     │   ...   │  Remote Site AP  │
    │              │   │                 │         │                  │
    │  MIB:        │   │  MIB:           │         │  MIB:            │
    │  - ifInOctets│   │  - ifUtilization│         │  - dot11Clients  │
    │  - cpuUsage  │   │  - bgpPeerState │         │  - channelUtil   │
    │  - memUsage  │   │  - ospfNbrState │         │  - signalStrength│
    └──────────────┘   └─────────────────┘         └──────────────────┘
    
    GET = NMS polls agent for MIB values (every 5 min)
    TRAP = Agent sends unsolicited alert (interface down, threshold exceeded)
    SNMPv3 = Adds authentication (HMAC-SHA) + privacy (AES) — Ch. 8
```

---

## Figure MGMT-2 — SIEM Log Aggregation Architecture

```mermaid
graph LR
    subgraph SOURCES["Log Sources (All Sites)"]
        FW["Firewalls\nNGFW Logs"]
        IDS["IDS/IPS\nAlert Feeds"]
        VPN["VPN Concentrators\nAuth Events"]
        SW["Switches/Routers\nSyslog + NetFlow"]
        EP["Endpoint Agents\nHIDS Alerts"]
        DNS_SRC["DNS Servers\nQuery Logs"]
    end

    SIEM["🔍 SIEM Platform\n(Log Correlation Engine)\n\nCorrelates events across sources\nDetects attack patterns\nGenerates incidents\nCompliance reporting"]

    SOC["Security Operations\nTeam (SOC)\nIncident Response"]

    TICKET["Ticketing System\nIncident Tracking"]

    FW --> SIEM
    IDS --> SIEM
    VPN --> SIEM
    SW --> SIEM
    EP --> SIEM
    DNS_SRC --> SIEM
    SIEM --> SOC
    SOC --> TICKET
```

---

*All diagrams © Barrios Manufacturing & Solutions Term Paper, 2026*  
*Networking concepts grounded in: Kurose & Ross, Computer Networking: A Top-Down Approach, 8th ed.*
