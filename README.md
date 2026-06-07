# Project: 3-Zone Corporate Firewall Configuration using Extended ACLs
## 🚀 Created By: TechCo4
 Implementation of a secure enterprise network architecture using **Cisco Packet Tracer**. This project demonstrates the configuration of a **3-Zone Firewall** (Internal LAN, Demilitarized Zone - DMZ, and External WAN) on a Cisco 4331 Edge Router using Extended Access Control Lists (ACLs) to enforce granular security policies.

---

## 📌 Project Overview
In modern network security, minimizing the attack surface is critical. This project simulates a corporate headquarters network (`HQ-Edge`) connected to an Internet Service Provider (`ISP-Internet`). The primary goal is to isolate public-facing servers in a **DMZ** and implement strict firewall rules so that internal network segments have controlled access based on organizational roles.

### Key Objectives:
* **Multi-Zone Segregation:** Divide the infrastructure into Private LAN, DMZ, and Public WAN.
* **Granular Traffic Filtering:** Utilize Cisco Extended Named ACLs to allow/deny specific protocols (HTTP, FTP, ICMP) based on port numbers.
* **Static Routing:** Implement reliable, bulletproof static routing between the Enterprise Edge and the ISP.
* **Network Verification:** Demonstrate successful security enforcement via simulation and live packet match counters.

---

## 🗺️ Network Topology & Architecture
The network is designed around a three-interface architecture on the core firewall router (`HQ-Edge`):

1.  **Inside LAN (Private Segment):** Houses internal departments (HR-PC, Staff-PC) on the `192.168.10.0/24` subnet.
2.  **DMZ (Demilitarized Zone):** Hosts public services (Web Server on Port 80, FTP Server on Port 21) on the `10.0.0.0/24` subnet.
3.  **Outside WAN (Public Internet):** Connects to the ISP via a `/30` point-to-point link (`100.0.0.0/30`) to simulate an internet-facing simulated host (`200.100.50.50`).

---

## 📊 IP Addressing Scheme

| Device Name | Interface / Connection | IP Address | Subnet Mask | Default Gateway | Zone / Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **HQ-Edge** | GigabitEthernet0/0/0 | `192.168.10.1` | `255.255.255.0` | N/A | Inside Gateway (LAN) |
| **HQ-Edge** | GigabitEthernet0/0/1 | `10.0.0.1` | `255.255.255.0` | N/A | DMZ Gateway (Servers) |
| **HQ-Edge** | GigabitEthernet0/0/2 | `100.0.0.1` | `255.255.255.252`| N/A | Outside Gateway (WAN/ISP) |
| **ISP-Internet**| GigabitEthernet0/0/0 | `100.0.0.2` | `255.255.255.252`| N/A | ISP Link to HQ |
| **ISP-Internet**| GigabitEthernet0/0/1 | `200.100.50.1` | `255.255.255.0` | N/A | ISP Link to Internet Host |
| **HR-PC** | FastEthernet0 | `192.168.10.10`| `255.255.255.0` | `192.168.10.1` | Secure Internal User |
| **Staff-PC** | FastEthernet0 | `192.168.10.20`| `255.255.255.0` | `192.168.10.1` | Restricted Internal User |
| **Web-Server**| FastEthernet0 | `10.0.0.80` | `255.255.255.0` | `10.0.0.1` | Publicly Accessible Web |
| **FTP-Server**| FastEthernet0 | `10.0.0.21` | `255.255.255.0` | `10.0.0.1` | Restricted Storage Server |
| **Outside-User**| FastEthernet0 | `200.100.50.50`| `255.255.255.0` | `200.100.50.1` | Public Internet Host |

---

## 🔒 Firewall Security Policies & ACL Implementation

An Extended Access Control List named **`OFFICE-FIREWALL`** is deployed inbound (`in`) on the Inside LAN interface (`GigabitEthernet0/0/0`). This intercepts malicious or restricted traffic immediately as it enters the router, saving CPU cycles.

### Configured Security Rules:
1.  **Rule 1 (HTTP Permit):** Allows the Secure HR Department (`192.168.10.10`) to access the corporate Web Server (`10.0.0.80`) over TCP Port 80.
2.  **Rule 2 (FTP Deny):** Strictly blocks the entire local office subnet (`192.168.10.0/24`) from accessing the sensitive FTP Storage Server (`10.0.0.21`) over TCP Port 21 to prevent unauthorized data exfiltration.
3.  **Rule 3 (Implicit Permit IP):** Allows all other standard IP communication (e.g., local ICMP pings, basic corporate traffic) across authorized zones.

### Cisco IOS Configuration Commands:

```cisco-ios
! Core Routing Configurations
HQ-Edge(config)# ip route 0.0.0.0 0.0.0.0 100.0.0.2
ISP-Internet(config)# ip route 192.168.10.0 255.255.255.0 100.0.0.1
ISP-Internet(config)# ip route 10.0.0.0 255.255.255.0 100.0.0.1

! Extended Access-List Definition
HQ-Edge(config)# ip access-list extended OFFICE-FIREWALL
HQ-Edge(config-ext-nacl)# permit tcp host 192.168.10.10 host 10.0.0.80 eq 80
HQ-Edge(config-ext-nacl)# deny tcp 192.168.10.0 0.0.0.255 host 10.0.0.21 eq 21
HQ-Edge(config-ext-nacl)# permit ip any any
HQ-Edge(config-ext-nacl)# exit

! Applying Firewall Policy to Interface
HQ-Edge(config)# interface GigabitEthernet0/0/0
HQ-Edge(config-if)# ip access-group OFFICE-FIREWALL in
HQ-Edge(config-if)# end
HQ-Edge# write memory
