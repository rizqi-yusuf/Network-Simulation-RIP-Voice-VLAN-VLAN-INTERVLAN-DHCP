<div align="center">
  <h1>🌐 Complete Enterprise Network Simulation: 4 Routers, 4 Switches, Multi-Site VoIP, Wireless, VLANs & DHCP</h1>
  
  <p>
    <b>The Ultimate Blueprint: Full CLI Configurations for All 4 Routers, 4 Switches, Dynamic Routing (RIPv2), DHCP Pools, and Multi-Site Telephony Services.</b>
  </p>
</div>

<br />

## 📖 Project Overview
This repository provides an exhaustive, step-by-step documentation and configuration guide for a complex 4-router and 4-switch enterprise network topology built in Cisco Packet Tracer. Every single configuration command for all networking hardware is fully disclosed here to serve as a comprehensive reference and learning resource.

---

## 🗺️ Network Architecture & Topology

The topology is structured across multiple zones connected via serial WAN links and high-speed trunk/access lines, handling segmented data, wireless mobility, centralized server farms, and multi-site VoIP endpoints.

<div align="center">

<img width="886" height="366" alt="{81BCB1C9-5C1D-48EA-826D-A04BFC87CC5C}" src="https://github.com/user-attachments/assets/c2c3a93e-fb82-4179-a5fc-29e6c645597e" />

</div>

---

## 📋 Comprehensive Subnet, VLAN & VoIP Extension Table

| Zone / Site | Device Target | VLAN ID | Subnet / Network | VoIP Extensions | Allocation & Purpose |
| :--- | :--- | :---: | :--- | :---: | :--- |
| **Site 1 (Top-Left)** | Switch 1 & Router 1 | VLAN 10 | `192.70.30.0/24` | **101 – 102** | Access Switch & IP Phones (Top-Left)[cite: 9] |
| **Site 2 (Top-Right)** | Switch 2 & Router 2 | VLAN 20 | `192.80.30.0/24` | **103 – 104** | Access Switch & IP Phones (Top-Right)[cite: 9] |
| **Site 3 (Bottom-Left)**| Switch 3 & Router 3 | VLAN 30 | `192.30.30.0/24` | **105 – 106** | Access Switch & IP Phones (Bottom-Left) |
| **Site 4 (Server Site)**| Switch 4 & Router 4 | VLAN 60 | `192.60.30.0/24` | N/A | Centralized Web & DNS Server Farm[cite: 9, 10] |
| **Data LAN** | Workstations | VLAN 10 | `192.10.30.0/24` | N/A | Static & Dynamic Client Workstations[cite: 10] |
| **Wireless LAN** | Access Points | VLAN 20 | `192.20.30.0/24` | N/A | DHCP Pool for Mobile Devices[cite: 10] |
| **WAN Backbones** | Serial Interfaces| N/A | Various `/30` | N/A | Point-to-Point Serial Links (`Se0/3/0`, `Se0/3/1`)[cite: 9, 10] |

---

## 📝 Complete Hardware Configurations (4 Switches & 4 Routers)

---

### PART A: SWITCH CONFIGURATIONS (Switch 1 to Switch 4)

#### 1. Switch 1 (Top-Left Site - Supporting Ext 101-102)
```text
enable
configure terminal
hostname Switch1

vlan 10
 name VOICE_TOPLEFT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 10
 no shutdown
```

#### 2. Switch 2 (Top-Right Site - Supporting Ext 103-104)
```text
enable
configure terminal
hostname Switch2

vlan 20
 name VOICE_TOPRIGHT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
```

#### 3. Switch 3 (Bottom-Left Site - Supporting Ext 105-106)
```text
enable
configure terminal
hostname Switch3

vlan 30
 name VOICE_BOTLEFT

interface FastEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 30
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 no shutdown
```

#### 4. Switch 4 (Server & Data Site)
```text
enable
configure terminal
hostname Switch4

vlan 10
 name DATA_LAN
vlan 60
 name SERVER

interface GigabitEthernet 0/1
 switchport mode trunk
 no shutdown

interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 60
 no shutdown
```

---

### PART B: ROUTER CONFIGURATIONS (Router 1 to Router 4)

#### 1. Router 1 (Top-Left Gateway - VoIP 101-102)[cite: 9]
```text
enable
configure terminal
hostname Router1

! --- Serial WAN Link ---
interface Serial0/3/0
 ip address 192.66.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Inter-VLAN Sub-interface & DHCP ---
interface FastEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.70.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R1
 network 192.70.30.0 255.255.255.0
 default-router 192.70.30.1
 option 150 ip 192.70.30.1

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.70.30.1 port 2000
 ephone-dn 1
  number 101
 ephone-dn 2
  number 102
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.70.30.0
 network 192.66.30.0
 no auto-summary
```

#### 2. Router 2 (Top-Right Gateway - VoIP 103-104)[cite: 9]
```text
enable
configure terminal
hostname Router2

! --- Serial WAN Links ---
interface Serial0/3/0
 ip address 192.66.30.2 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 192.46.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Inter-VLAN Sub-interface & DHCP ---
interface FastEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.80.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R2
 network 192.80.30.0 255.255.255.0
 default-router 192.80.30.1
 option 150 ip 192.80.30.1

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.80.30.1 port 2000
 ephone-dn 1
  number 103
 ephone-dn 2
  number 104
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.80.30.0
 network 192.66.30.0
 network 192.46.30.0
 no auto-summary
```

#### 3. Router 3 (Bottom-Left Gateway - VoIP 105-106 & Wireless DHCP)
```text
enable
configure terminal
hostname Router3

! --- Serial WAN Links ---
interface Serial0/3/1
 ip address 192.46.30.2 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 192.56.30.1 255.255.255.252
 clock rate 64000
 no shutdown

! --- Sub-interfaces, Data LAN & Wireless DHCP ---
interface FastEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.30.30.1 255.255.255.0

ip dhcp pool VOIP_POOL_R3
 network 192.30.30.0 255.255.255.0
 default-router 192.30.30.1
 option 150 ip 192.30.30.1

ip dhcp pool WIRELESS_POOL_R3
 network 192.20.30.0 255.255.255.0
 default-router 192.20.30.1
 dns-server 192.60.30.2

! --- Telephony Service (CME) ---
telephony-service
 max-ephones 2
 max-dn 2
 ip source-address 192.30.30.1 port 2000
 ephone-dn 1
  number 105
 ephone-dn 2
  number 106
 auto assign 1 to 2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.30.30.0
 network 192.20.30.0
 network 192.46.30.0
 network 192.56.30.0
 no auto-summary
```

#### 4. Router 4 (Server Site Gateway)
```text
enable
configure terminal
hostname Router4

! --- Serial WAN Link & Server LAN ---
interface Serial0/3/1
 ip address 192.56.30.2 255.255.255.252
 no shutdown

interface FastEthernet 0/0
 ip address 192.60.30.1 255.255.255.0
 no shutdown

ip dhcp pool SERVER_DATA_POOL
 network 192.10.30.0 255.255.255.0
 default-router 192.10.30.1
 dns-server 192.60.30.2

! --- Dynamic Routing (RIPv2) ---
router rip
 version 2
 network 192.56.30.0
 network 192.60.30.0
 network 192.10.30.0
 no auto-summary
```

---

## 🧪 Verification & Testing Procedures

* **Switch Port Validation:** Run `show vlan brief` and `show interfaces trunk` on all 4 switches to confirm proper trunk encapsulation and access port assignment.
* **Routing Table Verification:** Run `show ip route` across all 4 routers to verify complete path synchronization via RIPv2.
* **VoIP Call Test:** Dial extension **101** from extension **103** or **105** to test multi-site call routing across different router boundaries

## 👤 Author
 
* **Name:** Rizqi Maulana Yusuf
* **GitHub:** [@rizqi-yusuf](https://github.com/rizqi-yusuf)
<div align="center">
⭐ **If this helped you, drop a star on the repo!** ⭐
 
</div>
