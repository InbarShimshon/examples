```mermaid
graph TD
    Internet1[ISP 1] --> BGP[BGP Edge]
    Internet2[ISP 2] --> BGP

    BGP --> FW[Firewall Cluster]

    FW --> Core1[Core Switch 1]
    FW --> Core2[Core Switch 2]

    Core1 --> Dist1[Distribution Switches]
    Core2 --> Dist1

    Dist1 --> Access1[Access Switches]

    Access1 --> Users[User Devices]
    Access1 --> Phones[IP Phones]
    Access1 --> IoT[IoT Devices]

    Core1 --> DC[Data Center]
    Core2 --> DC

    FW --> VPN[VPN Concentrator]
